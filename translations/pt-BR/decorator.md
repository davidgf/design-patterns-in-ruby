# O Padrão *Decorator*

## Problema
Nós precisamos mudar as responsabilidades de um objeto, adicionando algumas
funcionalidades.

## Solução
No Padrão *Decorator* (Decorador) nós criamos um objeto que envolve o objeto real e
implementa a mesma interface, encaminhando as chamadas de métodos. No entanto,
antes de delegar as mensagens ao o objeto real, ele executa a funcionalidade
adicional. Uma vez que todos os *decorators* implementam a mesma interface
central, nós podemos construir cadeias de *decorators* e montar uma combinação
de funcionalidades em tempo de execução.

## Exemplo
Abaixo está uma implementação de um objeto que simplesmente escrever uma linha
de texto em um arquivo:

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

Em algum momento, nós podemos precisar escrever o número da linha antes de cada
uma, ou uma *timestamp* ou um *checksum*. Nós poderíamos conseguir isso
adicionando métodos que fazem exatamente o que queremos nessa classe, ou
poderíamos criar uma subclasse para cada caso de uso. No entanto, nenhuma dessas
soluções é ótima. No caso da primeira, o cliente precisaria saber que tipo de
linha está sendo escrita o tempo todo. No caso da última, nós poderíamos acabar
tendo uma grande quantidade de subclasses, principalmente se nós quisermos
combinar novas funcionalidades. Então, vamos criar uma classe *decorator* base
que implementa a mesma interface que nossa classe de escrita e delega para ela:

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
Agora, ao estender a classe `WriteDecorator`, nós podemos adicionar
funcionalidades extras em cima do escritor básico, como adicionar numeração:

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
Se criarmos *decorators* que implementam a mesma interface central, nós podemos
encadear eles:

```ruby
class TimeStampingWriter < WriterDecorator
  def write_line(line)
    @real_writer.write_line("#{Time.new}: #{line}")
  end
end

writer = CheckSummingWriter.new(TimeStampingWriter.new(
            NumberingWriter.new(SimpleWriter.new('final.txt'))))
writer.write_line('Hello out there')
```
