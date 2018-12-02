# Padrão de Linguagem de Domínio Específico

## Problema
Nós queremos construir uma sintaxe  conveniente para resolver problemas de um
domínio específico.

## Solução
Ruby é realmente flexível e tem sintaxe amistosa, algumas vezes, ler um pedaço
de código em Ruby até parece como ler um livro. O Padrão de Linguagem de Domínio
Específico (**DSL - Domain-Specific Language**) se aproveita disso e sugere a
criação de uma linguagem em cima do Ruby. Ele começa construindo estruturas de
dados que armazenam infomações sobre as tarefas que precisam ser executadas.
Então, ele define diversos métodos de alto nível que auxiliam a DSL e permitem
que o usuário final construa um programa com a linguagem criada. Finalmente,
o programa é avaliado e interpretado como chamadas a métodos Ruby.

## Exemplo
Nós queremos fornecer um jeito simples de criar backups periódicos de certas
pastas. Nós podemos facilmente fazer isso em Ruby, mas nossos usuários finais
podem não saber nada sobre programação, então nós iremos construir uma DSL para
eles. Primeiramente, nós precisamos configurar algumas estruturas de dados:

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

A classe `Backup` contém informações sobre os diretórios que precisam de cópias
de segurança e um método para realizar o backup. A classe `DataSource` é um
recipiente para um caminho para um diretório e tem capacidade de encontrar
arquivos. A única coisa que falta são os métodos de alto nível que farão uso
de nossas estruturas de dados e irão permitir que o usuário final defina seu
programa:

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

É um linguagem bem simples: o método `backup` configura os diretórios que
precisam de cópias de segurança, o método `to` configura a pasta de destino da
cópia, e o método `interval` configura quão frequentemente as cópias serão
realizadas. Então, o programa criado pelo usuário final é lido e interpretado
como código Ruby com `eval`. Como se parece um programa criado com a nossa DSL?
Bem simples:

```ruby
backup '/home/russ/documents'

backup '/home/russ/music', file_name('*.mp3') & file_name('*.wav')

backup '/home/russ/images', except(file_name('*.tmp'))

to '/external_drive/backups'

interval 60
```
