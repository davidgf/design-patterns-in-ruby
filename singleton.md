# Singleton Pattern

## Problem
We need to have a single instance of certain class across the whole application.

## Solution
In the **Singleton** pattern the access to the constructor is restricted, so that it cannot be instantiated. So, the creation of the single instance is done inside the class and is held is a class variable, and it can be accessed through a getter across the application.

## Example
Let's consider the implementation of a logger class:

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

Logging is a feature used across the whole application, so it makes sense that there should exist only a single instance of the logger. We can make sure that nobody instantiate twice the `SimpleLogger` class by making its constructor private:

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

We can get the same behavior by including the `Singleton` module, so that we can avoid duplicating code if we create several singletons:

```ruby
require 'singleton'

class SimpleLogger
  include Singleton
  # Lots of code deleted...
end
```
