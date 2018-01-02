# Modules and crates

- [Modules](#modules)
- [Crates](#crates)
- [Tests](#tests)

Rust supports modular development by providing *crates* and *modules*. A *crate* represents a library or package that can be distributed and managed through the Cargo package manager. A *module* is the code organizational unit, useful to split code into multiple files, and to organize code items like traits and structs into namespaces.


## Modules

Modules can be nested, and every crate contains at least the top-level root module. A new module is implicitly defined for every distinct file containing Rust code: this means that if we create a `game1.rs` file, the compiler will automatically create the `game1` module, containing everything defined inside `game1.rs`. Alternatively, we can explicitly define modules like:
```rust
mod game1 {
    // code here
}
```

this way we can define more than one module in the same file.

Items defined inside a module are by default private to that module, meaning that they're only visible in it. To make an item public from a module we have to use the `pub` keyword:
```rust
mod game1 {
    // Not visile outside game1
    fn func1() {
    }

    // Visible outside game1
    pub fn func2() {
    }
}
```

Items of a module can be accessed from outside that module with the notation `module::item`:
```rust
mod smt {
    pub fn func() {
    }
}

smt::func();
```

When a struct is defined inside a module, its fields are visible from within that module, but hidden from outside it. To make struct fields visible from outside the module where the struct has been defined, again we have to use `pub`. Additionally, the struct itself must be declared as `pub` to be visible from outside its module:
```rust
mod smt {
    struct Smt {
    }
}

let s = smt::Smt {}; // ERROR: struct `Smt` is private
```

```rust
mod smt {
    pub struct Smt {
        smt: u32
    }
}

let s = smt::Smt { smt: 12 };
println!("{}", s.smt); // ERROR: field `smt` of struct `smt::Smt` is private
```

```rust
mod smt {
    pub struct Smt {
        pub smt: u32
    }
}

let s = smt::Smt { smt: 12 };
println!("{}", s.smt); // prints 12
```

We can avoid prefixing the name of an item with its module every time by doing an *import*, which is done with the keyword `use`. We can create aliases with `use ... as`:
```rust
use game1::func2;           // imports func2 from game1
use game1::func1 as gf1;    // imports func1 from game1 with the alias gf1
use game1::{func2, func3};  // imports multiple items with the same statement
use game1::*;               // imports all items from game1
```

If a module is defined in a different file, it's not enough to use `use` though: we also need to declare the module that is defined outside the current file, with the `mod` keyword:
```rust
mod game1;

use game1::func2;
// ...
```

A `mod game1` statement will look for either a file named `game1.rs` in the current directory, or a file named `mod.rs` inside a `game1` directory.


## Crates

Crates are the unit of compilation in Rust, meaning that if a crate is composed of multiple files, the Rust compiler won't compile each single file; instead, all files will be compiled together, as if all definitions were crammed into a single file.

Crates have an attribute called `crate_type`, which defines the type of crate. There are several types of crates: the easiest distinction we can make is between *binaries* and *libraries*.

A *binary* is a crate containing the `main` function in the root module, and it's meant to be executable: its type is `bin`. On the other hand, there are several kinds of *libraries*: the possible `crate_type`'s are `lib`, `rlib`, `dylib`, and `staticlib`, and they represent different kinds of statically or dynamically linked libraries.

Binary crates are automatically created if the `main` function is detected in the root module, so no special compilation flag is needed. To create a library crate, instead, we can compile it like:
```bash
$ rustc --crate-type=lib --crate-name=mycrate structs.rs
```

this will create a file named `libmycrate.rlib`. We can still avoid using special compiler flags to create libraries, if we add the `crate_type` attribute to the source code:
```rust
#![crate_type = "lib"]
#![crate_name = "mycrate"]
```

this has to be added at the top of the root module file.

To access modules defined inside different crates, we cannot simply declare the module with `mod` anymore, because the file containing the module is not part of the current crate. To access it, we need to use `external crate`, which will also declare a module with the same name of the crate:
```rust
extern crate game1;   // Import the external crate: the root module is also declared
use game1::func1;     // Import func1 from the root module of the crate
```


## Tests

In Rust projects tests are organized this way: unit tests are located in a `test` module, integration tests are put in a `lib.rs` file in a `tests` directory.

For example, let's say we have a `cube` library project, with a `lib.rs` main file, containing the following:
```rust
#[cfg(test)]

mod test;

pub fn cube(val: u32) -> u32 {
    val * val * val
}
```

here we are defining the `test` module with the `cube` function. The `cfg(test)` attribute ensures that this module will be compiled only when tests are run. Then we add unit tests for this code, in a file `test.rs`:
```rust
use super::*;

#[test]
fn cube_of_2_is_8() {
    assert_eq!(cube(2), 8);
}
```

here with `use super::*` we are importing all items defined in the `test` module: we reference it as `super` because it's the module defined in the root file of the crate. With `#[test]` we declare a function as a unit test, meaning that it will be automatically called by the test framework when tests are run.

Then we add integration tests, in a `tests/lib.rs` file:
```rust
extern crate cube;

#[test]
fn cube_of_4_is_64() {
    assert_eq!(cube::cube(4), 64);
}
```

during integration tests the system under test is usually executed as a separate process, instead of inside the same process as the tests, to allow services like Web servers to be launched and used from the outside by the tests. For this reason, integration tests are part of a separate library, and therefore to access the actual code we want to test we have to import it as an external crate, with `extern crate cube`.

Tests are finally run with `cargo test`.
