# Data structures

The most simple kind of data structure supported by Java are arrays, like `String[] arrayOfStrings = {"a", "b", "c"}`. Arrays, however, have fixed length, and don't provide useful higher level abstractions like queues, stacks, trees, etc., out of the box.

Java supports more advanced types of data structures, which are categorized in two families, both located in the `java.util` package: data structures implementing the `java.util.Collection` interface, and data structures implementing the `java.util.Map` interface. `Collection`s are meant to be containers of objects, while `Map`s are meant to be containers of key/value couples. Both collections and maps are generic: `Collection<E>` and `Map<E>`, where `E` is the type of object that is contained by a specific data structure; this also means that data structures must contain all elements of the same type.


## Collections

The `Collection<E>` interface provides several methods:
- `boolean add(E element)`: adds an element of the generic type `E` to the collection, and returns `true` in case of success. It can return `false` in certain cases depending on the implementation: for example if a specific collection doesn't allow duplicate values, and we try to insert an already-existing element. This method returns `UnsupportedOperationException` if the collection is read-only.
- `boolean remove(E element)`: removes the given element from the collection, and returns `true` in case of success. It returns `false` if the element we want to remove doesn't exist. This method returns `UnsupportedOperationException` if the collection is read-only.
- `boolean contains(E element)`: tells whether the given element is contained in the collection.
- `int size()`: returns the number of elements contained in the collection.
- `boolean isEmpty()`: tells whether the collection is empty.
- `Iterator iterator()`: returns a new iterator for this collection.

Collections also support methods like `addAll()`, `removeAll()`, and `containsAll()`, which take another collection as an argument, and apply to all elements of it.


