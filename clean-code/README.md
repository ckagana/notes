## Naming things

### Names should be fully consistent with their meaning
It's important that names don't give false clues about their meaning. For example, `hp` can be an abbreviation of "hypotenuse", but it's also the name of a technology, and so is disinformative, because misleads the reader trying to get its meaning.

Another example is using programming-related concepts in names: `List` represents a very specific concept in programmer's knowledge, so `accountList` should be named like that only if it's actually a `List`: if it's just a collection of accounts, it should be named something like `accountGroup` or, even better, just plain `accounts`.

### Similar concepts should be spelled similarly
Different concepts spelled similarly leads to disinformation: never use names like `XYZControllerForEfficientHandlingOfStrings` and `XYZControllerForEfficientStorageOfStrings` together, because they look like they're almost identical, but their meanings are the opposite: the first is about reading operations, the second is about writing operations. A famous example are characters that look like numbers: `l` looks like `1` and `O` looks like `0`: names should not be ambiguous for this.

### Avoid noise words
*Noise words* are words used as part of names, that take space in the name without actually adding any meaning. `ProductInfo` and `ProductData` can't be distinguished because `Info` and `Data` do not add any meaningful information to `Product`: they are noise words. Also, there's no difference between `zork` and `theZork`: if two methods are named like this, and they do quite different things, this would be a big problem.

#### Avoid redundant words
Redundant words are also noise words: `NameString` is redundant because it's obvious that names are strings, and besides this, one could think that you also have a `NameInt` somewhere; if you have `Customer` and `CustomerObject` the reader won't understand the difference, and won't know which one should he use to get the customer's payment history.

### Names should be pronounceable
Names should be pronounceable, to let people make intelligent conversations about them easily. So, don't use names like `genymdhms`, `class DtaRcrd102` and such.

### Names should be searchable
Names being searchable is an invaluable aid to debugging. A variable named `e` isn't searchable, because searching `e` in a codebase returns absolutely every file, and almost every line present in the codebase. Un-searcheable names should be used only when their scope is very limited: a single function, or a single block; this way, you won't need to make any search.

#### Avoid magic numbers and strings
This is also another reason not to use magic numbers and strings for constants: if you use `7` instead of `MAX_CLASSES_PER_STUDENT`, you can't easily look for `7` in you codebase; if you use `154251` instead of `DEFAULT_VENDOR_PREFIX`, someone may have mispelled it as `152451`, introducing a bug which can't even be found with a search for the original, correct value.

### Avoid encodings inside names

#### Avoid type prefixes (hungarian notation)
Adding type prefixes, for example, is a bad practice because it clutters the name with information that is useless most of the times, since IDEs and compilers can check and enforce type correctness, and it makes much harder to make changes to the code, because if you have `strPhone` which is a string, and later you need to change it into an integer, you should also replace all occurrences of `strPhone` to `intPhone`.

#### Avoid visibility prefixes (hungarian notation)
Since classes should be small, and you should use text highlightning, it's not necessary to use encodings like `m_name` to tag member variables for them to be distinguished from local variables.

#### Avoid marking interfaces
You shouldn't use encodings to mark interfaces, like `CustomerInterface` or `ICustomer`, because an interface should be all about the meaning that it's conveying, and nothing about technical details, not even the fact that it is an interface rather than an abstract class. If you need to distinguish an implementation from an interface, encode the difference into the name of the implementation, like `ShapeFactoryImp`.

### Use nouns for classes, verbs for methods
Classes and objects should be named with noun or noun phrase names like `Customer`, `Account` and `AddressParser`. Too generic purpouse names like `Manager`, `Processor`, `Data`, or `Info` should be avoided. Method names should be made out of verb or verb phrase names, like `postPayment`, `deletePate` or `save`.

### Use keyword names for functions
When choosing a name for a function, try to add to it keywords describing what its arguments are. For instance, `writeField(name)` is better than `write(name)`, because it explains that `name` is actually a `Field`. `assertExpectedEqualsActual(expected, actual)` is better than `assertEquals(expected, actual)`, because it tells us what the order of arguments is without us needing to look it up.

