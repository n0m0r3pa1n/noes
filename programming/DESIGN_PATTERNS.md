# Design Patterns

## Creational patterns

They abstract the instantiation process of how objects are created, composed and represented. They do not add behavioral methods. They just operate on the data.

* Class creational pattern - It uses inheritance to vary the class that gets instantiated. 
* Object creational patterns - delegate instantiation to another object

### Idea

Creational patterns become important as systems evolve to depend more on object composition than class inheritance. As that happens, emphasis shifts away from hard-coding a fixed set of behaviors toward defining a smaller set of fundamental behaviors that can be composed into any number of more complex ones. Thus creating objects with particular behaviors requires more than simply instantiating a class.

* They all encapsulate knowledge of which concrete classes the system uses
* They hide how instances of these classes are created and put together

The system only knows the object's interfaces. The patterns give you a lot of flexibility in what gets created, who creates it, how it gets created, and when.

### Configurations

* Static
* Dynamic - runtime

### Index

#### Abstract Factory (Class)

* A system should be independent of how products are created
* A system should be configured with one family of products
* Enforce to use a family of related products
* Abstract factories defer the creation of objects to the concrete factories classes it has Basically you have an abstract class which uses concrete classes to create a type of object. 
* The difference between Abstract Factory and Facade is that Abstract Factories construct objects by a given rules while Facades hide the unpleasant implementation details to get a certain object.

#### Prototype (Object)

Create new objects by copying a specified prototype. Use it if:

* Classes to instantiate are known only at runtime
* To avoid building a class hierarchy of factories
* Remove and create prototypes at runtime

The whole idea is the clone objects and keep them in a HashMap from which you can access them again. For example in a DB call you can store the result in an object, clone it and pass it to different classes which need it. So you won't need to make several calls to DB. It is also true for cases where the new operator is expensive.

#### Builder (Object)

One construction of an object can create different representations.  The best usage would be if you have a common input which relies on building separate components underneath. Like if you are building a pizza and you have a Pizza builder. And you want to build a Hawai and an Italian pizza. You have those 2 pizza builders implementing the base pizza builder and returning different type of pizza. You pass the builder object to the waiter who will call the appropirate methods when needed.

Other usage is when you have many constructors with different parameters. So if you don't want to set all of the params you just make a static inner class with set*** methods which returns the current builder instance and a build() method which returns the already built object.

#### Singleton (Object)

To keep a single instance of a given class. Inside the class make a static instance variable of the class and a method that initializes it and only call this method.

#### Factory Method (Class)

The idea is to create an abstract class with an abstract method. So each subclass by overriding this method decides what class implementation to return, but the basic abstract class does the main work.

#### Object Pool (Object)

Create an object that manages a HashMap of objects, decides if they are locked or not and provides method to reset locked objects or take objects from the pool and provide them to other objects. 

The patterns is used when object creation is expensive and you do want to have limited number of object instances.

## Structural patterns

Structural patterns are concerned with how classes and objects are composed to form larger structures. 

### Idea

* Structural class patterns - use inheritance to compose interfaces or implementations.

* Structural object patterns - describe ways to compose objects to realize new functionality. Changing composition in runtime is the key benefit.

### Index

#### Adapter/Wrapper (Class)

Adapt something old but working to something new. It works by wrapping the old class and working with something that uses this class.

#### Bridge (Object)

The main idea is to decouple the binding between an implementtion and an abstraction. The idea is that you want to work with interfaces all the time. So if you have an abstract Window class that has to work with a specific Window Implementation for Windows or Linux, your window class should work with an interface of the implementation, not a class which extends Window. The idea is to favor composition over inheritance.

Subclassing to provide different implementations of a given class is not a good idea.

* Adapter makes things work after they're designed; Bridge makes them work before they are.

#### Composite (Object)

Compose objects into tree structures and treat invidividual objects and compositions of objects uniformly.

To use this pattern you have two types of classes: Composite and Leaf. Each composite may have a List of Leafs or other composites within it. 

How to use it? Create a base abstract class called Component. Make the Composite and the Leaf extend from it. The composite will override the add() method so it will support adding components while the Leaf won't. This way you can have a list of Component whether it is a Composite or a Leaf.

#### Decorator/Wrapper (Object)

The decorator allows you to change functionallity of an object at runtime. One way to add functionality is by subclassing. Another way is by using decorators.

The idea is that you create an interface which other classes implement. After that each decorator accepts a type of the interface in it's constructor, calls it's method and adds additional data to it. 

#### Facade (Object)

The idea is to provide a unified interface to get some data so you don't have to know all of the implementation details below it. You simply create a class that instantiates many other classes inside it and provides some useful public methods to get data from these classes if needed.

#### Flyweight (Object)

The idea is to keep a hashmap of objects which are similar in a given property and reuse them, not create them every time. This will save at least 200% of creation time. Like having a HashMap of Rectangles with colors, but setting their size by setting a property.

#### Private Class Data (Class)

The idea is to have a private data class within the class you want to hold your data. This way you can make all of the data to be final and not changeable.

