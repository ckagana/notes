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

Beware that you need to import a module not only when you want to use a data structure or trait defined in it, but also when another module that you already imported is using a method defined in it. For example, this is the typical situation that happens when using traits:
```
src/
    main.rs
    service/
        mod.rs
        service.rs
        service_interface.rs
```

where `service_interface.rs` defines the `ServiceInterface` trait with the `getValue()` method, and `service.rs` defines the `Service` struct, implementing `ServiceInterface`. Inside `main.rs`, if we do:
```rust
// main.rs
mod service;

use service::service::Service;

fn main() {
    let service = Service::new(2);
    println!("{}", service.getValue());
}
```

because we think that, after all, we are only using `Service` here, and not `ServiceInterface`, we would have a compiler error telling us that the `getValue()` method is undefined, and the reason for this is that the `getValue()` method isn't actually defined inside `Service`, but rather inside `ServiceInterface`, and as such to be able to use it we need to import also `ServiceInterface`:
```rust
// main.rs
mod service;

use service::service::Service;
use service::service_interface::ServiceInterface;

// ...
```


### Multi-modules projects

When a project is composed of multiple modules in multiple files, it's not enough to define them to allow them to be used:
```rust
// main.rs
mod mod1;
use mod1::smt;

fn main() {
    smt();
}
```
```rust
// mod1.rs
mod mod2;
use mod2::smt2;

pub fn smt() {
    smt2();
}
```
```rust
// mod2.rs
pub fn smt2() {
    println!("SMT2");
}
```

trying to run this code will produce the following error:
```
error: cannot declare a new module at this location
 --> src/mod1.rs:1:5
```

What we are trying to do is quite simple: `main` depends on `mod1`, which in turn depends on `mod2`; thus, `mod1` declares `mod2` to be able to use its functionality. This, however, is not accepted by the Rust compiler.

The reason why this happens is that modules in Rust can be organized so that they form a hierarchy of nested modules, and Rust put a lot of emphasis on the relation between modules and the file structure. One module can contain several sub-modules nested into it. For example, a program can use several modules, that don't depend on each other:
```rust
// main.rs
mod dep1;
mod dep2;

use dep1::some_fun1;
use dep2::some_fun2;
// ...
```
```rust
// dep1.rs
pub fn some_fun1() {
    // ...
}
```
```rust
// dep2.rs
pub fn some_fun2() {
    // ...
}
```

In this example, the program defined by `main.rs` depends on the two sub-modules `dep1` and `dep2`, and so they are declared in the module file `main.rs`. Don't forget that the program module is only defined inside `main.rs`, since modules cannot be defined in more than one file, and so all dependencies must be declared there.

What happens now if we get a level deeper in the hierarchy, for example requiring that `dep1` depends on `dep2`, meaning that `dep2` should now be a sub-module of `dep1`? Suddenly the file structure is not properly laid out anymore: `dep2`, which we want to be a sub-module of `dep1`, is instead defined in `dep2.rs` which is at the same level as `dep1.rs`. The only way to fix this situation is by moving `dep2` into a new position in the filesystem that properly represents the dependency hierarchy that we want to build. For this to happen, first of all we have to realize that `dep1` now contains sub-modules, and as such cannot be defined as a single file anymore, but with its own directory, like we were already doing for the main program module. Once `dep1` has got its own directory, it can contain `dep2` as a submodule:
```rust
// main.rs
mod dep1;
use dep1::some_fun1;
// ...
```
```rust
// dep1/mod.rs
mod dep2;
use self::dep2::some_fun2;
// use dep1::dep2::some_fun2;
pub fn some_fun1() {
    // ...
}
```
```rust
// dep1/dep2.rs
pub fn some_fun2() {
    // ...
}
```

here we moved the `dep2` declaration out of `main.rs`, because now `dep2` is not a direct dependency of the main program anymore, but of `dep1` instead. `dep1` now is defined inside `dep1/mod.rs`, and it's the one containing the `dep2` declaration now, since `dep2` is direct dependency of it. `dep2` is still defined as a single file, but this time it's contained inside the `dep1` directory, being dependency of `dep1`.

Notice also that `use` declarations *always* refer to the root module of the entire project. When we write something like `use dep2::some_fun2` we are telling Rust that there must be a module called `dep2` at the root of the project. This is a bit counter-intuitive, because we might be lead to think that `use` is relative to the current module, but this is not the case. If we wrote `use dep2::some_fun2` in `dep1/mod.rs`, we'd have had an "unresolved import" error, because `dep2` indeed doesn't exist at the root of the project. To write the fully qualified name of `dep2`, we have to start from the root, and list all its parent modules (only `dep1` in this case):
```rust
use dep1::dep2::some_fun2;
```

Additionally, since at this location `dep1` is also the current module, we can refer to it with the `self` shorthand instead:
```rust
use self::dep2::some_fun2;
```

Let's say we had an additional module, `dep3`, still sub-module of `dep1` (thus at the same level of `dep2`), and that `dep2` depended on `dep3`. In this case, `dep2` would've needed to reference `dep3` with a `use` statement: since we are inside a sub-module, we need to write the fully qualified name of `dep3` from within `dep2` code:
```rust
// dep1/dep2.rs
use dep1::dep3::some_fun3;
```

Not unlike the previous case, here we can also simplify this code as:
```rust
// dep1/dep2.rs
use super::dep3::some_fun3;
```

where `super` is a shorthand for the parent module, and the parent of `dep2` is of course `dep1`.


### Same module with multiple files

In Rust it's not possible to define a single module with more than one file, because every different file is regarded as a different module in itself. However, we can achieve the same result exposing sub modules.

Let's say we have a `service` module containing `ServiceInterface` and `Service`: of course we could define the trait and the struct in the same `service.rs` file, but we want to stick to the rule of having only one definition per file:
```
src/
    lib.rs
    service/
        mod.rs
        service.rs
        service_interface.rs
```

and we want to be able to reference those elements like:
```rust
// lib.rs
mod service;

use service::Service;
use service::ServiceInterface;
```

So, the first thing we'd think of doing would probably be something like:
```rust
// service/mod.rs
mod service;
mod service_interface;
```

This, however, is wrong, because `Service` can be actually found as `service::service::Service`, and `ServiceInterface` as `service::service_interface::Service`. What we actually need to do, is to expose these two elements directly from the `service` module, using a public import:
```rust
// service/mod.rs
mod service;
mod service_interface;

pub use service::Service;
pub use service_interface::ServiceInterface;
```

this way, from the point of view of the external code, it'd be exactly as if `Service` and `ServiceInterface` were defined directly within `service`, as if the service module was fully contained inside a single `service.rs` file.


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