#### Use static factory methods
When you have different overloaded constructors, use static factory methods that describe arguments instead: `Complex fulcrumPoint = Complex.FromRealNumber(23.0)` instead of `Complex fulcrumPoint = new Complex(23.0)`, and consider using private constructors to enforce their use.

### Be consistent when referring to the same concepts
You should be consistent with the names you use to refer to the same concepts: for instance, you should use one of `get`, `fetch` or `retrieve` for all classes, otherwise users of your API would be confused about what's the correct name to use in each class. Or, pick one of `controller`, `manager` and `driver`, since there's no sensible difference between them, as far as classes or methods names are concerned.

### Never use the same term with two different meanings
For example, if several classes already use `add` to indicate creating a new value by adding together two existing values, and we want to add a new method that puts a value into a collection, we may be tempted to use `add` again; however, in this case `add` would be used with two very different meanings, and this could mislead the reader: use `insert` or `append` instead.

### Take names from standard programming concepts, or the problem domain
When our implementation uses very known programming concepts, like pattern names, algorithm names, math terms, etc., we should use these terms inside our names. When there's no obvious programming idea behind what we're doing, we should use terms taken from the problem domain.

### Use names inside small contexts
Most names, taken alone, aren't very meaningful: for this reason, they need to be put inside a context. For instance, `state` could refer to an address, or the state of a system. The best way to add context to names, is enclosing them in packages, classes and functions. If you have a long function with several local variables, it's hard to give each variable a meaningful context, and you usually resort to add prefixes, like `addrFirstName`, `addrLastName`, etc. 

What you should do in this case is refactoring the function, for instance creating an `Address` class keeping all information related to addresses; inside such a class, you don't need to add prefixes any more, because `firstName` inside an `Address` class is obviously the name of a person, as part of her address.

Thus, you should always use contexts of limited size, with a limited number of variables, so that the context of each name is immediately clear. Framing names inside functions, classes and namespaces, also allows us to use small names, avoiding cluttering names with unnecessary context information.

## Functions
Functions should be small, hardly 20 lines long, and their meaning should be obvious.

Blocks with `if`, `else`, `while` and so on, should be one line long, where that line is probably a function call. In this way the reader is not forced to maintain the branching or conditional logic in his brain while at the same time thinking at the rest of the enclosing function.

Functions shouldn't contain nested structures, thus the indenting level of functions shouldn't be grater that one or two.

Functions should do one thing, should do it well, and should do it only. To correctly define what this "one thing" should be, first we break the function logic down to a series of steps, describing what the function is accomplishing, for instance:

>TO RenderPageWithSetupsAndTeardowns, we check to see whether the page is a test page and if so, we include the setups and teardowns. In either case we render the page in HTML.

Then we check and see if the steps we described are one level of abstraction below the stated name of the function. This is because functions are used to decompose larger concepts (the name of the function) into steps that go into more detail about how that concept is build (thus, going into a lower level of abstraction).

A function isn't doing only one thing, if the steps it comprises are at many different levels of abstraction, and thus aren't just the detailed description of its title. To see this, we can try and extract other functions from the code: if that's easy to do, it means that the code is doing multiple different things inside the same function, so much so that it was easy for us to spot a different thing and extract it into its own function; if extracting code into new functions only lead to trivial functions whose names simply restate their code, it means that our original function was focused enough and didn't need any further shrinking.

Again, functions that are doing only one thing can't be reasonably divided into sections, like *declarations*, *initializations*, etc.

Statements inside a function should all be at the same level of abstraction. For instance, a method call like `getHtml()` is at a relatively high level of abstraction, compared to something like `PathParser.render(pagePath)`, and even more with `append("\n")`, which is less abstract than both the previous ones. This is important so that we don't mix interface code (meaning code of higher abstraction) with implementation details code (meaning code of lower abstraction).

Organize functions so that each on is followed by those at the next level of abstraction. In this way, the program starts with functions at the topmost level of abstraction; then each function definition contains the implementation details of the interface represented by the function, but these details are likely other function calls; these new functions are thus at the next level of abstraction with respect to the first ones, and are written immediately after then, and so on:

