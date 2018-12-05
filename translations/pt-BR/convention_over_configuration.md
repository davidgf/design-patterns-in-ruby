# O Padrão Convenção Sobre Configuração

## Problema
Nós queremos construir um sistema extensível sem ter que carregar a obrigação de configura-lo.

## Solução
O padrão **Convenção Sobre Configuração** sugere estabelecer algumas convenções para nomes de arquivos, classes e métodos, assim como a organização padrão de diretórios, ao invés de depender de arquivos de configurações.

## Exemplo
Vamos considerar um *gateway* que recebe mensagens e as encaminha para seus destinos. Um requisito chave é que seja fácil adicionar protocolos de
mensagens. Uma mensagem se pareceria com isso:

```ruby
require 'uri'

class Message
  attr_accessor :from, :to, :body

  def initialize(from, to, body)
    @from = from
    @to = URI.parse(to)
    @body = body
  end
end
```

`to`(para) é uma `URI` representando o destino final da mensagem. A mensagem pode ser enviada através de uma requisição HTTP *Post* via e-mail ou para um arquivo. Para lidar com todos esses tipos de mensagens, nós iremos criar um adaptador para cada protocolo:

```ruby
require 'net/http'
class HttpAdapter
  def send(message)
    Net::HTTP.start(message.to.host, message.to.port) do |http|
      http.post(message.to.path, message.text)
    end
  end
end
```

Nós precisamos saber qual adaptador nós deveríamos usar para uma determinada mensagem. Nós podemos obter o adaptador em um expressão `case` de acordo com o protocolo, mas isso não parece extensível de forma alguma. Como alternativa, nós vamos definir a primeira convenção: **O nome da classe adaptadora precisa ser `<protocol>Adapter`**. Dessa forma, nós podemos criar o seguinte método para obter o adaptador que precisamos:

```ruby
def adapter_for(message)
  protocol = message.to.scheme.downcase
  adapter_name = "#{protocol.capitalize}Adapter"
  adapter_class = self.class.const_get(adapter_name)
  adapter_class.new
end
```

Basicamente criamos a o nome da classe adaptadora assumindo que ela segue a convenção e obtemos a classe com o método `const_get`.
Agora podemos facilmente adicionar o protocolo `FTP` se nós quisermos. Apesar disso, ainda temos que lidar com o problema de ler novos adaptadores em nosso sistema. Ao invés de ter um arquivo onde vamos requerer (`require`) todos os adaptadores de nossa aplicação, nós definimos uma segunda convenção: **coloque todos os adaptadores no diretório `adapter`**. Se todos os adaptadores estão no mesmo diretório, nós podemos carregar dinamicamente todos eles com o seguinte método:

```ruby
def load_adapters
  lib_dir = File.dirname(__FILE__)
  full_pattern = File.join(lib_dir, 'adapter', '*.rb')
  Dir.glob(full_pattern).each {|file| require file }
end
```

Com essas duas convenções nós tornamos realmente fácil para outros desenvolvedores adicionarem protocolos novos, prontos para serem utilizados automaticamente pela nossa aplicação. A class `MessageGateway` se pareceria com algo assim:

```ruby
class MessageGateway
  def initialize
    load_adapters
  end

  def process_message(message)
    adapter = adapter_for(message)
    adapter.send_message(message)
  end

  def adapter_for(message)
    protocol = message.to.scheme
    adapter_class = protocol.capitalize + 'Adapter'
    adapter_class = self.class.const_get(adapter_class)
    adapter_class.new
  end

  def load_adapters
    lib_dir = File.dirname(__FILE__)
    full_pattern = File.join(lib_dir, 'adapter', '*.rb')
    Dir.glob(full_pattern).each {|file| require file }
  end
end
```

Agora queremos adicionar alguma segurança em nossa plataforma ao controlar quais usuários têm permissão de enviar mensagens a um determinado *host*.
Nós iremos ter uma classe de autorização por *host* com um método genérico `authorized?` (autorizado?) para checar se o usuário pode enviar a mensagem.
Nós também iremos definir uma terceira convenção que irá nos ajudar a criar regras para certos usuários: **se o usuário possuir alguma regra especial, ela vai ser implementada em um método chamada <user>_authorized?**:

```ruby
class RussolsenDotComAuthorizer
  def russ_dot_olsen_authorized?(message)
    true
  end

  def authorized?(message)
    message.body.size < 2048
  end
end

def worm_case(string)
  tokens = string.split('.')
  tokens.map! {|t| t.downcase}
  tokens.join('_dot_')
end

def authorized?(message)
  authorizer = authorizer_for(message)
  user_method = worm_case(message.from) + '_authorized?'
  if authorizer.respond_to?(user_method)
    return authorizer.send(user_method, message)
  end
  authorizer.authorized?(message)
end
```

A última cocisa que podemos fazer se quisermos deixar que outros desenvolvedores ampliem nosso sistema é providenciar alguns exemplos ou um gerador de modelo (*template*) para criar um protótipo de um adaptador, o que seria muito melhor:

```ruby
protocol_name = ARGV[0]
class_name = protocol_name.capitalize + 'Adapter'
file_name = File.join('adapter', protocol_name + '.rb')

scaffolding = %Q{
  class #{class_name}
    def send_message(message)
      # Code to send the message
    end
  end
}

File.open(file_name, 'w') do |f|
  f.write(scaffolding)
end
```
