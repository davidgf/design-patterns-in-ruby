# O Padrão *Command*

## Problema
Nós queremos realizar alguma tarefa específica sem saber como todo o processo
funciona ou ter qualquer informação sobre o destinatário da requisição.

## Solução
O Padrão *Command* (Comando) desacopla o objeto que precisa realizar uma tarefa
específica daquele que sabe como fazer. Ele encapsula toda a informação necessária
para fazer o trabalho em seu próprio objeto incluindo: quem são os destinatários,
os métodos que serão chamados e seus parâmetros. Dessa forma, qualquer objeto
que quiser realizar a tarefa apenas precisa conhecer a interface do objeto comando.

## Exemplo
Vamos considerar uma implementação de um botão de algum framework GUI, o qual tem
um método chamado durante um clique de um botão.

```ruby
class SlickButton

  # Lots of button drawing and management
  # code omitted...

  def on_button_push
    # Do something when the button is pushed
  end
end
```

Nós podemos estender a classe botão que sobrescreve o método `on_button_push`
para realizar certas ações sempre que o usuário clique no botão. Por exemplo,
se o objetivo do botão é salvar um documento, poderíamos fazer algo assim:  

```ruby
class SaveButton < SlickButton
  def on_button_push
    # Save the current document...
  end
end
```

No entanto, uma GUI complexa poderia ter centenas de botões, o que significa que
nós poderíamos acabar tendo centenas de subclasses para nossos botões. Existe
uma maneira mais fácil. Nós podemos refatorar o código que executa a ação em seu
 próprio objeto, o qual implementa uma interface simples. Então, nós podemos
refatorar a implementação de nosso botão para receber um objeto comando como
parâmetro e chama-lo quando clicado.

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

O padrão *Command* é muito útil se nós precisarmos implementar uma funcionalidade
 do tipo **desfazer/voltar**. Tudo que precisamos fazer é implementar o método
 `unexecute` em nosso objeto comando. Por exemplo, assim é como nós iríamos
 implementar a criação de um arquivo:

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
Outra situação útil onde o padrão **Command** é muito conveniente é em programas
de instalação. Ao combinar ele com o padrão **Composite** (Composto), nós podemos
guardar uma lista de tarefas que precisem ser realizadas.

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
