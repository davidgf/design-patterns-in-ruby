# Decorator Pattern

## Problem
We need to vary the responsibilities of an object; adding some features.

## Solution
In the **Decorator** pattern we create an object that wraps the real one, and implements the same interface and forwarding method calls. However, before delegating to the real object, it performs the additional feature. Since all decorators implement the same core interface, we can build chains of decorators and assemble a combination of features at runtime.

## Example
Below is an implementation of an object that simply writes a text line to a file:

```ruby
class SimpleWriter
  def initialize(path)
    @file = File.open(path, 'w')
  end

  def write_line(line)
    @file.print(line)
    @file.print("\n")
  end

  def pos
    @file.pos
  end

  def rewind
    @file.rewind
  end

  def close
    @file.close
  end
end
```

At some point, we might need to print the line number before each one, or a timestamp or a checksum. We could achieve this by adding new methods to the class that performs exactly what we want, or by creating a new subclasses for each use case. However, none of these solutions is optimal. In the case of the former, the client should know what kind of line is printing all the time. In the case of the latter, we could end up having a huge amount of subclasses, especially if we want to combine the new features. So, let's create a decorator base class that implements the same interface that our writer class and delegates on it:

```ruby
class WriterDecorator
  def initialize(real_writer)
    @real_writer = real_writer
  end

  def write_line(line)
    @real_writer.write_line(line)
  end

  def pos
    @real_writer.pos
  end

  def rewind
    @real_writer.rewind
  end

  def close
    @real_writer.close
  end
end
```

Now, by extending the `WriteDecorator` class, we can add extra features on top of the basic writer, like adding numbering:

```ruby
class NumberingWriter < WriterDecorator
  def initialize(real_writer)
    super(real_writer)
    @line_number = 1
  end

  def write_line(line)
    @real_writer.write_line("#{@line_number}: #{line}")
    @line_number += 1
  end
end

writer = NumberingWriter.new(SimpleWriter.new('final.txt'))
writer.write_line('Hello out there')
```

If we create new decorators that implement the same core interface, we can chain them:

```ruby
class TimeStampingWriter < WriterDecorator
  def write_line(line)
    @real_writer.write_line("#{Time.new}: #{line}")
  end
end

writer = TimeStampingWriter.new(
            NumberingWriter.new(SimpleWriter.new('final.txt')))
writer.write_line('Hello out there')
```
