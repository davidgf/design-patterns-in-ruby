# Convention Over Configuration Pattern

## Problem
We want to build an extensible system without carrying the configuration burden.

## Solution
The **Convention Over Configuration** pattern suggests establishing some conventions based on class, method and file names, as well as a standard directory layout, instead of relying on configuration files.

## Example
Let's consider a message gateway that receives messages and forwards them to their destinations. A key requirement is that it must be easy to add new messaging protocols. A message would look like this:

```ruby
require 'uri'

class Message
  attr_accessor :from, :to, :body

  def initialize(from, to, body)
    @from = from
    @to = URI.parse(to)
    @body = body
  end
end
```

`to` is an `URI` representing the final destination of the message. It can be sent through a HTTP Post request via e-mail or to a file. To handle all these types of messages, we'll build an adapter for each protocol:

```ruby
require 'net/http'
class HttpAdapter
  def send(message)
    Net::HTTP.start(message.to.host, message.to.port) do |http|
      http.post(message.to.path, message.text)
    end
  end
end
```

We need to know what adapter we should use for a given message. We could pick the adapter in a `case` expression depending on the protocol, but this doesn't sound extensible at all. Instead, we'll define the first convention: **the adapter class name must be `<protocol>Adapter`**. This way, we can build the following method for picking the adapter we need:

```ruby
def adapter_for(message)
  protocol = message.to.scheme.downcase
  adapter_name = "#{protocol.capitalize}Adapter"
  adapter_class = self.class.const_get(adapter_name)
  adapter_class.new
end
```

We basically build the adapter's class name assuming it follows the convention and get the class with the `const_get` method. Now we can easily add the `FTP` protocol if we want to. However, we still have to deal with the problem of loading new adapters into our system. Instead of having a file where we require all the adapters of our application, we define the second convention: **put all the adapters in the adapter directory**. If all the adapters are in the same folder, we can dynamically load all of them with the following method:

```ruby
def load_adapters
  lib_dir = File.dirname(__FILE__)
  full_pattern = File.join(lib_dir, 'adapter', '*.rb')
  Dir.glob(full_pattern).each {|file| require file }
end
```

With these two conventions we make it really easy for other developers to add new, ready to use, protocols automatically in our application. The `MessageGateway` would look like this:

```ruby
class MessageGateway
  def initialize
    load_adapters
  end

  def process_message(message)
    adapter = adapter_for(message)
    adapter.send_message(message)
  end

  def adapter_for(message)
    protocol = message.to.scheme
    adapter_class = protocol.capitalize + 'Adapter'
    adapter_class = self.class.const_get(adapter_class)
    adapter_class.new
  end

  def load_adapters
    lib_dir = File.dirname(__FILE__)
    full_pattern = File.join(lib_dir, 'adapter', '*.rb')
    Dir.glob(full_pattern).each {|file| require file }
  end
end
```

Now we want to add some security into our platform by controlling which users are allowed to send messages to a given host. We'll have one authorization class per host with a generic method `authorized?` to check if the user can send the message. We'll also define our third convention that will help us create specific policies for certain users: **if the user has a special policy, it will be implemented in a method called <user>_authorized?**:

```ruby
class RussolsenDotComAuthorizer
  def russ_dot_olsen_authorized?(message)
    true
  end

  def authorized?(message)
    message.body.size < 2048
  end
end

def worm_case(string)
  tokens = string.split('.')
  tokens.map! {|t| t.downcase}
  tokens.join('_dot_')
end

def authorized?(message)
  authorizer = authorizer_for(message)
  user_method = worm_case(message.from) + '_authorized?'
  if authorizer.respond_to?(user_method)
    return authorizer.send(user_method, message)
  end
  authorizer.authorized?(message)
end
```

The last thing we can do if we want to let other developers extend our system is to provide them with some examples or (much better) with template generators that create the scaffold of a new adapter:

```ruby
protocol_name = ARGV[0]
class_name = protocol_name.capitalize + 'Adapter'
file_name = File.join('adapter', protocol_name + '.rb')

scaffolding = %Q{
  class #{class_name}
    def send_message(message)
      # Code to send the message
    end
  end
}

File.open(file_name, 'w') do |f|
  f.write(scaffolding)
end
```
