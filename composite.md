# Composite Pattern

## Problem
We need to build a hierarchy of tree objects and want to interact with all them the same way, regardless of whether it's a leaf object or not.

## Solution
There are three main classes in the Composite pattern: the **component**, the **leaf** and the **composite** classes. The **component** is the base class and defines a common interface for all the components. The **leaf** is an indivisible building block of the process. The **composite** is a higher-level component built from subcomponents, so it fulfills a dual role: is a component and a collection of components. As both composite and leaf classes implement the same interface, the can be used the same way.

## Example
We've been asked to build a system that keeps track of the manufacturing of cakes, being a key requirement being able to know how long it takes the task of baking it. Making a cake is a complicated process, as it involves multiple tasks that might be composed of different subtasks. The whole process could be represented in the following tree:

```
|__ Manufacture Cake
    |__ Make Cake
    |   |__ Make Batter
    |   |   |__ Add Dry Ingredients
    |   |   |__ Add Liquids
    |   |   |__ Mix
    |   |__ Fill Pan
    |   |__ Bake
    |   |__ Frost
    |
    |__ Package Cake
        |__ Box
        |__ Label
```

In the Composite pattern, we'll model every step in a separate class with a common interface, which will report back how long they take. So we'll define a common base class, `Task`, which plays the role of **component**.

```ruby
class Task
  attr_accessor :name, :parent

  def initialize(name)
    @name = name
    @parent = nil
  end

  def get_time_required
    0.0
  end
end
```

We can now create the classes in charge of the most basic jobs, this is, **leaf** classes, like `AddDryIngredientsTask`:

```ruby
class AddDryIngredientsTask < Task
  def initialize
    super('Add dry ingredients')
  end

  def get_time_required
    1.0
  end
end
```

What we need now is a container to deal with complex tasks, which are internally built up of any number of subtasks, but from the outside look like any other `Task`. We'll create the **composite** class:

```ruby
class CompositeTask < Task
  def initialize(name)
    super(name)
    @sub_tasks = []
  end

  def add_sub_task(task)
    @sub_tasks << task
    task.parent = self
  end

  def remove_sub_task(task)
    @sub_tasks.delete(task)
    task.parent = nil
  end

  def get_time_required
    time = 0.0
    @sub_tasks.each {|task| time += task.get_time_required}
    time
  end
end
```

With this base class we can build complex tasks that behave like a simple one, as it implements the `Task` interface, and also add subtasks with the method `add_sub_task`. We'll create the `MakeBatterTask`

```ruby
class MakeBatterTask < CompositeTask
  def initialize
    super('Make batter')
    add_sub_task(AddDryIngredientsTask.new)
    add_sub_task(AddLiquidsTask.new)
    add_sub_task(MixTask.new)
  end
end
```

We must keep in mind that the objects tree may go as deep as we want. `MakeBatterTask` contains only leaf objects, but we could create a class that contains composite objects and it would behave exactly the same:

```ruby
class MakeCakeTask < CompositeTask
  def initialize
    super('Make cake')
    add_sub_task(MakeBatterTask.new)
    add_sub_task(FillPanTask.new)
    add_sub_task(BakeTask.new)
    add_sub_task(FrostTask.new)
    add_sub_task(LickSpoonTask.new)
  end
end
```
