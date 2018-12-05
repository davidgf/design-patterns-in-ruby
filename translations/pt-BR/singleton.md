# O Padrão *Singleton*

## Problema
Nós precisamos ter apenas uma única instância de uma certa classe em toda nossa
aplicação.

## Solução
No padrão *Singleton*, o acesso ao construtor é restrito, de forma que não é
possível instanciar. Então, a criação de uma única instância é realizada dentro
da classe e guardado em uma variável de classe. Ela pode ser acessada através
de um *getter* em qualquer parte da sua aplicação.

## Exemplo
Vamos considerar a implementação de uma classe de registro de log.

```ruby
class SimpleLogger
  attr_accessor :level

  ERROR = 1
  WARNING = 2
  INFO = 3

  def initialize
    @log = File.open("log.txt", "w")
    @level = WARNING
  end

  def error(msg)
    @log.puts(msg)
    @log.flush
  end

  def warning(msg)
    @log.puts(msg) if @level >= WARNING
    @log.flush
  end

  def info(msg)
    @log.puts(msg) if @level >= INFO
    @log.flush
  end
end
```

O registro em log é uma funcionalidade usada em toda a sua aplicação, então faz
sentido que exista apenas uma instância de um objeto com essa responsabilidade.
Nós podemos nos certificar que ninguém vai instanciar a classe  `SimpleLogger`
duas vezes ao tornar o seu construtor privado:

```ruby
class SimpleLogger

  # Lots of code deleted...
  @@instance = SimpleLogger.new

  def self.instance
    return @@instance
  end

  private_class_method :new
end

SimpleLogger.instance.info('Computer wins chess game.')
```

Nós podemos obter o mesmo comportamento ao incluir o módulo `Singleton`, de
forma que nós possamos evitar duplicação de código se criarmos diversos
*singletons*:

```ruby
require 'singleton'

class SimpleLogger
  include Singleton
  # Lots of code deleted...
end
```
