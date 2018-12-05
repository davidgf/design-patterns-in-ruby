# O Padrão *Composite*

## Problema
Nós precisamos construir uma hierarquia de objetos árvore e queremos interagir
com todos eles da mesma forma, independentemente se são um objeto folha ou não.

## Solução
Existem três classes principais no padrão *Composite* (Composto): o
**componente** , a **folha** e as classes **compostas**.
O **componente** é a classe base e define uma interface comum para todos os
componentes. A **folha** é um bloco de construção indivisível do processo.
O **composto** é um componente de alto nível construído de subcomponentes,
então ele preenche um papel duplo: ele é um componente e uma coleção de
componentes. Como ambas as classes do objeto composto e da folha implementam
a mesma interface, eles podem ser usados da mesma forma.

## Exemplo
Fomos solicitados para criação de um sistema que rastreia a fabricação de bolos,
com um requerimento chave de ser capaz de sabermos quanto tempo ele leva para ser
assado. Fazer um bolo é um processo complicados, uma vez que envolve múltiplas
tarefas que podem ser compostas de subtarefas. O processo inteiro poderia ser
representado pela seguinte árvore:

```
|__ Fabricar Bolo
    |__ Fazer Bolo
    |   |__ Fazer Massa
    |   |   |__ Adicionar Ingredientes Secos
    |   |   |__ Adicionar Líquidos
    |   |   |__ Misturar
    |   |__ Preencher Panela
    |   |__ Cozinhar
    |   |__ Congelar
    |
    |__ Empacotar Bolo
        |__ Caixa
        |__ Rótulo
```

No Padrão *Composite*, nós vamos modelar cada passo em uma classe separada com
uma interface em comum que vai reportar de volta quanto tempo ela levou.
Então vamos definir uma classe base em comum, chamada `Task` (Tarefa), que
desempenha o papel de **componente**.

```ruby
class Task
  attr_accessor :name, :parent

  def initialize(name)
    @name = name
    @parent = nil
  end

  def get_time_required
    0.0
  end
end
```
Agora nós podemos criar as classes responsáveis por criar os trabalhos mais
básicos (classes **folha**) como `AddDryIngredientsTask`
(TarefaAdicionarIngredientesSecos):

```ruby
class AddDryIngredientsTask < Task
  def initialize
    super('Add dry ingredients')
  end

  def get_time_required
    1.0
  end
end
```

O que nós precisamos agora é de um contêiner para lidar com tarefas complexas
que são internamente construídas a partir de qualquer número de subtarefas, mas
se parecem com qualquer outra `Task` por fora. Nós vamos criar a classe
**composta**:

```ruby
class CompositeTask < Task
  def initialize(name)
    super(name)
    @sub_tasks = []
  end

  def add_sub_task(task)
    @sub_tasks << task
    task.parent = self
  end

  def remove_sub_task(task)
    @sub_tasks.delete(task)
    task.parent = nil
  end

  def get_time_required
    @sub_tasks.inject(0.0) {|time, task| time += task.get_time_required}
  end
end
```
Com essa classe base nós podemos construir tarefas complexas que se comportam
como uma tarefa simples (devido a implementação da interface `Task`), e também
adicionar subtarefas com o método `add_sub_task`. Nós vamos criar o
`MakeBatterTask` (TarefaFazerMassa):

```ruby
class MakeBatterTask < CompositeTask
  def initialize
    super('Make batter')
    add_sub_task(AddDryIngredientsTask.new)
    add_sub_task(AddLiquidsTask.new)
    add_sub_task(MixTask.new)
  end
end
```
Nós devemos manter em mente que os objetos árvores podem ir tão fundo quanto
quisermos. `MakeBatterTask` contém apenas objetos folha, mas nós poderíamos
criar uma classe que contém objetos compostos e ela iria se comportar exatamente
da mesma forma:

```ruby
class MakeCakeTask < CompositeTask
  def initialize
    super('Make cake')
    add_sub_task(MakeBatterTask.new)
    add_sub_task(FillPanTask.new)
    add_sub_task(BakeTask.new)
    add_sub_task(FrostTask.new)
    add_sub_task(LickSpoonTask.new)
  end
end
```
