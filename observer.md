# Observer Pattern

## Problem
We want to build a system that is highly integrated, that is, a system where every part is aware of the state of the whole. We also want it to be maintainable, so we should avoid coupling between classes.

## Solution
If we want some component (observer) to know about the activities of another one (subject), we could simply hard-wire both classes and inform the former upon some actions performed on the latter. This means that we should pass a reference to the observer when we create the subject, and call some of its methods when the latter changes. However, in this approach we are doing something we want to avoid: increasing coupling. What is more, if we wanted to inform some other observer, we should modify the implementation of the subject so that it notifies it, even though nothing has changed. A much better approach is keeping a list of objects interested in the subject changes and defining a clean interface between the source of the news (the subject) and the consumers (the observers). That way, whenever there's some change on the subject, we just need to iterate over the list of observers and notify them using the interface we defined.

## Example
Let's consider a `Employee` object that has a `salary` property. We'd like to be able to change his salary as well as keeping informed the payroll system about such modifications. The simplest way to achieve this is passing a reference to payroll and inform it whenever we modify the employee `salary`:

```ruby
class Employee
  attr_reader :name, :title
  attr_reader :salary

  def initialize( name, title, salary, payroll)
    @name = name
    @title = title
    @salary = salary
    @payroll = payroll
  end

  def salary=(new_salary)
    @salary = new_salary
    @payroll.update(self)
  end
end
```

The problem is that if we want to notify somebody else (`TaxMan`, for instance), we should modify the `Employee`. This means that other classes are driving the changes to `Employee`, even though nothing has changed there. Let's provide a way to keep a list of interested objects on salary changes.

```ruby
class Employee
  attr_reader :name, :title
  attr_reader :salary

  def initialize( name, title, salary)
    @name = name
    @title = title
    @salary = salary
    @payroll = payroll
    @observers = observers
  end

  def salary=(new_salary)
    @salary = new_salary
    notify_observers
  end

  def add_observer(observer)
    @observers << observer
  end

  def delete_observer(observer)
    @observers.delete(observer)
  end

  def notify_observers
    @observers.each do |observer|
      observer.update(self)
    end
  end
end
```

Now we can use the `add_observer` method to add as many objects as we want to the observers list, and all of them will be notified whenever the salary changes.

```ruby
fred = Employee.new('Fred', 'Crane Operator', 30000.0)

payroll = Payroll.new
fred.add_observer(payroll)

tax_man = TaxMan.new
fred.add_observer(tax_man)

fred.salary=35000.0
```

Even though we can implement the pattern by ourselves, the Ruby standard library has a prebuilt module that let us make any of our objects observable, freeing us from defining the same methods everywhere. Let's refactor the `Employee` class to use the `Observable` module:

```ruby
require 'observer'

class Employee
  include Observable

  attr_reader :name, :address
  attr_reader :salary

  def initialize( name, title, salary)
    @name = name
    @title = title
    @salary = salary
  end

  def salary=(new_salary)
    @salary = new_salary
    changed
    notify_observers(self)
  end
end
```
