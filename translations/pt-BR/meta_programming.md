# O Padrão de Meta Programação

## Problema
Nós queremos ganhar mais flexibilidade ao definir novas classes e criar objetos
customizados sob medida em tempo de execução.

## Solução
Ruby é uma linguagem muito dinâmica, mas isso não se aplica apenas a tipagem.
Nós podemos até definir novos metódos em nossas classes em tempo de execução
graças a **métodos singleton**. Se ao invés de adicionar um método nós quisermos
adicionar um grupo deles, nós também podemos usar o método `extend`, o qual
teria o mesmo efeito que incluir um módulo. Por último mas não menos importante,
com o método `class_eval`nós podemos avaliar uma string no contexto de uma
classe, o qual combinado com interpolação de **string** é realmente um grande
recurso ao criar métodos em tempo de execução.

## Exemplo
Retornando ao [Padrão *Factory* (Fábrica)](factory.md) onde nós criamos
habitats de flora e fauna, nós podemos imaginar que precisamos de mais
flexibilidade. Naquela época, nós tinhamos múltiplas fábricas que nos forneciam
uma lista fixa de combinações para criação de diferentes organismos. Se nós
quiséssemos ter flexibilidade total para criar qualquer combinação que surgisse
evitando ter que criar uma tonelada de classes que sustentem cada combinação,
nós podemos fazer uso de métodos singleton:

```ruby
def new_plant(stem_type, leaf_type)
  plant = Object.new
  if stem_type == :fleshy
    def plant.stem
      'fleshy'
    end
  else
    def plant.stem
      'woody'
    end
  end
  if leaf_type == :broad
    def plant.leaf
      'broad'
    end
  else
    def plant.leaf
      'needle'
    end
  end
  plant
end

plant1 = new_plant(:fleshy, :broad)
plant2 = new_plant(:woody, :needle)
puts "Plant 1's stem: #{plant1.stem} leaf: #{plant1.leaf}"
puts "Plant 2's stem: #{plant2.stem} leaf: #{plant2.leaf}"
```
Nós começamos com um objeto simples em Ruby `Object` e customizamos ele
adicionando métodos. Ao invés de adiciona-los um por um, nós podemos adicionar
um grupo deles:

```ruby
def new_animal(diet, awake)
  animal = Object.new
  if diet == :meat
    animal.extend(Carnivore)
  else
    animal.extend(Herbivore)
  end
  if awake == :day
    animal.extend(Diurnal)
  else
    animal.extend(Nocturnal)
  end
  animal
end
```

Nesse exemplo, o método `extend` faz a mesma coisa que a inclusão de um módulo.

Agora vamos imaginar que nós queremos agrupar animais e árvores que compartilham
a mesma sessão da florestas e manter um controle de suas classificações
biológicas. Apensar deles parecerem dois problemas de programação diferentes,
eles são bem semelhantes, ambos tem como objetivo agrupar uma coleção de
objetos. Idealmente, nós poderíamos estabelecer diferentes tipos de
relacionamentos entre os objetos em tempo de execução dessa forma:

```ruby
class Tiger < CompositeBase
  member_of(:population)
  member_of(:classification)
end

class Tree < CompositeBase
  member_of(:population)
  member_of(:classification)
end

class Jungle < CompositeBase
  composite_of(:population)
end

class Species < CompositeBase
  composite_of(:classification)
end

tony_tiger = Tiger.new('tony')
se_jungle = Jungle.new('southeastern jungle tigers')
se_jungle.add_sub_population(tony_tiger)
```

REVISÃO: NÃO TENHO CERTEZA SE ISSO FOI O QUE VOCÊ QUIS DIZER
!!!!!
!!!!!
=begin
O método `composite_of(:group)` forneceria um método para incluir membros em qualquer grupo
`:group` que pudermos imaginar ao adicionar métodos dinâmicos `add_sub_:group` para as instâncias da classe.
Da mesma forma, o método `member_of(:group)` iria fornecer um método `parent_:group` que saberia de qual grupo
eles são membros. Acontece que isso tudo seria absolutamente possível com alguma meta programação:
=end
!!!!!
!!!!!

```ruby
class CompositeBase
  attr_reader :name

  def initialize(name)
    @name = name
  end

  def self.member_of(composite_name)
    code = %Q{
      attr_accessor :parent_#{composite_name}
    }
    class_eval(code)
  end

  def self.composite_of(composite_name)
    member_of composite_name
    code = %Q{
      def sub_#{composite_name}s
        @sub_#{composite_name}s = [] unless @sub_#{composite_name}s
        @sub_#{composite_name}s
      end
      def add_sub_#{composite_name}(child)
        return if sub_#{composite_name}s.include?(child)
        sub_#{composite_name}s << child
        child.parent_#{composite_name} = self
      end
      def delete_sub_#{composite_name}(child)
        return unless sub_#{composite_name}s.include?(child)
        sub_#{composite_name}s.delete(child)
        child.parent_#{composite_name} = nil
      end
    }
    class_eval(code)
  end
end
```

`CompositeBase` é a classe surporte para o restante dos componentes e implementa
os métodos `member_of` e `composite_of`. Ao simplesmente envia-los o nome do
grupo, eles configuram todos os métodos que nós precisamos através da construção
de uma string que os define. A chamada para `class_eval` interpreta a string
no contexto da classe, tornando-os disponíveis.
