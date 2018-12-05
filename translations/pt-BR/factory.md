# O Padrão *Factory* (Fábrica)

## Problema
Nós precisamos criar objetos sem ter que especificar exatamente a classe do objeto que será criado.

## Solução
O Padrão *Factory* é uma especialização do padrão [Template](template.md). Nós começamos criando
uma classe base genérica onde nós não fazemos a decisão "qual classe". Ao invés, sempre que ela precisar
criar um objeto, ela chama um método que é definido na subclasse. Então, dependendo da subclasse que
usarmos (*factory*), nós criamos objetos de uma classe ou outra (**produtos**).

## Exemplo
Imagine que te pediram para construir um simulador de vida em uma lagoa que tem vários patos:

```ruby
class Pond
  def initialize(number_ducks)
    @ducks = number_ducks.times.inject([]) do |ducks, i|
      ducks << Duck.new("Duck#{i}")
      ducks
    end
  end

  def simulate_one_day
    @ducks.each {|duck| duck.speak}
    @ducks.each {|duck| duck.eat}
    @ducks.each {|duck| duck.sleep}
  end
end

pond = Pond.new(3)
pond.simulate_one_day
```

Mas como nós iriamos modelar nossa classe `Pond` (Lagoa) se nós quisermos ter sapos ao invés de patos?
Na implementação acima, nós estamos especificando no inicializador da classe que ela deveria ser
preenchida com patos. Por isso, nós iremos refatora-lo para que a decisão de criar um tipo de animal
ou outro seja realizada em uma subclasse.

```ruby
class Pond
  def initialize(number_animals)
    @animals = number_animals.times.inject([]) do |animals, i|
      animals << new_animal("Animal#{i}")
      animals
    end
  end

  def simulate_one_day
    @animals.each {|animal| animal.speak}
    @animals.each {|animal| animal.eat}
    @animals.each {|animal| animal.sleep}
  end
end

class FrogPond < Pond
  def new_animal(name)
    Frog.new(name)
  end
end

pond = FrogPond.new(3)
pond.simulate_one_day
```
