# Data structures

- [Arrays and vectors](#arrays-and-vectors)
- [Slices](#slices)
- [Strings](#strings)
- [String slices](#string-slices)
- [Tuples](#tuples)
- [Structs](#structs)
- [Enumerations](#enumerations)


## Arrays and vectors

Rust supports bare arrays, which are fixed-sized collections of elements of the same type:
```rust
let aliens: [&str; 4] = ["Cherfer", "Fynock", "Shirack", "Zuxu"];
let zuxus = ["Zuxu"; 3]; // initialize the array repeating 3 times the given string
println!("{}", zuxus.len()); // prints "3"
```

Notice how the type of an array is made of the type of the elements contained, and their number, which must be known at compile time. As usual, these arrays are stored in contiguous areas of memory, and as such they support random access in constant time. A mutable array allows to change its elements, but not their number, thus it's still impossible to add or remove elements.

Arrays support the index operator `[]` to access their items given their index:
```rust
println!("The first item is: {}", aliens[0]);
println!("The third item is: {}", aliens[2]);
```

Pointers to arrays are automatically dereferences, without the need to use the `*` operator:
```rust
let pa = &aliens;
println!("Third item via pointer: {}", pa[2]);
```

To iterate over an array's elements:
```rust
for a in &aliens {
	...
}
```

The `Vec` type provides heap allocated arrays that can change their capacity:
```rust
let mut numbers: Vec<i32> = Vec::new();
let mut magic_numbers = vec![7i32, 42, 47, 45, 54];
let mut ids: Vec<i32> = Vec::with_capacity(25);
```

In the first and third case the type annotation is required, because it cannot be inferred from the construction statement, while in the second case the type of the elements can be inferred by the literals passed.
```rust
for n in numbers: {
	...
}
```

unlike arrays, vectors can be iterated over without using the `&` operator on the vector name. The index operator `[]`, however, works the same as in arrays.

The `push()` and `pop()` methods can be used to add a new element to, or remove from, the end of the array:
```rust
numbers.push(magic_numbers[4]);
let fifty_four = numbers.pop();
```


## Slices

If we just need a subset of an array or vector, instead of copying that subset we can create a view of that part of the array, called a *slice*:
```rust
let slc:&[i32] = &magic_numbers[1..4]; // get from the second to the fifth element, the latter excluded
let slc1:&[i32] = &magic_numbers[..]; // get all elements, in fact converting the vector into an array
let slc2:&[i32] = &magic_numbers; // identical to the previous
```

as we can see, the type of a slice is `&[T]`, where `T` is the actual type of the elements of the slice (and of the original array or vector). Since slices are just views on an existing object, i.e. pointers, we don't need to specify their size when creating them, because no memory has to be allocated. Slices are immutable and of fixed size, being just views of the original array or vector.


## Strings

In Rust all strings are valid sequences of UTF-8 bytes (including null bytes). Strings are actually character buffers, meaning that they can grow or shrink in size, and that characters can be added, removed or replaced.

An empty string can be created with:
```rust
let mut s1 = String::new();
let mut s2 = String::with_capacity(25);
```

Usually a string will be increased in capacity when new characters are added to it exceeding its current capacity; however, we can pre-allocate a certain number of bytes at the moment of creation if we know that we'll need them. In the previous example both strings have been declared as mutable, simply because it makes little sense to create an character buffer that cannot be modified later.

Strings are in fact backed by vectors of characters: this means that they support many of the operations supported by vectors, like the `push()` and `pop()` methods, in addition to the `push_str()` method, which is only available on strings:
```rust
let mut s1 = String::new();
s1.push('c'); // append a single character to the string
s1.push_str("some string"); // append another string to the string
```


## String slices

Since strings are vectors of characters, slices can be taken from them: these are called *string slices* and have the type `&str` (or `&[u8]`). String literals are in fact slices; as usual with array and vector slices, they are immutable and of fixed size:
```rust
let sl1:&str = "Some string";
let mut string1 = String::new();
string1.push_str(sl1);
let sl2:&str = &string1[2..4]; // "me "
let sl3:&str = &string1[..] // "Some string"
let sl4:&str = &string1 // "Some string"
```

String slices can be converted to strings:
```rust
let mut s = "Hello, World!".to_string();
```

When defining a function taking a string, it's a best practice to always require a string slice as parameter, instead of an actual string, because otherwise memory would be allocated for the local string argument:
```rust
fn how_long(s: &str) -> usize { s.len() }
```

We can get the underlying vector from a string slice with:
```rust
let magician = "Merlin";
let mut chars: Vec<char> = magician.char().collect();
```


## Tuples

While arrays and vectors can contain values of the same type, to collect values of different types we can use *tuples*:
```rust
let thor:(&str, bool, u32) = ("Thor", true, 3500u32);
let one = (1,); // one value tuple
```

We can access a tuple's fields with the dot notation:
```rust
let name:&str = thor.0;
let is_hero:bool = thor.1;
let power:u32 = thor.2;
```

Of course if the tuple has been declared as mutable, its fields can be changed:
```rust
let mut smt = ("Hey", 1);
smt.0 = "Joe";
smt.1 = 2;
```

tuples can be *destructured*:
```rust
let (name, _, power):(&str, _, u32) = thor;
println!("{} has {} points of power", name, power);
```

where the `_` character is used to discard a value we are not interested in, instead of storing its value in a variable.

Tuples can be compared or assign to one another only if they have the exact same type. Tuples are handy for returning multiple values from functions:
```rust
fn increase_power(name: &str, power: u32) -> (&str, u32) {
	if power > 1000 {
		(name, power * 3)
	} else {
		(name, power * 2)
	}
}
```


## Structs

Tuples are usually good for temporarily aggregating values of different types, but they're not very handy as proper data structures, mostly because you cannot assign meaningful names to either the structure itself, or its fields. Instead, Rust provides the `struct` type, which can be used as a record type.

Tuple structs are a kind of structs that resemble tuples very much:
```rust
struct Score(i32, u8);
let score1 = Score(73, 2);
```

these can be accessed exactly like tuples, with the dot notation (where fields are called as their index), or destructuring.

Proper structs, on the other hand, allow to assign names to their fields:
```rust
struct Player {
	name: &'static str,
	health: i32,
	level: u8
}
let mut pl1 = Player{ name:  "Dzenan", health: 73, level: 2};
```

with proper structs, we can access the fields by their name with the dot notation:
```rust
println!("Player {} is at level {}", pl1.name, pl1.level);
```

Structs can still be destructured like tuples and tuple structs:
```rust
let Player{ health: ht, name: nn, .. } = pl1;
pritnln!("Player {} has {} health", nn, ht);
```

in this case, however, the character to skip unwanted fields is `..` instead of `_`.

Unlike tuples, structs (both tuple structs and proper structs) define new types, meaning that the `pl1` variable of the preceding example has in fact the type `Player`.

As in arrays and vectors, also in tuples and structs pointers are automatically dereferences when accessing fields, without the need to use the dereferencing operator `*`:
```rust
let tpl = ("Hey", 1);
let tplp = &a;
println!("{}", tplp.0); // tplp is automatically dereferenced

struct Something {
	name: &'static str,
	value: u32
}
let smt = Something{name: "Hey", value: 1};
let smtp = &smt;
println!("{}", smtp.name); // smtp is automatically dereferenced
```


## Enumerations

While usually data structures are used to define sets of values that can contain any number of items, if we want to define a set that contains only a limited number of specific values, we can use enumerations:
```rust
enum Compass {
	North, South, East, West
}
let direction = Compass::West;
```

As usual, an enumeration defines a new type, so `North` for example, is of type `Compass`.

Enumerations in Rust can very well be used as typical enumerations, thus containing a fixed set of values. However, in Rust enumerations can also be used as union types:
```rust
enum PlanetaryMonster {
	VenusMonster(&'static str, i32),
	MarsMonster(&'static str, i32)
}
let martian = PlanetaryMonster::MarsMonster("Chela", 42);
```

this means that a `PlanetaryMonster` object can be either a `VenusMonster` or a `MarsMonster` (thus the union type). However, any number of values of these two types can be contained in the set of `PlanetaryMonster`, which doesn't happen with usual Enumerations.

