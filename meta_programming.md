# Meta-Programming Pattern

## Problem
We want to gain more flexibility when defining new classes and create custom tailored objects on the fly.

## Solution
Ruby is a very dynamic language, but that doesn't apply only to typing— we can even define new methods in our classes at runtime thanks to **singleton methods**. If instead of adding one method we want to add a group of them, we can also use the `extend` method, which would have the same effect as including a module. Last but not least, with the `class_eval` method we can evaluate a string in the context of a class, which combined with string interpolations is a really great asset to create new methods at runtime.

## Example
Going back to the [Factory Pattern](factory.md) where we created flora and fauna habitats, we might think we need more flexibility. Back then, we had multiple factories that provided us with a fixed list of combinations to create different organisms. If we wanted to have full flexibility for creating any combination we come up with while avoiding a ton of classes that support every single combination, we can make use of singleton methods:

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

We start with a plain Ruby `Object` and tailor it adding new methods. Instead of adding them one by one, we can add a group of them:

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

In this example, the `extend` method does the same thing as including a module.

Now, let's imagine that we want to group together animals and trees that share a section of the jungle and keep track of their biological classifications. Although they look like two different programming problems, they are quite similar— both of them aim to group a set of objects. Ideally, we could stablish different kinds of relationships between objects on the fly like this:

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

REVIEW: IM NOT SURE IF THIS IS WHAT YOU MEANT
!!!!!
!!!!!
=begin
The method `composite_of(:group)` would provide a method for including members to any `:group` we could think of by adding dynamic `add_sub_:group` methods to the class instances. In the same way, the method `member_of(:group)` would provide a method `parent_:group` that could leaf nodes so that they can know what group they are member of. It happens that all this is absolutely feasible with some meta programming:
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

`CompositeBase` is the base class for the rest of the components and implements the `member_of` and `composite_of` methods. By simply passing them the name of the group, they set up all the methods we need by constructing a string that defines them. The call to `class_eval` interprets the string in the context of the class, making them available.
