# Domain-Specific Language Pattern

## Problem
We want to build a convenient syntax for solving problems of a specific domain.

## Solution
Ruby is a really flexible an has human friendly syntax, sometimes reading a piece of Ruby code even feels like reading a book. **The Domain-Specific Language (DSL)** pattern takes advantage of this and suggest building a new language on top of Ruby. It starts by building data structures, which hold the information about the tasks to be performed. Then, it defines several top-level methods that support the DSL and which the end user will use to build a program with the newly created language. Finally, the program is evaluated and interpreted as Ruby method calls.

## Example
We want to provide an simple way to create periodic backups of certain folders. We can easily do so with Ruby, but our end users might not know anything about programming, so we'll build a DSL for tat matter. First, we need to set up some data structures:

```ruby
class Backup
  include Singleton

  attr_accessor :backup_directory, :interval
  attr_reader :data_sources
  
  def initialize
    @data_sources = []
    @backup_directory = '/backup'
    @interval = 60
  end

  def backup_files
    this_backup_dir = Time.new.ctime.tr(' :','_')
    this_backup_path = File.join(backup_directory, this_backup_dir)
    @data_sources.each {|source| source.backup(this_backup_path)}
  end

  def run
    while true
      backup_files
      sleep(@interval*60)
    end
  end
end

class DataSource
  attr_reader :directory, :finder_expression

  def initialize(directory, finder_expression)
    @directory = directory
    @finder_expression = finder_expression
  end

  def backup(backup_directory)
    files=@finder_expression.evaluate(@directory)
    files.each do |file|
      backup_file( file, backup_directory)
    end
  end

  def backup_file( path, backup_directory)
    copy_path = File.join(backup_directory, path)
    FileUtils.mkdir_p( File.dirname(copy_path) )
    FileUtils.cp( path, copy_path)
  end
end
```

The `Backup` class holds the information about the directories to be backed up and a method to perform it, while the `DataSource` class is a container for a path to a directory and has file finding capabilities. The only thing left is the top-level methods that will make use of the data structures and will let the end user define his program:

```ruby
def backup(dir, find_expression=All.new)
  Backup.instance.data_sources << DataSource.new(dir, find_expression)
end

def to(backup_directory)
  Backup.instance.backup_directory = backup_directory
end

def interval(minutes)
  Backup.instance.interval = minutes
end

eval(File.read('backup.pr'))
Backup.instance.run
```

It's a pretty simple language: with the `backup` method we set the directories to be backed up, with `to` the destination folder of the copy and with `interval` how often the copy will be performed. Then, the program created by the end user is read and interpreted as Ruby code with `eval`. How does a program program created with our DSL look like? Pretty simple:

```ruby
backup '/home/russ/documents'

backup '/home/russ/music', file_name('*.mp3') & file_name('*.wav')

backup '/home/russ/images', except(file_name('*.tmp'))

to '/external_drive/backups'

interval 60
```
