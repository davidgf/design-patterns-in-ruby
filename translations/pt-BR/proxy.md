# O Padrão *Proxy*

## Problema
Nós queremos ter mais controle sobre como e quando nós acessamos um certo objeto.

## Solução
Com o Padrão *Proxy* nós criamos um objeto, **proxy (representante)**, que tem
uma referência para o objeto real que queremos acessar. Então, sempre que o
cliente chama o representante, ele simplesmente envia a requisição para o
objeto real. Existem três cenários onde esse padrão pode ser útil:
* **Proxy de Proteção**: antes de delegar chamadas ao objeto real, ele adiciona
uma camada de segurança. Uma grande vantagem dessa abordagem é que ela nos adiciona
separação de responsabilidades, uma vez que o representante toma conta de controle
de acesso, enquanto o objeto real está apenas preocupado o com a lógica de negócio.
* **Proxy Remoto**: quando o objeto que nós queremos usar está em outra máquina
e ele deve ser buscado pela rede, o representante toma conta de toda a complexidade
envolvida na conexão, enquanto o cliente pode usar o objeto como se estivesse na
mesma máquina.
* **Proxy Virtual**: ele atrasa a criação de uma objeto até o momento em que ele
é usado.

## Exemplo
Vamos considerar um objeto de conta de banco `BankAccount`:

```ruby
class BankAccount
  attr_reader :balance

  def initialize(starting_balance=0)
    @balance = starting_balance
  end

  def deposit(amount)
    @balance += amount
  end

  def withdraw(amount)
    @balance -= amount
  end
end
```

Se quiser controlar quem acessa ele, podemos construir um objeto representante
que adiciona algumas verificações antes de delegar chamadas a conta do banco.

```ruby
require 'etc'

class AccountProtectionProxy
  def initialize(real_account, owner_name)
    @subject = real_account
    @owner_name = owner_name
  end

  def deposit(amount)
    check_access
    return @subject.deposit(amount)
  end

  def withdraw(amount)
    check_access
    return @subject.withdraw(amount)
  end

  def balance
    check_access
    return @subject.balance
  end

  def check_access
    if Etc.getlogin != @owner_name
      raise "Illegal access: #{Etc.getlogin} cannot access account."
    end
  end
end
```
Talvez o que nós queremos fazer seja apenas criar o objeto da conta do banco
quando ele realmente for necessário. Nós podemos criar um representante que
inicializa o objeto real apenas quando um de seus métodos é chamado e mantém uma
cópia em cache para chamadas futuras:

```ruby
class VirtualAccountProxy
  def initialize(starting_balance=0)
    @starting_balance=starting_balance
  end

  def deposit(amount)
    s = subject
    return s.deposit(amount)
  end

  def withdraw(amount)
    s = subject
    return s.withdraw(amount)
  end

  def balance
    s = subject
    return s.balance
  end

  def subject
    @subject || (@subject = BankAccount.new(@starting_balance))
  end
end
```
