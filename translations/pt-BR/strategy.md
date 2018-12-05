# O Padrão *Strategy*

## Problema
Nós precisamos modificar parte de um algoritmo, algo que resolvemos
anteriormente usando o padrão de Método *Template*. Contudo nós queremos
evitar suas desvantagens, introduzidas pelo fato dele ser construído ao redor de
herança.

## Solução
Para evitar os problemas introduzidos pela herança, nós devemos usar delegação.
Ao invés de criar subclasses (como no padrão de Método *Template*), nós
despedaçamos as partes do código que variam e as isolamos em suas próprias
classes. Criamos uma para cada variação. A ideia principal do padrão *Strategy*
é definir uma família de objetos (estratégias), onde todos fazem (quase) a mesma
coisa e suportam a mesma interface. Então, o usuário da estratégia (contexto)
pode tratar as estratégias como partes substituíveis.

## Exemplo
Seguindo o exemplo do padrão de Método *Template*, nós podemos refatorar o código
de modo que cada formato seja uma classe (estratégia), ao invés de uma subclasse.
Dessa forma, a classe `Report` se tornaria muito mais simples, ela iria desempenhar
o papel de objeto de contexto e seria providenciada com a estratégia.

```ruby
class Report
  attr_reader :title, :text
  attr_accessor :formatter

  def initialize(formatter)
    @title = 'Monthly Report'
    @text = ['Things are going', 'really, really well.']
    @formatter = formatter
  end

  def output_report
    @formatter.output_report(self)
  end
end
```
A implementação de cada formato teria sua própria classe, o que nos permite
alcançar um separação melhor de responsabilidades.

```ruby
class HTMLFormatter
  def output_report(context)
    puts('<html>')
    puts(' <head>')
    puts("<title>#{context.title}</title>")
    puts(' </head>')
    puts(' <body>')
    context.text.each do |line|
      puts("<p>#{line}</p>")
    end
    puts(' </body>')
    puts('</html>')
  end
end

class PlainTextFormatter
  def output_report(context)
    puts("***** #{context.title} *****")
    context.text.each do |line|
      puts(line)
    end
  end
end
```
Para utilização, nós precisamos apenas fornecer um objeto formatador (estratégia)
ao objeto da classe `Report` (contexto). *Report* significa relatório.

```ruby
report = Report.new(HTMLFormatter.new)
report.output_report

report.formatter = PlainTextFormatter.new
report.output_report
```
