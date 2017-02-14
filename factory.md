# Factory Pattern

## Problem
We need to create objects without having to specify the exact class of the object that will be created.

## Solution
The **Factory** pattern is a specialization of the [Template](template.md) pattern. We start by creating a generic base class where we don't make the "which class" decision. Instead, whenever it needs to create a new object, it calls a method that is defined in a subclass. So, depending on the subclass we use (**factory**), we create objects of one class or another (**products**).

## Example
Imagine that you are asked to build a simulation of life in a pond, which has plenty of ducks:

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

But, how would we model our `Pond` if we wanted to have frogs instead of ducks? In the implementation above, we are specifying in the `Pond`'s initializer that it should be filled up with ducks. So, we'll refactor it so that the decision of creating one type of animal or another is made in a subclass:

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
