# Variables and types

- [Constants](#constants)
- [Variables](#variables)
- [Scope](#scope)
- [Types management](#types-management)
- [Expressions](#expressions)
- [References](#references)


Values can be assigned to identifiers with the usual `=` operator. In Rust an assignment is like:
```rust
const PI:f32 = 3.14;
```

where the `const` modifier specifies the type of identifier we want to declare, which is followed by the identifier itself `PI`, and the type annotation `f32` introduced by the `:` character. Next to this the `=` operator assigns the `3.14` value to the identifier.


## Constants

The most common identifiers are variables and constants. Rust supports two types of constants: they're both static in lifespan, meaning that they're created at the beginning of the program, and destroyed only at the end; they both also have global accessibility; additionally, unlike variables, constants must always be declared with their type annotation.

Constants defined with the `const` keyword are just inlined during compilation, meaning that all their occurrences in the code are replaced with their actual value:
```rust
const PI: f32 = 3.14;
```

The fact that they are inlined means that you can never be sure of what exact memory is being used at every occurrence of the constant, so new memory might be allocated each time a new value is used.

On the contrary, constants declared with the `static` keyword behave like global immutable variables, meaning that they are allocated once at the beginning of the program execution, and as such their values are stored in fixed memory locations, and there is always exactly one instance of each value:
```rust
static MAX_HEALTH: i32 = 100;
```


## Variables

Variables are instead declared with the `let` keyword:
```rust
fn main() {
	let energy = 5;
}
```

Unlike constants, variables can never be declared in the global scope, but only inside a block (local scope). The Rust compiler can very often infer the type of a variable from the value that's being assigned to it: as such, it's often possible to omit the type annotation from the declaration.

Variables in Rust are immutable by default. Unlike other languages like JavaScript and Java, where "constant" variables are those that cannot be reassigned to different values (while the values themselves can still be modified), variables in Rust cannot be reassigned, and their values cannot be modified, meaning that the following code generates an error:
```rust
let energy = 5;
energy = 6; // ERROR! Re-assigning an immutable variable
```

However, it's possible to reassign a variable if we explicitly use the `let` keyword again:
```rust
let energy = 5;
let energy = 6;
```

this also allows to change the type of the variable. Basically, `let` creates a brand new variable with the same name, in fact reassigning that name.

To allow a variable to be reassigned, or its value to be changed, we have to declare it using the `mut` modifier:
```rust
let mut energy = 5;
energy = 6;
```

here, however, it's not allowed to assign a new value of a different type, because, unlike the previous example, the variable has not been re-created but is still the same as defined before, thus with the same type.

A variable can be declared without being initialized, provided that we explicitly specify its type:
```rust
let energy: i32;
let energy2; // ERROR! Missing type
```

Despite not having been declared as `mut`, the variable can still be assigned the first time:
```rust
energy = 5;
```

At this point, however, it cannot be assigned again:
```rust
let energy: i32;
energy = 5;
energy = 6; // ERROR! Re-assigning an immutable variable
```

It's not possible to use a variable that hasn't been initialized:
```rust
let energy: i32;
println!("{}", energy); // ERROR! Using a non-initialized variable
```

When the values being assigned to identifiers are defined as literals, we can specify their exact types using suffixes instead of type annotations:
```rust
let energy = 5u8;
```

here we're specifying that the value `5` is in fact an 8-bit unsigned integer. If we had let the compiler infer the type, it would've gone with the default `i32` instead.


## Scope

The scope of a variable is the portion of code within which the variable exists. The scope of global constants is thus the whole program, because they exist from the beginning to the end of the program. The start of a scope is the point where a variable is declared. The end of a block (i.e. the closed curly brace `}`) always terminates a scope.

Variables are allocated as soon as they are declared (that's possible because the compiler must always know the type of the variable). They are immediately destroyed as soon as they go out of scope (when the execution reaches the end of the block within which the variable has been declared).

Variables within an inner block can shadow variables with the same name declared in outer blocks.
```rust
fn main() {
	let outer = 42; // `outer` available from here
	// `inner` NOT available yet
	{
		let inner = 3.14 // `inner` available from here
		// `outer` still available here

		let outer = 99; // inner `outer` shadows the outer one
		// `inner` still available here
		// outer `outer` NOT available here
	}
	// `inner` DESTROYED here
	// inner `outer` DESTROYED here
	// outer `outer` available again here
}
// `outer` DESTROYED here
```


## Types management

Rust doesn't perform automatic casts of any type. For example:
```rust
let points = 10i32;
let mut saved_points: u32 = 0;
saved_points = points; // ERROR! Type mismatch
```

in other programming languages a safe (read: without data loss) cast would be performed here from the narrower `i32` to the larger `u32`. In Rust, however, this is not permitted. We can still perform this cast explicitly, using the keyword `as`:
```rust
saved_points = points as u32;
```

This would also work in case of unsafe casts, thus when information is lost for instance because of decimals truncation. Not all types can be cast to others: for example it's not possible to cast a `&str` to a `i32`.

It's possible to create alias of existing types, much like C's `typedef`, with the `type` keyword:
```rust
type MagicPower = u16;
```

usually to give a more semantic name to existing types.


## Expressions

Most of Rust constructs are in fact expressions, meaning that they can be evaluated. This doesn't apply to `let` declarations and assignment statements, however, which cannot be evaluated. This, by the way, means that a statement like:
```rust
let a: i32;
let b: i32;
a = b = 2; // ERROR! Type mismatch
```

is not allowed, because `b = 2` is not an expression, and thus doesn't return a value that can be in turn assigned to `a`.

To be more precise, statements that are not expressions, still return a special value, that is called *unit value* `()`. Thus, the statement `b = 2` will in fact return `()`, and since this is not of type `i32` (it is of its own type, different from any other), it cannot be assigned to a `i32`.

A block is always an expression, meaning that it can be assigned:
```rust
let n1 = {
	let a = 2;
	let b = 5;
	a + b
};
```

here we are calculating the resulting value of the block through a series of statements. To return the result, however, we have to avoid adding the semicolon to the last line of the block. We cannot use `return` inside the block, though, because it would return from the outer function that contains that block, instead of from the block itself. The result thus returned is used as the r-value of the assignment statement.

Returning the value of an expression by means of leaving out the semicolon at the end of the line works with all blocks, including function definitions:
```rust
fn myFun() -> i32 {
	1 + 2 // notice the missing semicolon: this function returns 3
}
```

Had we added the semicolon at the end of the line, the resulting value would've been consumed, and the unit value `()` would've been returned instead. This means that we must pay special attention to the usage of semicolons, to avoid getting the unit value `()` where we were expecting an actual value:
```rust
let n2 = {
	let a = 2;
	let b = 5;
	a + b; // the unit value () is returned here
};
```

in this case `n2` would be assigned `()` instead of the integer result.

In Rust conditionals are expressions as well:
```rust
let active = if health >= 50 { true } else { false };
```

again, we must omit the semicolon to let the two values `true` and `false` be returned by the block, and assigned to the variable. Of course, conditional constructs must return values of the same type for them to be used as expressions. We can return the value of an `if` block from a function, but we must be ware that all branches should be part of the `if`:
```rust
fn my_fun(v: i32) -> i32 {
	if v > 0 {
		2 * v // ERROR! We are not returning the value of `if` because we are returning the next line instead
	}
	3 * v
}
```

in this case the last line is correct, since we're avoiding to consume its value, in order for it to be returned, but we are doing the same inside the `if`, which is not being returned, and as such must not be used as an expression (the actual error message would say that the unit type `()` was expected, but an `i32` was found instead). To fix this, we either have to explicitly `return` from inside the `if`, or incorporate also the last line inside the `if`, and return the `if` itself as an expression:
```rust
fn my_fun(v: i32) -> i32 {
	if v > 0 {
		return 2 * v;
	}
	3 * v
}
```

or:
```rust
fn my_fun(v: i32) -> i32 {
	if v > 0 {
		2 * v
	} else {
		3 * v
	}
}
```


## References

Let's consider the following example:
```rust
let health = 32;
let mut game = "Space Invaders";
```

The `health` value contains a primitive type of fixed and limited size, which as such is stored on the stack at a certain memory address. The `game` value contains a value of type `&str`: while integers like `i32` have fixed size known at compile time, strings can be of any size, and thus cannot be stored on the stack. The value of `game` is thus allocated on the heap, and the memory address of the first byte of the allocated space is stored in a variable on the stack, which then acts as a reference to the real value. This means that `game` really contains the address of the first byte on the heap where the actual string has been allocated.

To know the address where a value is stored, we can use the `&` operator:
```rust
println!("{:p}", &health);
println!("{:p}", &game);
```

the `:p` control format is used to treat the given value as a pointer, thus printing the memory address it contains, instead of dereferencing it and printing the pointed value.

The ampersand `&` is also used to denote reference types. For example:
```rust
let a:i32 = 12;
let b:&i32 = &a;
println!("{}", a); // prints 12
println!("{}", b); // prints 12
println!("{:p}", b); // prints memory location of a
```

here `b` is a reference to an integer `&i32`, and is initialized with the address of the integer `a`. Notice that `println!` automatically dereferences pointers to get the pointed value: to print the actual contents of a pointer (that is, a memory location), we have to explicitly use the pointer format string `:p`.

In this case, we created a reference that points to a value allocated on the stack. The same thing can be done to reference values allocated on the heap:
```rust
let a:&str = "Some string";
let b:&str = &a;
println!("{:p}", a);
println!("{:p}"; b); // prints the same memory location as the previous
```

here `a` is already a reference to the string `"Some string"`, because strings cannot be allocated on the stack, and as such we can only handle them with references. What we did with `b`, however, was creating another reference to the same memory location pointed to by `a`, i.e., and *alias*: no copy of the string was made, just another pointer was created to the same object.

Now, if we have a reference type, like `&i32`, we can get the value that is being pointed to, instead of the reference, using the dereferencing operator `*`:
```rust
let a:i32 = 12;
let b:&i32 = &a;
let c:i32 = *b;
println!("{}", b); // prints 12
println!("{}", c); // prints 12
println!("{:p}", b); // prints 0x7ffdcec1f924
println!("{:p}", &c); // prints 0x7ffdcec1f934
```

here in the declaration `let c:i32 = *b` what happened was that we first dereferenced the `b` pointer, to get the value it was pointing to, that is `12`, and then we assigned this value to the new `c` variable.

```rust
type Text = &'static str;
let a:Text = "Some string";
let b:&Text = &a;
let c:Text = *b;
println!("{:p}", a); // prints 0x55e372099ac0
println!("{:p}", b); // prints 0x7ffe4e3893e8
println!("{:p}", c); // prints 0x55e372099ac0
```

here we are defining a new type `Text` which is the same as `&str`, to avoid cluttering the code with too many `&`'s, since we want to handle references to `&str`. First we allocate a string `"Some string"` on the heap, and point to it with `a`, which thus contains the location of that string on the heap. Then in `let b:&Text = &a` we get the address of `a`, and store it inside a `&Text`, named `b`: here, however, we're not getting the address *contained in* `a`, but the address *of* `a`, meaning the address of the variable in the stack that contains the address of the original string; this is why when we print `b` we get a totally different address. Since `b` is a pointer to `a`, when we dereference `b`, we get `a`, which is a pointer to the original string: this is why when we print `c` we get the original address again.

It's not possible to change an immutable variable through a reference:
```rust
let a:i32 = 12;
let b:&i32 = &a;
*b = 2; // ERROR! Cannot assign to immutable variable
```

Furthermore, references are by default immutable, even when they point to a mutable variable:
```rust
let mut a:i32 = 12;
let b:&i32 = &a;
*b = 2; // ERROR! Cannot assign to immutable variable
```

this is basically because `b` has been declared as immutable (which is the default for all declarations). However, it's possible to create a mutable reference:
```rust
let mut a:i32 = 12;
let b:&mut i32 = &mut a;
*b = 2;
```

Notice how mutable references are distinct types from regular (immutable) references, as in `&i32` versus `&mut i32`.

In this case, `b` is an *immutable binding to a mutable reference*, meaning that the reference is mutable (we can change the value that is referenced to), but we cannot assign a different pointer to the `b` identifier:
```rust
let a:i32 = 12;
let b:&i32 = &a;
let c:&i32 = &a;
c = b; // ERROR! Cannot re-assign to immutable variable
```

This is quite trivial, since this is how all variables work. To allow `c` to be assigned again, we have to declare it as mutable (which is different than declaring that it points to a mutable value, as it was in the example before):
```rust
let a:i32 = 12;
let b:&i32 = &a;
let mut c:&i32 = &a;
c = b;
```

Then, following this reasoning, we could think it would be possible to also create mutable bindings to mutable references, like `let mut c:&mut i32 = &mut a`, but this doesn't actually work because it's not possible to create more than one mutable reference to the same variable.
