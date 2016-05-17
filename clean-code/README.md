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