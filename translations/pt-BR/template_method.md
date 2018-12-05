# O Padrão de Método *Template*

## Problema
Nós temos um trecho de código complexo, mas em algum lugar um pedaço de
código precisa variar no meio desse trecho.

## Solução
A ideia geral do Padrão de Método *Template* é construir uma classe abstrata com
um método esqueleto, que guia o pedaço que precisa variar ao fazer chamadas para
métodos abstratos, os quais por sua vez são fornecidos pela subclasses concretas.
A classe abstrata controla o processamento de alto nível e as subclasses simplesmente
preenchem os detalhes.

## Exemplo
Nós temos que gerar um relatório em HTML, então nós inventamos algo parecido com
isso:

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

Mais tarde, nós percebemos que vamos precisar adicionar um novo formato: texto
simples. Fácil, nós podemos passar o formato como parâmetro e decidir o que
exibir baseado nisso:

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

Isso é meio bagunçado. O código que lida com ambos os formatos estão emaranhados
 e, pior ainda, não é nada extensível (e se nós quisermos adicionar um novo
 formato?). Vamos refatorar o código procurando pelo que permanece igual. Na
 maioria dos relatórios o fluxo básico é o mesmo, independentemente do formato:
 escrita do cabeçalho, escrita do título, escrita de cada linha do relatório e
 escrita de bordas e qualquer coisa necessária pelo formato. Nós podemos criar
 uma classe base abstrata que executa todos esses passos, mas deixa os detalhes
 para uma subclasse.

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
Nós agora podemos definir uma subclasse que implementa os detalhes:

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
