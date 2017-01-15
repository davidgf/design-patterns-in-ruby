# Iterator Pattern

## Problem
We have an aggregate object and we want to provide a way to access its collection of sub-objects, without exposing its underlaying representation.

## Solution
There are two proposed solutions: **external iterators** and **internal iterators**.

### External Iterators
The iterator is a separate object from the aggregate, which is passed in as an argument to initialize the iterator. The iterator keeps a reference to the current index and provides with an interface to ask if there are items left, to get the current item and the next one.

### Internal Iterators
With internal iterators we use a code block to pass the logic down into the aggregate. A really good example of this approach is the `Array` method `each`.

## Example
An example of external iterator for a Ruby array might look something like this:

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

Thanks to duck typing, this implementation would work on any class that has the `length` method and can be indexed by an integer, such as `String`.
To create an aggregate class with an internal iterator, we'll make use of the `Enumerable` mixin module. We only have to make sure that our iterator method is named `each` and we implement the comparison operator `<=>`. By doing this, we automatically get a great number of handy methods, like `include?`, `all?` or `sort`. As an example, we'll consider two classes: `Account` and `Portfolio`, which manages multiple accounts.

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
