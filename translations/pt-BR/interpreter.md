# O Padrão *Interpreter* (Tradutor)

## Problema
Nós precisamos de uma linguagem especializada para resolver um problema bem
definido de um domínio conhecido.

Nós precisamos de uma linguagem especializada para resolver um problema de
nenhum (como em nulo/*nil*) domínio.

## Solução
Tradutores normalmente trabalham em duas fases: **parsing** (análise) e **evaluating** (avaliação).
O *parser* faz a leitura dos dados e cria uma estrutura chamada árvore abstrata de sintaxe (**AST - Abstract Syntax Tree**), que contém as mesmas informações, mas representada por uma árvore de objetos. Então, a *AST* é avaliada contra condições externas.
Na *AST*, os nós folhas (**terminais**) são os blocos de construção mais básicos da linguagem. Os nós não-folhas (**não terminais**) representam conceitos de ordens mais altas da linguagem. Depois de fornecer as condições externas (**contexto**), a AST é avaliada recursivamente.

## Exemplo
Vamos considerar uma ferramenta de busca de arquivos que nós gostaríamos de usar com uma linguagem simples de consulta. A operação mais básica é retornar todos os arquivos e nós poderíamos implementar dessa forma:

```ruby
require 'find'

class Expression
  # Common expression code will go here soon...
end

class All < Expression
  def evaluate(dir)
    results= []
    Find.find(dir) do |p|
      next unless File.file?(p)
      results << p
    end
    results
  end
end
```

Além disso, nós iriamos precisar buscar arquivos os quais o nomes correspondem a um certo padrão:

```ruby
class FileName < Expression
  def initialize(pattern)
    @pattern = pattern
  end

  def evaluate(dir)
    results= []
    Find.find(dir) do |p|
      next unless File.file?(p)
      name = File.basename(p)
      results << p if File.fnmatch(@pattern, name)
    end
    results
  end
end

expr_all = All.new
files = expr_all.evaluate('test_dir')
```

Algumas outras operações interessantes seriam buscar por arquivos maiores do que um determinado tamanho ou
pesquisar por arquivos graváveis.

```ruby
class Bigger < Expression
  def initialize(size)
    @size = size
  end

  def evaluate(dir)
    results = []
    Find.find(dir) do |p|
      next unless File.file?(p)
      results << p if( File.size(p) > @size)
    end
    results
  end
end

class Writable < Expression
  def evaluate(dir)
    results = []
    Find.find(dir) do |p|
      next unless File.file?(p)
      results << p if( File.writable?(p) )
    end
    results
  end
end
```

Essas operações básicas são os **terminais** de nossa AST. Vamos construir o primeiro **não terminal** que nega a operação:

```ruby
class Not < Expression
  def initialize(expression)
    @expression = expression
  end

  def evaluate(dir)
    All.new.evaluate(dir) - @expression.evaluate(dir)
  end
end
```

Agora podemos encontrar arquivos que não são graváveis, por exemplo:

```ruby
expr_not_writable = Not.new( Writable.new )
readonly_files = expr_not_writable.evaluate('test_dir')
```

Se ao invés de negar a operação nós quiséssemos combinar os resultados de duas operações ou obter arquivos que
se encaixam em duas condições, nós poderíamos construir os não terminais **OR** (OU) e **AND** (E).

```ruby
class Or < Expression
  def initialize(expression1, expression2)
    @expression1 = expression1
    @expression2 = expression2
  end

  def evaluate(dir)
    result1 = @expression1.evaluate(dir)
    result2 = @expression2.evaluate(dir)
    (result1 + result2).sort.uniq
  end
end

class And < Expression
  def initialize(expression1, expression2)
    @expression1 = expression1
    @expression2 = expression2
  end

  def evaluate(dir)
    result1 = @expression1.evaluate(dir)
    result2 = @expression2.evaluate(dir)
    (result1 & result2)
  end
end

big_or_mp3_expr = Or.new( Bigger.new(1024), FileName.new('*.mp3') )
big_or_mp3s = big_or_mp3_expr.evaluate('test_dir')
```

Você deve ter percebido que o **Interpreter** também implementa o padrão **Composite**, o que nos fornece flexibilidade para adicionar operações ou combinar as existentes:

```ruby
complex_expression = And.new(
                      And.new(Bigger.new(1024), FileName.new('*.mp3')),
                      Not.new(Writable.new ))
complex_expression.evaluate('test_dir')
```
