# Proxy Pattern

## Problem
We want to have more control over how and when we access a certain object.

## Solution
With the proxy pattern we create an object, **proxy**, that has a reference to the real object we want to access. Then, whenever the client calls the proxy, it simply forwards the request to the real one. There are three main scenarios where this pattern might be useful:
* **Protection Proxy**: before delegating calls to the real object, it adds a layer of security. A big advantage of this approach is that it gives us separation of concerns, as the proxy takes care of access control, while the real object is only concerned about business logic.
* **Remote Proxy**: when the object we want to use is in another machine and it should be fetched across the network, the proxy handles all the connection complexity, while the client can use the object as if it was in the same machine.
* **Virtual Proxy**: it delays the creation of an object until it is used.

## Example
Let's consider a `BankAccount` object:

```ruby
class BankAccount
  attr_reader :balance

  def initialize(starting_balance=0)
    @balance = starting_balance
  end

  def deposit(amount)
    @balance += amount
  end

  def withdraw(amount)
    @balance -= amount
  end
end
```

If we wanted to control who accesses it, we could build a proxy object that introduces some checks before delegating calls to the bank account:

```ruby
require 'etc'

class AccountProtectionProxy
  def initialize(real_account, owner_name)
    @subject = real_account
    @owner_name = owner_name
  end

  def deposit(amount)
    check_access
    return @subject.deposit(amount)
  end

  def withdraw(amount)
    check_access
    return @subject.withdraw(amount)
  end

  def balance
    check_access
    return @subject.balance
  end

  def check_access
    if Etc.getlogin != @owner_name
      raise "Illegal access: #{Etc.getlogin} cannot access account."
    end
  end
end
```

Maybe what we want to do is create the bank account object only when it's really needed. We could create a proxy that initializes the real object only when one of its methods is called and keep a cached copy for further calls:

```ruby
class VirtualAccountProxy
  def initialize(starting_balance=0)
    @starting_balance=starting_balance
  end

  def deposit(amount)
    s = subject
    return s.deposit(amount)
  end

  def withdraw(amount)
    s = subject
    return s.withdraw(amount)
  end

  def balance
    s = subject
    return s.balance
  end

  def subject
    @subject ||= BankAccount.new(@starting_balance)
  end
end
```
