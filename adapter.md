# Adapter Pattern

## Problem
We want an object talk to another but their interfaces don't match.

## Solution
We simply wrap the **adaptee** with our new **adapter** class. This class implements an interface that the invoker understands, although all the work is performed by the adapted object.

## Example
Let's think of a class that encrypts a file, which takes two open files, one for reading and another one for writing:

```ruby
class Encrypter
  def initialize(key)
    @key = key
  end

  def encrypt(reader, writer)
    key_index = 0
    while not reader.eof?
      clear_char = reader.getc
      encrypted_char = clear_char ^ @key[key_index]
      writer.putc(encrypted_char)
      key_index = (key_index + 1) % @key.size
    end
  end
end
```

What happens if the data we want to secure happen to be in a string, rather than in a file? We need an object that looks like an open file, this is, supports the same interface as the Ruby `IO` object. We can create an `StringIOAdapter` to achieve so:

```ruby
class StringIOAdapter
  def initialize(string)
    @string = string
    @position = 0
  end

  def getc
    if @position >= @string.length
      raise EOFError
    end
    ch = @string[@position]
    @position += 1
    return ch
  end

  def eof?
    return @position >= @string.length
  end
end
```

Now we can use a `String` as if it was an open file (it only implements a small part of the `IO` interface, essentally what we need)

```ruby
encrypter = Encrypter.new('XYZZY')
reader= StringIOAdapter.new('We attack at dawn')
writer=File.open('out.txt', 'w')
encrypter.encrypt(reader, writer)
```
