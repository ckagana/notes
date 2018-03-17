# Tests

- [Unit tests](#unit-tests)
- [Integration tests](#integration-tests)
- [Documentation tests](#documentation-tests)
- [Test doubles](#test-doubles)

In Rust tests are functions decorated with the `#[test]` annotation, which are run when launching `cargo test` from the command line. As usual, we may want to separate unit tests from integration tests.


## Unit tests

When we run `cargo test`, Cargo will look for all sub-modules containing tests, i.e. functions annotated with `#[test]`, that it can find, and run them. Tests are conventionally put inside the same module where the code to be tested is defined, or inside a different module named `test` (this module could really be called anyway, Rust doesn't care). For Cargo to be able to properly run tests, some conditions must be satisfied:
- the project must have a `lib.rs` file loading all first-level modules (or at least those that we want to be tested): this is needed even if we have a binary project with a `main.rs` file, because `cargo test` will *not* call our program as a binary, rather it will load our program as a library, and then call tests on it;
- if we want to write unit tests separately for a specific module, that module must support sub-modules (i.e. must be defined as a directory), because `test` is indeed a sub-module. Unit tests there will then be able to test the code of that module, and that of all its sub-modules as well (of course its sub-modules defined as directories will be able to define their own `test` sub-module as well);
- the `test` module must still obey the rules of all sub-modules: in particular, it must be loaded by the module root file, otherwise the Rust compiler won't be able to see it;
- we can use `#[cfg(test)]` at the beginning of a file or module definition, to prevent the code thus tagged from being compiled unless in the case tests are run: however, this is *not* necessary for tests to run. This makes sense to do, so that test code is not unnecessarily compiled when the program is run outside of the testing framework.
```
src/
    main.rs
    lib.rs
    functions.rs
    test.rs
    dep1/
        mod.rs
        dep2.rs
        test.rs
```
```rust
// main.rs
mod dep1;
mod functions;
use functions::get_result;

fn main() {
    println!("{}", get_result());
}
```

here we must declare both `dep1` and `functions` because they are at the same level of the hierarchy, so `dep1` is not a sub-module of `functions` (even if `functions` uses it), and as such cannot be declared from inside `functions`.

```rust
// lib.rs
mod test;
mod functions;
mod dep1;
```

here we must declare both `dep1` and `functions` for the same reason as `main.rs` (remember that `lib.rs` is run when `main.rs` is not run, and vice-versa, so they both act as entry-points for the program). Also, here we must declare `test` otherwise Cargo won't be able to load the `test` module to run the first-level tests.
```rust
// functions.rs
use dep1::smt;

pub fn get_result() -> i32 {
    smt()
}
```

here we can just use `use dep1::smt` since the module has already been declared in the root file (whichever it will be: `main.rs` or `lib.rs` according to the way these files are run). Not that we could have declared it here: it would have been an error since they are at the same level of the hierarchy.
```rust
// test.rs
#[cfg(test)]
use functions::get_result;

#[test]
fn test_get_result() {
    assert_eq!(3, get_result());
}
```

here, as before, we don't need to declare any module, since they've all been already declared in the root file. The `test` module must've been declared in the root file for this file to be loaded tough.
```rust
// dep1/mod.rs
mod test;
mod dep2;
use self::dep2::smt2;

pub fn smt() -> i32 {
    smt2()
}
```

this is the root file for the `dep1` module, and as such it needs to contain all declarations. In particular, we need to declare `test` so that the unit tests specific for this module can be loaded, and `dep2`, because it's used here.
```rust
// dep1/dep2.rs
pub fn smt2() -> i32 {
    3
}
```

this is a "leaf" module, meaning that it has no dependencies, and as such it doesn't reference any other module.
```rust
// dep1/test.rs
#[cfg(test)]
use super::smt;
use super::mod2::smt2;

#[test]
fn test_smt() {
    assert_eq!(3, smt());
}

#[test]
fn test_smt2() {
    assert_eq!(3, smt2());
}
```

here we're using `super` to refer to the fully qualified name of the `dep1` module, which is in fact the parent of the current `test` module; also, this test module contains tests for all modules of `dep1`.


## Integration tests

Integration tests are meant to test our program as a whole, from the outside: they're run loading the program in a separate process, and exercise its public API from the outside. To define integration tests for our project, we must create an additional crate named `tests`, living next to the main `src` directory:
```
src/
    lib.rs
    ...
tests/
    lib.rs
```

Then, inside `tests/lib.rs`, we can put the test code:
```rust
extern crate project_name;
use project_name::functions::get_result;

#[test]
fn test_get_result() {
    assert_eq!(3, get_result());
}
```

Since integration tests sit in a completely different crate than our project, the code we want to test must be imported as an external crate. Also beware that the `functions` module we are testing must've been declared as public, otherwise it won't be visible from outside the crate.

Also, it's important to notice that it's necessary for the project to have a `lib.rs` file (i.e. to be exposed as a library) for integration tests to work, because otherwise it would be impossible to import the project as an external crate inside the `tests` crate.


## Documentation tests

These tests can be a quick way to embed documentation *and* unit tests at the same time.
These tests have a special `//!` comment placed at the module level and can be run with `cargo test`.
You can then run `cargo doc --no-deps` and build the documentation too:

```
//! This is a documentation test
//!
/*! <-- begin a multiline comment
        to demonstrate the syntax
        and the proper tags needed
*/
//!
//! ## Example of how my module works
//! ```                                                               .
//! use my_crate::sum;
//! let a = 5;
//! let b = 6;
//! assert_eq!(11, sum(a, b));
//! ```                                                               .

fn sum(a: i32, b: i32) -> i32 {
    a + b
}
```

## Test doubles
@todo