>To includeSetupsAndTeardowns, we include setups, then we include the test page content, and then we include the teardowns.

>To includeSetups, we include the suite setup if this is a suite, then we include the regular setup.

>To includeSuiteSetup, we search the parent hierarchy for the "SuiteSetUp" page and add an include statement with the path of that page.

>To searchParent...


### Limit `switch` statements with polymorphism
`switch` statements are usually source of violations of several design principle:
- being lists of several different cases, they are large, and they make functions large;
- being lists of how to do things in several different cases, they usually do more than one thing;
- they violate the Single Responsibility Principle, because there's more than one reason for them to change;
- they violate the Open Closed principle, because they can't be closed for modification, since as soon as there's a new case, they have to be modified;
- if we are splitting code amongst multiple cases here, it's very likely that we'll have to split other code amongst the same cases in several other places.
The solution is creating a different class or family of classes for each different case: each of them will have its own set of objects that have the responsibility of knowing how to deal with that case; then, a single Abstract Factory will contain the single `switch` statement choosing which family of classes to instantiate, depending on the current case.

### Give functions long descriptive names
If you manage to create short functions that do only one very specific thing, it will be easier to choose a descriptive name for it. Often, these names will be quite long, especially if we are naming a function deep into the abstraction hierarchy, but this is not a problem.

### Use as few arguments as possible, prefer instance variables
The ideal number of arguments for a function is zero, and the next acceptable case is one. The greater the number of argument, the strongest justification you'll need to provide for them. You should never use more than three arguments anyway. If possible, us instance variables instead, and avoid using arguments to keep output values.

This is because arguments are really an implementation detail of the function interface (signature), but the client needs to know about them, because he has to provide values for them when calling the function. Thus, adding arguments forces the client to keep in mind details about the function implementation.

Furthermore, functions with many arguments are difficult to test, because you should test them with all combinations of appropriate values.

### With single arguments, make a question or filter input
When creating a function with a single argument, let it either be a question about the argument, like `boolean fileExists("MyFile")`, or a filter transforming that argument into something else, like `InputStream openFile("MyFile")`.

### Never use flag arguments
Flag arguments are to be avoided because their purpose is making the function do two different things according to their boolean value, and functions should only do one thing. In addition to this, they are confusing because a call like `render(true)` doesn't have a clear meaning. Better to split the function into two versions: `renderForSuite()` and `renderForSingleTest()`.

