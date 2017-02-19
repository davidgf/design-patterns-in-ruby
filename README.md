# Design Patterns in Ruby

Summary of the design patterns explained in the book [Design Patterns in Ruby](http://designpatternsinruby.com/), where [Russ Olsen](http://russolsen.com/) explains and adapts to Ruby 14 of the original 23 GoF design patterns.

## Design Patterns

### GoF Patterns

* [Adapter](adapter.md): helps two incompatible interfaces to work together
* [Builder](builder.md): create complex objects that are hard to configure
* [Command](command.md): performs some specific task without having any information about the receiver of the request
* [Composite](composite.md): builds a hierarchy of tree objects and interacts with all them the same way
* [Decorator](decorator.md): vary the responsibilities of an object adding some features
* [Factory](factory.md): create objects without having to specify the exact class of the object that will be created
* [Interpreter](interpreter.md): provides a specialized language to solve a well defined problem of know domain
* [Iterator](iterator.md): provides a way to access a collection of sub-objects without exposing the underlaying representation
* [Observer](observer.md): helps building a highly integrated system, maintainable and avoids coupling between classes
* [Proxy](proxy.md): allows us having more control over how and when we access to a certain object
* [Singleton](singleton.md): have a single instance of certain class across the application
* [Strategy](strategy.md): varies part of an algorithm at runtime
* [Template Method](template_method.md): redefines certain steps of an algorithm without changing the algorithm's structure

### Non-GoF Patterns: Patterns For Ruby

* [Convention Over Configuration](convention_over_configuration.md): build an extensible system and not carrying the configuration burden.
* [Domain-Specific Language](dsl.md): build a convenient syntax for solving problems of a specific domain.
* [Meta-Programming](meta_programming.md): gain more flexibility when defining new classes and create custom tailored objects on the fly.


## Contributing

Contributions are welcome! What could you do?:
* Find typos and grammar mistakes
* Propose a better way to explain a pattern
* Add clearer examples of a pattern usage
* Add other GoF patterns that are not covered in the book

**Code examples refactoring** PR's will **not be considered**. The examples provided by Russ Olsen in his book are meant to be simple and self explanatory, not the best performing or most elegant, their purpose is just educational.
