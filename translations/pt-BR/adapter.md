# O Padrão *Adapter*

## Problema
Nós queremos conversar que um objeto converse com outro objeto, mas as interfaces
deles não combinam.

## Solução
Nós simplesmente envolvemos o **adaptado** com a nossa nova classe **adaptadora**.
Essa classe implementa uma interface que o invocador compreenda, embora todo o
trabalho seja realizado pelo objeto adaptado.

## Exemplo
Vamos pensar em uma classe que receba dois arquivos (um leitor e um escritor) e
criptografe um arquivo.

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
Mas o que acontece se a informação que queremos deixar segura estiver em uma
string, ao invés de em um arquivo? Nós precisamos de um objeto que se parece
com um arquivo, ou seja, suporte a mesma interface que um objeto de `IO` do
Ruby. Podemos criar uma classe adaptadora `StringIOAdapter` para alcançar isso:

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

Agora podemos usar uma `String` como se fosse um arquivo, apenas uma pequena
parte da interface `IO` é implementada, basicamente o que precisamos.

```ruby
encrypter = Encrypter.new('XYZZY')
reader= StringIOAdapter.new('We attack at dawn')
writer=File.open('out.txt', 'w')
encrypter.encrypt(reader, writer)
```