#### Proxy (Object)

The idea of the pattern is to limit the visibility of certain methods in a class. It is a proxy to another object. You keep a reference to the object in your proxy and only call certain methods in it. It is simple to implement.

#### Unit of work (Class)

Create a class that holds reference to a single context so all underlying repositories are working with this context.

## Behavioral patterns

### Idea
 
Behavioral patterns are concerned with algorithms and the assignment of responsibilities between objects. Behavioral patterns describe not just patterns of objects or classes but also the patterns of communication between them.

* Behavioral class patterns use inheritance to distribute behavior between classes.
* Behavioral object patterns use object composition rather than inheritance.

### Index

#### Chain of Responsibility (Object)

Chain a sequence of objects to receive the data for a single request.

Case 1: For the implementation each object contains a reference to the next object whish should be executed.  All objects conform to a common abstract class which has a method called handle. Then the first object of the sequence is called with handle and if it is not interested in the request, it calls the next object handle.   

Case 2: You can keep a simple array of objects which conform to a single interface and just call the method.

#### Command (Object)

Encapsulate a request as an object, have queues, parametrize for different clients, log requests and support undoable operations.

Case 1: The idea is to have a single interface called Command which has a method called execute. You have different implementations of Command and the client passes the concrete implementation to the object that works with Commands. Then the object calls the execute method of the command.

Case 2:  Each Command object has method, args and receiver. Whenever you create a command you use reflection to call the metod with the arguments and pass it to the receiver. It's way too complicated.

#### Interpeter (Class)

Given a language, define a represention for its grammar along with an interpreter that uses the representation to interpret sentences in the language.

The idea is to create a custom parser of a simple language which is connected in some way with the object hierarchy or with the languages themselves.

#### Iterator (Object)

Provide a way to access the elements of an aggregate object sequentially without exposing its underlying representation. Separating the traversal part saves code which has to be put in the collection object.

*External iterators*: The idea is to have a common interface called Iterator which has next(), remove() and hasNext() methods. Every class which has data in it can share an iterator so you can use it to traverse the whole structure.

*Internal iterators*: The idea is to not handle the iteration but just write code which is meant  to do the thing with each elements. Stream are the type of internal iterators in Java.

#### Mediator (Object)

The mediator handles the communication between related objects. All communication is handled by the mediator and the objects don't know anything about each other. It allows loosle coupling between the objects.

So you make different classes which accept the mediator interface in their constructor. After accepting it they call different methods on the mediator object to communicate with each other. In the Mediator you keep a kind of reference to each object. It may be a string id or something so this way you allow objects to target other objects within the mediator.

#### Memento (Object)

Without violating encapsulation, capture and externalize an object's internal state so that the object can be restored to this state later.

There are 3 classes in this pattern.
*Memento* which is the class that holds the latest state of the object. *Originator* is the class which creates the memento and passes it's internal state to it. Also restores from Mementos. *Caretaker* is the class that is responsible for the memento's safekeeping. When you want to restore an object's state you call the Caretaker to get a memento and then pass it to the Originator. 

#### Null object (Object)

The idea is simple. Create a NullSomething.class which does nothing. You may use it in cases where nothing should be done. In most cases this class is just a singleton, no need to have multiple instances of it.

#### Observer (Object)

Use it when you need many other objects to receive an update when an object changes. The subjects doesn't know anything to the observers.

You have a common interface for the observers and one for the subject. Basically the subject notifies all observers when a change occurs. It keeps a list of observers for the type of change.

#### State (Object)

Allow an object to alter its behavior when its internal state changes. The object will appear to change its class.

You have classes representing the different states for an object. Each state class has a reference to the object and can dynamically set the state of the object to another one. These classes can call other methods on the objet or they can execute things themselves. The main idea is that you set the state once and then each state knows what different state to assign to the machine. That is how states work.

#### Strategy (Object)

When you want to define a class that will have one behavior that is similar to other bejaviors in a list. You can easily switch between behaviors.

The idea is to have a single interface with multiple implementations. Then in the client class you can switch between these implementations to do different kinds of things. You can set each one of them as the current strategy that has to be executed.

#### Template method (Class)

This pattern is used to create a group of subclasses that have to execute a similar group of methods. Template Method lets subclasses redefine certain steps of an algorithm without changing the algorithm's structure.

You have an abstract class with a "template" method - that is a method that does call many other methods defined in the abstract class. All classes which extend from this abtract class have the option to override some of it's methods so they will do different things when called by the template method.

#### Visitor (Object)

Represent an operation to be performed on the elements of an object structure. Visitor lets you define a new operation without changing the classes of the elements on which it operates.

This is best explained with example. You have different types of classes. Neccessity, Tobacco, Liquor. You want to tax them differently. You make all of the classes implement a common interface which has a method *tax(Visitor visitor)* and accept  a visitor object. This visitor object will have different *calculateTax()* methods for the different types of objects it has to calculate the tax for. It looks like a dependency injection of an object into other objects. 