### Use dyadic functions only when the meaning of arguments is natural
There are little cases in which writing a function that takes two arguments is acceptable. In a function like `Point(0, 0)` we are using two arguments, but they really are just two components of a single value, which is the set of coordinates of the point. A function like `assertEquals(expected, actual)`, while still a natural case (it's obvious that to compare for equality we need to provide two values), it's confusing though, because the ordering of arguments isn't obvious. When possible, it's better to convert dyadic functions into monadic ones, replacing certain arguments with member variables, or extracting new classes that take one argument in their constructors.

### Refactor multiple arguments into their own classes
When you feel the need to pass multiple arguments to a function, it's very likely that most, or all, of them are really part of the same concept. If this is the case, you should create a new class representing this concept, let the former arguments be properties of that class, and pass an object of that class to the function instead of the single arguments. For instance we can easily convert `Circle makeCircle(double x, double y, double radius)` into `Circle makeCircle(Point center, double radius)`.

### Avoid side effects
If a function's name doesn't explicitly state that the function is going to change the state of the system in some way (writing memory, doing I/O, etc.), but then it does (even if that's an implementation detail), that's a side effect, and means that the function is doing something different than expected, and certainly more than one thing.

Side effects add temporal coupling, meaning that calling functions with side effects at different times would result in different effects, possibly introducing subtle bugs.

### Avoid output arguments
The need for using output arguments can be mitigated using the invocation object. For instance, a function like `appendFooter(s)` is confusing because it's not clear if it appends something to a footer, or appends a footer to something. In the latter case, `s` would be an output argument, and it would be much clearer if it was `report.appendFooter()`, where the argument has been converted into an invocation object.

In general, if functions must change the state of something, they should change the state of their owning objects.

### Use command query separation
Functions should either perform an action changing the state of their object, or answer a question about their object, but never both. For instance, consider the function `public boolean set(String attribute, String value)`, that sets the value of a named attribute, and returns if that was successful or not. This is often used in confusing statements like `if (set("username", "name"))...`, where it's not clear if `set` is just a check if the username exists (`set` used as an adjective), or it's also setting the username (`set` used as a verb). The ambiguity is avoided separating the command from the query:

```java
setAttribute("username", "name");
if (attributeExists("username")) {
    ...
}
```

### Throw exceptions instead of returning error codes
Returning error codes from functions performing actions is a violation of the command query separation principle, because a function like this is both performing an action, and telling if that action is successful or not, and thus promotes writing statements such as `if (deletePage(page) == E_OK)` that, while not being that confusing since it's clear that they are actions (but still it would be better to place them outside an `if`, that should really contain a query, not a command), still leads to deeply nested structures.

Returning an error code forces the client to deal with the error immediately:
```java
if (deletePage(page) == E_OK) {
    if (registry.deleteReference(page.name) == E_OK) {
        if (configKeys.deleteKey(page.name.makeKey()) == E_OK) {
            logger.log("page deleted");
        } else {
            logger.log("configKey not deleted");
        }
    } else {
        logger.log("deleteReference from registry failed");
    }
} else {
    logger.log("delete failed");
    return E_ERROR;
}
```

Using exceptions, instead, the error management can be separated from the happy path:

```java
try {
    deletePage(page);
    registry.deleteReference(page.name);
    configKeys.deleteKey(page.name.makeKey());
} catch (Exception e) {
    logger.log(e.getMessage());
}
```

### Extract the body of try/catch blocks in their own functions
Try/catch blocks add a new level to the nested structure of the code, making it less readable: it's better to extract their bodies into functions:

```java
public void delete(Page page) {
    try {
        deletePageAndAllReferences(page);
    } catch (Exception e) {
        logError(e);
    }
}
```

### Functions with try/catch blocks should have nothing else
Handling errors is "a thing" on its own, and as such a function using a try/catch shouldn't do anything else, otherwise it would be doing more than one thing. The rule we can extract here is that if a function has a try/catch block, the function should start with `try` and have nothing after the `catch` or `finally` block.

## Formatting
Code inside a file should be ordered so that at the beginning of the file the more generic and abstract code is placed, and then, the more we scroll down the file, the more detailed and specific the code becomes.

Concepts that are closely related should also be closely located in the same file. This, for instance, is one of the reasons why protected variables should be avoided: a protected variable is a concept closely related to code that is located in another file.

Since functions should be small, local variable declarations should appear at the top of the function. Variables declared at the top of a block should be avoided. The only exception are control variables for loops, which should be declared within the loop statement.

Instance variables should all be declared in the same place, be it at the top of bottom of the class.

If a function is called by another, it should be placed after it, as much as possible. This allows the reader to be confident that the definitions of the used functions will follow shortly.

## Objects and data structures
The purpose of an object is to provide an abstract interface representing the meaning of the data that it's used to model. This interface should expose abstract properties of the data, and an access policy. It's very rare that the underlying implementation chosen for building this model is at the same time a good interface for understanding its meaning. Thus, avoid to expose implementation details, either with public properties, or with single getters and setters. Looking at the object interface, it should be impossible to guess the underling implementation, because in fact it would be meaningless to do so.

While objects hide their data behind abstractions, exposing functions to work on it, data structures expose all their implementation details, without providing any meaningful action available to be performed on them, because they are provided externally. Thus, objects are flexible when it comes to change the way data is manipulated (implementation), but rigid when it comes to change the set of operations being provided (interface); on the other hand, data structures are flexible when it comes to add or remove functions (interface), because they are external and the data structure is not affected, but rigid when it comes to change the way data is represented (implementation).

As an example, consider the following procedural example, leveraging data structures:
```java
public class Square {
    public Point topLeft;
    public double side;
}

public class Circle {
    public Point center;
    public double radius;
}

public class Geometry {
    public final double PI = 3.14159;

    public double area(Object shape) throws NoSuchShapeException
    {
        if (shape instanceof Square) {
            Square s = (Square)shape;
            return s.side * s.side;
        }
        else if (shape instanceof Rectangle) {
            Rectangle r = (Rectangle)shape;
            return r.height * r.width;
        }
        throw new NoSuchShapeException();
    }
}
```

In this code, using data structures instead of objects allows us to easily add new methods to the set of functions we use to manipulate the data: for instance, if we wanted to add a way to calculate the perimeter (interface change), we could just add a `perimeter()` function to `Geometry` without needing to touch all the existing data structures. On the other hand, if I added a new kind of shape (implementation change), I would need to update all existing functions of `Geometry`, teaching them how to deal with the new shape.

Let's consider the object-oriented version of this example:
```java
public class Square implements Shape {
    private Point topLeft;
    private double side;

    public double area() {
        return side*side;
    }
}

public class Circle implements Shape {
    private Point center;
    private double radius;
    public final double PI = 3.14159;

    public double area() {
        return PI * radius * radius;
    }
}
```

Here we can see how easy it is do add a new kind of shape (implementation change) without the client code being affected at all (the client just accepts `Shape` arguments, and calls `area()` on them, without bothering of what actually is the current shape). On the other hand, if we wanted to add the `perimeter()` method (interface change), we would need to change `Shape`, and this would mean changing also all client code relying on it.

In the end, then, we should understand when we need to adapt for change to the data, and use object-orientation in that case, and when we need to adapt for change to the interface, and use procedural programming in that case.

The Law of Demeter states that: if a class `C` has a method `f`, the code of `f` should only call the methods of: `C` itself, or a local object created by `f`, or an object passed as argument to `f`, or, finally, an object located in an instance variable of `C`. More sintetically, `f` should only call methods on objects that it knows directly, and shouldn't call methods on objects obtained indirectly (for instance as return values of other function calls).

Consider for instance this code:
```java
Options opts = ctxt.getOptions();
File scratchDir = opts.getScratchDir();
final String outputDir = scratchDir.getAbsolutePath();
```

This code could be easily rewritten as `final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();`. Writing it as such it looks like a violation of the Law of Demeter, since the current method has to know a lot about the inner implementations of the objects it's using. This is certainly true if those are indeed objects, and it's also a problem of those objects, because they're exposing their implementation.

On the other hand, if those were not objects, but data structures, then this wouldn't constitute a problem, since data structure are exactly used to expose their implementation. In this case, however, it would have been better if we have had `final String outputDir = ctxt.options.scratchDir.absolutePath`.

If those were indeed objects, how could we write this kind of logic without going against the Law of Demeter? We should be abiding by the Tell, Don't Ask principle: we shouldn't ask any object for its implementation details in order to do something with them; we should rather tell it what to do, in order for it to decide how to best perform the task according to its implementation details. So, first we need to understand why we needed to get that `outputDir` value, for instance to create a file in that directory, and then refactor the code so that we ask our object to do all the work directly:
```java
BufferedOutputStream bos = ctxt.createScratchFileStream(classFileName);
```

It's often useful to rely on Data Transfer Objects: classes with only public variables and no methods (thus, data structures essentially). They come in handy for instance when translating a data format into another, in multiple stages.

## Error handling
Using return codes to handle error situation ends up cluttering the caller, because it would need to check if an error occurred immediately after the call, and react to it if any happened. Using exceptions, the error situations can be handled in the place where it makes most sense, because we can move that code into an appropriate place where only error handling, and no business logic, is present.

To add error handling in a consistent manner, it's useful to start with a test that asserts that an exception will be thrown: using TDD; the next step would be creating a stub method to use in that test. Since what we are testing is the exception, the first thing we need to add is a `try/catch/finally` block that executes some code throwing an exception, and converts that exception into the one we are expecting in the `try` block. Working like this, we use tests to drive us to always setup a proper error handling mechanism.

A `try` block defines a transaction, meaning that, whatever happens inside the `try`, we should always end (`finally`) in a consistent state.

If your function returns `null` to signal an exceptional situation, it's forcing your client to immediately react to it, but the caller might very likely not be the best place where this case can be properly handled. This often leads to deeply nested conditional structures, or internal functions calls, that clutter the codebase with code that has nothing to do with the current logic. Furthermore, if the client misses one null check, this puts the application at risk of crashing in incomprehensible ways, or having misterious behavior.

In places where you would return `null`, throw an exception, or return a special case object instead. If you're relying on a third-party method that may return `null`, wrap it with a method throwing an exception or returning a special case. If the method is normally returning collections, in the case when it would return `null`, make it return an empty collection instead.

Never pass `null` to methods, because they force the callee to add all kinds of conditional structures to deal with the possibility that arguments are null. Let "never pass `null`" be the default rule, and if you find a `null` being passed in the codebase, that would signal a coding error.

## Boundaries
When using interfaces at the boundary of our program, which are interfaces of the third-party APIs, we are not protected against updates of the APIs: if the third-party releases a new version with updated interfaces, we have to find and change all occurrences of it in our code, or give up updating in the first place. In addition to this, third parties usually strive to create APIs that are all-around useful: this means that a third party interface that we may be using can usually do much more than what we need that object to do; this poses us at risk of letting clients of our code be able to do unexpected and dangerous things with that object.

For instance, the third party `Map` interface provides also methods to delete the elements: if we use it for collections for which the client isn't really meant to delete elements, we may have a problem if the client doesn't use our code in the way we are expecting.

The solution here is to always use boundary interfaces inside the private code of methods' implementations, and never as return values or arguments: the usage of the boundary should be hidden from the client; this obviously does not apply to communications among private methods or very closely related classes.

Even if third party interfaces weren't changing, and contained no more functionality that we need, they'd still be different from what our program required. For instance, we would need an interface extending another one we are using; or we would need interfaces segregated in a certain way, that the API is not respecting. This is why it's a good practice to write adapters to convert the third party interface to another interface that works best for us.

## Test-Driven Development
Unit tests need to be very readable; in particular, they should implement the "build-operate-check" pattern, which means that they should have a section where needed objects are built, a section where operations are performed on these objects, and finally a section where results of those operations are checked. To make this structure clear, unit tests should contain only code that is at the same abstraction level of these sections: details about how things are created, how operations are performed, etc., should be hidden in other functions.

It's better in unit tests to use a domain specific language tailored to the needs of tests, rather than just using the same API used by production code, directly. This testing API can't typically be designed upfront, rather it evolves along with tests.

Use only one assert per test method, and apply the convention of using `given`, `when`, `then` propositions to identify the standard steps of every test. For instance:
```java
public void testGetPageHierarchyAsXml() throws Exception {
    givenPages("PageOne", "PageOne.ChildOne", "PageTwO");

    whenRequestIsIssued("root", "type:pages");

    thenResponseShouldBeXML();
}

public void testGetPageHierarchyHasRightTags() throws Exception {
    givenPages("PageOne", "PageOne.ChildOne", "PageTwo");

    whenRequestIsIssued("root", "type:pages");

    thenResponseShouldContain(
        "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>");
}
```

Using only one assert per method may result in a lot of code duplication, but usually this is worth it since in this way we get tests that are way more clear and easy to understand.

Tests should be fast, otherwise you won't run them as often as necessary.

Tests should be independent from one another, otherwise one test failing would cause other tests to fail as well, hiding other potential errors.

Tests should be repeatable in every environment, otherwise you'll always have an excuse for why they failed.

Tests should be self-validating, meaning that they should return either `true` or `false`, and not reading through some other kind of output and figure out what happened. Not obeying to this rule makes failures subjective, and slows down development.

Tests should be written in a timely manner, meaning just before the production code that makes them pass. Writing tests after the production code has already been written may make it hard to test it, and some code may end up being not testable.

## Classes
Classes should begin with public constants, followed by private static variables and private instance variables. It's rarely needed to have public properties. Private utilities should follow public functions using them.

Properties and methods may be made protected to help writing tests. If possible, package level visibility may be used as well. Anyway, maintaining privacy is always the topmost necessity.

Classes should have few responsibilities, regardless of the number of methods. Since the class name should very precisely state the responsibilities of the class, not being able to come up with a descriptive name, or relying on a generic or ambiguous one, is a symptom that the class we are creating may have too many responsibilities. Presence of the words "if", "and", "or", or "but" in the class description is also a symtom of too many responsibilities.

Classes should be choese. A good way to measure choesion is counting the number of private instance variables. If a variable is used only by a small subset of methods, the class is not much cohese, and this is bad because it probably means that the class has more than one responsibility. If there's a subset of variables only used by a subset of methods, those should all be moved in another class, because they probably represent a different concept. This also makes it easier to respect the open-closed principle, because when a class has multiple responsibilities, if we have to add new functionality or change an existing one, we are forced to modify the class; but if we extract each responsibility in its own class, we can add or change behaviour just by extending them, instead of modifying the original class.

Maximum cohesion is achieved when all methods use all instance variables; whilst this is very difficult to achieve, it still indicates that methods should all use a great part of the class variables. It's obvious that big classes with a lot of variables and methods will very hardly be choese enough.

As per the Dependency Inversion Principles, classes should depend on abstractions, and not on concrete implementations. This favor reuse, allows to change or add functionality by extension instead of modification (open-closed principle), and helps testability.

## Systems
### Separate the construction concern
Often applications create objects on the fly, when they need them. This couples system startup (building of all necessary objects), with business logic, while the two concerns should happen distinctly. What we should do is have a single entry point to the application, for instance a `main` module, that contains everything will be executed at the startup of the application, and have all objects constructed and wired up there. Only after all necessary objects have been constructed and connected, the business logic can happen. The application, thus, has no knowledge of the building process, and simply expects that all objects it needs are just handed to it.

### Use an inversion of control mechanism
The most common pattern to achieve this separation is *Inversion of Control*. Objects needing dependencies don't have the responsibility of instantiating those dependencies themselves: instead, that responsibility is assigned to a single authoritative object, that is the only component of the program knowing how to create objects, i.e. which instances are to be created. Inversion of control can be implemented with *Service Locator* or *Dependency Injection*. In the first case, dependent objects still have the responsibility to do the wiring: they ask an external service for the instance of the dependency they need, and the service creates it, and pass it to them; with dependency injection, instead, there's a container object injecting dependency into dependant objects, which thus have no knowledge of the wiring process either.

### Use factories for objects that needs to be constructed late
Sometimes objects can't really be constructed at the beginning of the execution, because, for example, there's not enough information to do that. In these cases, the startup should build proper factories, that are passed to the main logic, which then uses them to create the objects it needs at the proper time. The details of how the construction is done are still hidden from the business logic, though.

## Emergent design
The system's correctness must be verifiable. Therefor, the system must be comprehensively testable, and all tests must pass all of the time. To make a system more testable, we are forced to make classes smaller and more focused on a single responsibility.

Having tests in place, we are free to refactor the codebase to make it cleaner, without fear of regression. Refactor is the step where we strive to employ rules of good design.

During refactoring, we strive to eliminate all duplication. Duplication comes in many forms, not only identical code, or code that looks alike, but also code that is different, but serves the same purpose, or is redundant, like having both a method `int size()` and another `boolean isEmpty()`.

Avoid exaggerating while minimizing the size of classes and methods: you shouldn't have too many of them either.

## Concurrency
Concurrency is a decoupling strategy, since it helps to separate what gets done from when it gets done. Applying concurrency means transforming the system from one big loop to a set of small computers collaborating.

Code dealing with concurrency is complex enough to be by itself another reason to change. This means that, following the Single Responsibility Principle, concurrent code should be moved to its own classes. Concurrent code should thus be always separated from other code.

You should minimize the number of places where you use shared data among multiple threads. Shared data should be strictly encapsulted, and only a very few selected clients should be able to access it.

It's often possible to just pass read-only copies of data to other threads, instead of actually sharing it. Or, you can pass copies that will get modified, and then gather all modified versions in a single thread and merge them.

When designing the multithreaded application, strive to make threads as independent as possible, partitioning data in indepenent subsets, such that there is the smaller need for sharing as possible.

Always try to use data structures that are optimized for concurrency.

In the producer-consumer execution model, ore or more producer threads set up work to do in a queue, and then one or more consumer thread pick up the work from the queue and complete it. The queue is a bound resources, so producers must wait for free space to appear in the queue, and consumers must wait for some work to be present in the queue. Communication is needed, because producers put work in the queue and signal that the queue is no longer empty, while consumer take work from the queue and signal that the queue is no longer full.

In the readers-writers execution model, writer threads modify a shared resource that needs to be read by reader threads. Readers can't read a resource being currently written; readers must be served as fast as possible, and this can cause starvation, preventing writers to ever be able to write a particularly requested resource; on the other hand, allowing writers to write can force readers to wait, decreasing throughput.

In the dining philosophers execution model, multiple threads need a certain amount of shared resources to work, and threads must wait that resources are available before starting working. In this situation several problems may occurr, such as deadlock and efficiency degradation.

Avoid using more than one method on a shared object. If you need to use more methods on an object you're currently holding the lock of, you can apply the following strategies: with client-based locking, you have the client lock the server before calling the first methods, so that the lock's extent includes code calling the last method; with server-based locking, you add a method to the server that locks it, and call all needed methods, then unlock: the client then calls that method; with adapted server, you create an intermediary that performs the locking (this happens with server-based locking, when you can't change the server).

Keep synchronized sections as small as possible, because locks create delays and add overhead, so there should be as few and as short as possible.

## Final notes - best practices

### Wrap third party libraries
When dealing with third party libraries, it's always useful to wrap them inside custom classes. In this way you can decouple your code from the library interface, while still exploiting its functionality: this means that you can use your own interface (maybe an already existing one) instead of being forced to adapt your code to the third party interface, for instance rethrowing the library exceptions using local ones instead, or changing the way error situations are mapped to exceptions; then it will be easier in the future to move to a different library, having only one place in the code to change (the wrapper); in addition to this, it's easier to mock third-party calls inside tests.

### The special case pattern
It often happens that we need to add a special case to a piece of business logic. Consider this example:
```java
try {
    MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
    m_total += expenses.getTotal();
} catch (MealExpensesNotFound e) {
    m_total += getMealPerDiem();
}
```

Here we are using the occurrence of an exception to drive the business logic: if no meals is found for the given employee, a standard value should be used instead, but we are using the exception mechanism to know when this special business logic case occurrs. This is a bad practice, since exceptions should be used for cases outside of the business rules, they should be separated from the normal logic expressed by code. This is the typical situation where the "special case" pattern can be applied: the solution here is to create an interface representing the concept that has the special case, in this case it would be a `MealExpenses` type, and create two different implementations of this interface, one for the regular case, and the other for the special case. The client code only works with the interface, so it won't be bothered by the fact that the current case is special or not.
```java
// Client code
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();

// Service code
public class PerDiemMealExpenses implements MealExpenses {
    public int getTotal() {
        // return the per diem default
    }
}
```

### Test polymorphism to choose if to use static
The least important reason to make a method `static` is if it depends only on its arguments, and not on any notion of "current state" that may be encapsulated into an object. This is the least important reason, because it's easy to game this rule, moving state from the invocation object to an argument:

```java
HourlyPayCalculator.calculatePay(employee, overtimeRate)
```

Here, `calculatePay` might have as well be a method of an `Employee` class, where `employee` would have been the invocation object instead of an argument.
How to tell, then, when it makes sense to use static? The most important rule is checking if there's a need for that method to be polymorphic: would be useful if this method (or better, its signature) had different implementations according to different concrete types of `Employee` that we might use? If we realize that we could need to write multiple different versions of `calculatePay` related to different types of `Employee`s, then it's better to make `calculatePay` an instance method, rather than a static one.