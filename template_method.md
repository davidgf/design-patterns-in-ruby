# Template Method Pattern

## Problem
We have a complex bit of code, but somewhere in the middle there is a bit that needs to vary.

## Solution
The general idea of the Template Method pattern is to build an abstract base class with a skeletal method, which drives the bit of the processing that needs to vary by making calls to abstract methods, which are then supplied by the concrete subclasses. So, the abstract base class controls the higher-level processing and the sub-classes simply fill in the details.

## Example

We have to generate a HTML report, so we come up with something like this:

```ruby
class Report
  def initialize
    @title = 'Monthly Report'
    @text = ['Things are going', 'really, really well.']
  end

  def output_report
    puts('<html>')
    puts(' <head>')
    puts("<title>#{@title}</title>")
    puts(' </head>')
    puts(' <body>')
    @text.each do |line|
      puts("<p>#{line}</p>")
    end
    puts(' </body>')
    puts('</html>')
  end
end
```

Later on, we realize that we must add a new format: plain text. Easy, we can pass format as a parameter and decide what to display based on it:

```ruby
class Report
  def initialize
    @title = 'Monthly Report'
    @text = ['Things are going', 'really, really well.']
  end

  def output_report(format)
    if format == :plain
      puts("*** #{@title} ***")
    elsif format == :html
      puts('<html>')
      puts(' <head>')
      puts("<title>#{@title}</title>")
      puts(' </head>')
      puts(' <body>')
    else
      raise "Unknown format: #{format}"
    end
    @text.each do |line|
      if format == :plain
        puts(line)
      else
        puts("<p>#{line}</p>")
      end
    end
    if format == :html
      puts(' </body>')
      puts('</html>')
    end
  end
end
```

That's kind of messy, the code that handles both formats is tangled up and, even worse, it's not extensible at all (what if we want to add a new format?). Let's refactor the code looking for what stay the same. In most reports the basic flow is the same, regardless of the format: output header, output title, output each line of the report and output any trailing stuff required by the format. We could create an abstract base class that performs all those steps but leaves the details to a subclass:

```ruby
class Report
  def initialize
    @title = 'Monthly Report'
    @text = ['Things are going', 'really, really well.']
  end

  def output_report
    output_start
    output_head
    output_body_start
    output_body
    output_body_end
    output_end
  end

  def output_body
    @text.each do |line|
      output_line(line)
    end
  end

  def output_start
    raise 'Called abstract method: output_start'
  end

  def output_head
    raise 'Called abstract method: output_head'
  end

  def output_body_start
    raise 'Called abstract method: output_body_start'
  end

  def output_line(line)
    raise 'Called abstract method: output_line'
  end

  def output_body_end
    raise 'Called abstract method: output_body_end'
  end

  def output_end
    raise 'Called abstract method: output_end'
  end
end
```

We can now define a subclass that implements the details

```ruby
class PlainTextReport < Report
  def output_start
  end

  def output_head
    puts("**** #{@title} ****")
  end

  def output_body_start
  end

  def output_line(line)
    puts(line)
  end

  def output_body_end
  end

  def output_end
  end
end
```

```ruby
report = PlainTextReport.new
report.output_report
```
