# Types

- [Typeclasses](#typeclasses)
- [Common typeclasses](#common-typeclasses)

Every object in Haskell has a type, and every function must obey strict rules about the types of its arguments. In this way, the compiler can spot a great number of errors just checking if all values are used coherently with their declared types. Unlike other languages, such as C or Java, Haskell has *type inference*, meaning that it's not necessary to declare the type of a value, because Haskell can *infer* it from the context where it's used.
