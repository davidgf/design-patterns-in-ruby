# O Padrão Iterador **(Iterator)**

## Problema
Nós temos um objeto agregado e queremos fornecer um jeito de acessar sua coleção
de sub-objetos sem expor sua representação interna.

## Solução
Existe duas soluções sugeridas: **iteradores externos** e **iteradores internos**

### Iteradores Externos
O iterador é um objeto separado do agregado, o qual é passado como argumento
para inicializar o iterador. O iterador mantém uma referência para o índice
atual e fornece uma interface para perguntar se existem itens restantes, com o
objetivo de pegar o item atual e o próximo.

### Iteradores Internos
Com iteradores internos nós usamos um bloco para passar a lógica para dentro do
agregado. Um exemplo muito bom dessa abordagem é o método `each` de `Array`.

## Exemplo
Um exemplo de um iterador externo para um vetor em Ruby pode se parecer algo
como isso:

```ruby
class ArrayIterator
  def initialize(array)
    @array = array
    @index = 0
  end

  def has_next?
    @index < @array.length
  end

  def item
    @array[@index]
  end

  def next_item
    value = @array[@index]
    @index += 1
    value
  end
end
```

Graças ao *duck typing*, essa implementação funcionaria para qualquer classe
que tem o método `length` e pode ser indexada por um inteiro, como uma `String`.
Para criar uma classe agregada com um iterador interno, nós iremos fazer uso
do módulo `Enumerable`. Nós apenas precisamos nos certificar que nosso método
iterador é chamado de `each` e nós implementamos o operador de comparação `<=>`.
Ao fazer isso, nós automaticamente obtemos um grande quantidade de método úteis,
como `include?`, `all?` ou `sort`. Como exemplo, nós iremos considerar duas
classes: `Account` e `Portfolio`, o qual administram múltiplas contas.

```ruby
class Account
  attr_accessor :name, :balance

  def initialize(name, balance)
    @name = name
    @balance = balance
  end

  def <=>(other)
    balance <=> other.balance
  end
end

class Portfolio
  include Enumerable

  def initialize
    @accounts = []
  end

  def each(&block)
    @accounts.each(&block)
  end

  def add_account(account)
    @accounts << account
  end
end
```

```ruby
my_portfolio.any? {|account| account.balance > 2000}
```
