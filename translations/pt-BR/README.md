# Padrões de Design em Ruby

Sumário dos padrões de design explicados no livro [Design Patterns in Ruby](http://designpatternsinruby.com/), onde [Russ Olsen](http://russolsen.com/)
explica e adapta para Ruby 14 dos 23 padrões de design da Gangue dos Quatro
(GoF - Gang of Four).

## Padrões de Design

### Padrões dos GoF

* [Adapter](adapter.md): ajuda duas interfaces incompatíveis a trabalharem em conjunto
* [Builder](builder.md): cria objetos complexos que são difíceis de serem configurados
* [Command](command.md): executa uma tarefa específica sem ter informação alguma sobre o receptor da requisição
* [Composite](composite.md): constrói  uma hierarquia de objetos árvore e interage com todos eles da mesma forma
* [Decorator](decorator.md): modifica as responsabilidades de um objeto adicionando algumas funcionalidades
* [Factory](factory.md): cria objetos sem ter que especificar a classe exata do objeto que será criado
* [Interpreter](interpreter.md): fornece uma linguagem especializada para resolver um problema bem definido em um domínio conhecido
* [Iterator](iterator.md): fornece um jeito de acessar uma coleção de sub-objetos sem expor a representação por baixo
* [Observer](observer.md): ajuda na construção de uma sistema altamente integrado, sustentável e evita acoplamento entre classes
* [Proxy](proxy.md): permite que tenhamos mais controle sobre como e quando nós acessamos um determinado objeto
* [Singleton](singleton.md): tenha uma única instância de uma certa classe em toda sua aplicação
* [Strategy](strategy.md): modifica parte de um algoritmo em tempo de execução
* [Template Method](template_method.md): redefine certos passos de um algoritmo sem mudar a estrutura do algoritmo

### Padrões que não são dos GoF: Padrões para Ruby

* [Convention Over Configuration](convention_over_configuration.md): constrói um sistema extensível sem ter que carregar o peso da configuração
* [Domain-Specific Language](dsl.md): constrói uma sintaxe conveniente para resolver problemas de um domínio específico
* [Meta-Programming](meta_programming.md): ganhe mais flexibilidade quando estiver definindo novas classes e criando objetos customizado em tempo de execução

## Contribuições

Contribuições são bem-vindas! O que você poderia fazer?:

* Encontrar erros de digitação e gramática
* Propor um jeito melhor de explicar algum dos padrões
* Adicionar um exemplo melhor de utilização de um padrões
* Adicionar algum outro padrão da GoF que não foi coberto no livro

Pull Requests de **refatoração de exemplos de código não serão consideradas**.
Os exemplos fornecidos por Russ Olsen no livro foram feito para serem simples
e auto explicativo, não o mais elegante ou com melhor desempenho, o propósito
deles é apenas educacional.
