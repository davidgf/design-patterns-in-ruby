# Command Pattern

## Problem
We want to perform some specific task without knowing how the whole process is or having any information about the receiver of the request. 

## Solution
The Command pattern decouples the object that needs to perform a specific task from the one that knows how to do it. It encapsulates all the needed information to do the job into its own object, including who the receiver(s) is(are), the methods to invoke and the parameters. That way, any object that wants to perform the task only needs to know about the command object interface.

## Example
Let's consider a button implementation of some GUI framework, which has a method called upon button click.

```ruby
class SlickButton

  # Lots of button drawing and management
  # code omitted...

  def on_button_push
    # Do something when the button is pushed
  end
end
```

So, we could extend the button class overriding the `on_button_push` method to perform certain actions whenever a user clicks it. For example, if the button's purpose is saving a document, we could do something like this:

```ruby
class SaveButton < SlickButton
  def on_button_push
    # Save the current document...
  end
end
```

However, a complex GUI could have hundreds of buttons, which means that we would end up having several hundreds of subclasses of our button. There is an easier way. We can factor out the code that performs the action into its own object, which implements a simple interface. Then, we can refactor our button's implementation to receive the command object as a parameter and calling it when it's clicked.

```ruby
class SaveCommand
  def execute
    # Save the current document...
  end
end

class SlickButton
  attr_accessor :command

  def initialize(command)
    @command = command
  end

  def on_button_push
    @command.execute if @command
  end
end

save_button = SlickButton.new(SaveCommand.new)
```

The Command pattern is pretty useful if we need to implement **undo** feature. All we need to do is implementing the `unexecute` method in our command object. For example, this is how we would implement the task of creating a file:

```ruby
class CreateFile < Command
  def initialize(path, contents)
      super "Create file: #{path}"
      @path = path
      @contents = contents
    end

  def execute
    f = File.open(@path, "w")
    f.write(@contents)
    f.close
  end

  def unexecute
    File.delete(@path)
  end
end
```

Another situation where the command pattern is really handy is in installation programs. Combining it with the **composite** pattern, we can store a list of tasks to be performed:

```ruby
class CompositeCommand < Command
  def initialize
    @commands = []
  end

  def add_command(cmd)
    @commands << cmd
  end

  def execute
    @commands.each {|cmd| cmd.execute}
  end
end

cmds = CompositeCommand.new
cmds.add_command(CreateFile.new('file1.txt', "hello world\n"))
cmds.add_command(CopyFile.new('file1.txt', 'file2.txt'))
cmds.add_command(DeleteFile.new('file1.txt'))
```
