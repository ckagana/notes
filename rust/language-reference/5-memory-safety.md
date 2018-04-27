# Memory safety

- [Memory layout](#memory-layout)
- [Copying and cloning](#copying-and-cloning)
- [References](#references)
- [Lifetime](#lifetime)
- [Mutable references](#mutable-references)


## Memory layout

When the operating system spawns a new process from a given program, it assigns some space in memory to it (in addition to the memory needed to carry the actual binary program). This space comprises three areas: the *static area*, the *stack* and the *heap*:
- the *static area* is used to store all variables and constants that are "static", meaning that they are created at the beginning of the process execution, and deleted at its end;
- the *stack* contains variables local to function calls: every time a new function call is issued, a new frame is pushed to the stack, containing the local variables of that function (including the actual arguments), and when the function returns, its frame is popped from the stack, causing the destruction of all the variables saved on it;
- the *heap* is used for dynamic memory allocation, since it's much bigger, and data stored on it is always available, unless manually free'd. The heap is usually used for data structures with non fixed size, like strings, dynamic arrays, lists, etc.
Accessing the stack is much more efficient than accessing the heap, because the heap must first be scanned to find out a chunk of free memory large enough to store our data, while on a stack frame we know at compilation time already how big the variables to be allocated will be.


## Copying and cloning

When a value or a variable is assigned to another, there can be different possible behaviors. The most simple is the copy, where the literal value, or the value contained in the existing variable, is copied, and assigned to the new variable:
```rust
let n = 42u32;  // an instance of 42 is created and stored in n
let n2 = n;     // the value of n is copied, and assigned to n2,
                // so that now there are two copies of 42
life(n);        // the value of n is copied, and assigned to m

fn life(m: u32) -> u32 {
    let o = m;  // the value of m is copied, and assigned to o
    o
}
```

in the previous snippet of code, the value `42` turns out to be copied 4 times, in order for it to be stored in the variables `n`, `n2`, `m` and `o`. Many built-in types, such as `u32` and `i64`, express this copy behavior: in general, all types implementing the `Copy` trait work like this.

To implement the `Copy` trait, all fields must also implement `Copy`:
```rust
struct MagicNumber {
    value: u64      // u64 implements Copy, so MagicNumber can, too
}

impl Copy for MagicNumber {}
```

otherwise, we can annotate our type with the `Copy` attribute:
```rust
#[derive(Copy)]
struct MagicNumber {
    value: u64
}
```

Now, our `MagicNumber` values will undergo a deep copy avery time they are assigned to a variable:
```rust
let mag = MagicNumber { value: 42 };
let mag2 = mag;     // a deep copy happens here
```

We can do the same thing explicitly, rather than implicitly, with a *clone* operation:
```rust
#[derive(Clone)]
struct MagicNumber {
    value: u64
}

let mag3 = mag.clone();
```


## References

References contain the memory address where values are stored:
```rust
let q = &42;
println!("{:p}", q); // prints 0x564a7ea96c00
```

in this case the value `42` has type `i32`, and is stored on the stack. The variable `q`, therefore, contains the address of the new instance of the value `42` on the stack, and is therefore called a *reference*. Since we are working with addresses instead of actual values here, we can do some interesting stuff, like creating several references pointing all to the same value:
```rust
let q = &42;
let m = q;
let n = m;
println!("{:p}\n{:p}\n{:p}", q, m, n);
// Prints:
// 0x5602bcd65d00
// 0x5602bcd65d00
// 0x5602bcd65d00
```

Comparing this with the "copy" behavior we saw before, we understand that no copy of the original value is being done here: rather, there is only one copy of the value, which is being referenced by three different variables. We can still say, however, that the references themselves are copied, since the address of that single `42` is indeed copied into the three different references, and it's the one being printed.


## Lifetime

The lifetime of a value is the part of execution time starting when the value is created (usually when a variable is initialized with that value), and ending when no more references to that value are available (all references have been destroyed by going out of scope). Normally, lifetime is automatically inferred looking at the code: for example the lifetime of a local variable is from when the variable is declared, to when the function returns.

When using values, instead of references, lifetime poses no problem at all, and can be totally handled by the borrow checker, meaning that values are created and destroyed automatically when they appear in the code, and when they go out of scope. This is due to the fact that values are always copied when variables are assigned to one another, and not more than one variable can contain the same value.

However, when using references, lifetimes can become an issue, because suddenly more variables (references) can point to the same value in memory, possibly leading to situations like this:
```rust
{
    let r;                  // ------+-- 'a
                            //       |
    {                       //       |
        let x = 5;          // -+------- 'b
        r = &x;             //  |    |
    }                       // -+    |
                            //       |
    // ERROR: `x` does not live long enough
                            //       |
    println!("r: {}", r);   //       |
}                           // ------+
```

what happens here is that at the end of the inner block, `x` is deallocated, and the value it was pointing to is destroyed: this means that after the inner block, `r` would be pointing to some non allocated memory, being thus a dangling pointer.

What the borrow checker of Rust is doing to detect these issues is comparing references lifetimes. The lifetime of `r` starts at its declaration in the outer block, and ends at the end of the outer block; let's call it `'a`. The lifetime of the value that will be referenced starts at its declaration in the inner block, and ends at the end of the inner block; let's call it `'b`. Now, what happens is that the reference lifetime `'b` is shorter than the lifetime of `r`, `'a`, and this is what causes the dangling pointer.

Let's consider the following example instead:
```rust
{
    let x = 5;      // ------+-- 'b
                    //       |
    let r = &x;     // --+------ 'a
                    //   |   |
    println!("r: {}", r);
                    // --+   |
}                   // ------+
```

in this case the lifetime of the referenced value, still `'b`, is larger than the lifetime of the reference `'a`, meaning that the when the reference is used, the value is still available in memory.

So, lifetimes become tricky especially when different scopes are involved, as it's the case with functions. Let's consider the following code:
```rust
let string1 = String::from("abcd");
let string2 = "xyz";

let result = longest(string1.as_str(), string2);
println!("The longest string is {}", result);

// ERROR: missing lifetime specifier
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

Here the `longest` function takes string slices to avoid taking ownership over the given strings, since it doesn't need it. This, however, means that we are creating references to the two existing strings `string1` and `string2`.

When the `longest` function is being written, nobody knows where the values referenced to by the arguments are coming from, and thus what their lifetime is:
```rust
let first = "first";
{
    let second = "second";
    let result = longest(first, second);
}
println!("result: {}", result);
```

considering the implementation of `longest`, the `println!` macro may be trying to dereference a dangling pointer, or not, depending on which is longer between `first` and `second`: if `first` is longer, then the value with the bigger lifetime is assigned to `result`, and `println!` can be safely called; if, instead, `second` is longer, then `result` will contain a value that will be destroyed at the end of the inner block, and `println!` would be trying to dereference a dangling pointer!

When we define a function taking only one reference parameter (it can have other parameters, as long as they're not references), and returning a reference, then the compiler is sure that the returned value will have the same lifetime as the one of the given argument (because we cannot return a reference of a value created locally, since it would automatically become a dangling pointer as soon as the function return). However, when we pass more than one reference parameters, and return a reference, the compiler has no way to know which lifetime the returned reference will have, among the ones of the given reference parameters:
```rust
// ERROR: missing lifetime specifier
fn test(t: &str, u: &str) -> &str {
    t
}
```

here the compiler doesn't know (in this case it's obvious, but in general it cannot know) if the returned reference will have the lifetime of `t`, or the lifetime of `u`, and thus it cannot check if the calling code is trying to reference deallocated memory.

The solution to this problem is telling the compiler what the relationships among the input references and the output reference are:
```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

here we are constraining the given references to have exactly the same lifetime, that will then be also the same as the returned reference. This way the compiler will know exactly what the lifetime of the returned value is, and will be able to use this information to proceed checking the calling code.

When we call this function with references with different lifetimes, the concrete lifetime assigned to `'a` will be equal to the smaller of the lifetimes of the actual arguments: this ensures that a proper lifetime check of the calling code can be performed, since the return value will be considered dropped the moment the shortest-lived of the argument is dropped.

Here `'a` is the syntax used to define lifetimes: a `'` character following by the identifier of the lifetime. The identifier can be any valid identifier, but in Rust it's common practice to use single, lower-case letters. A common example of lifetime specifier is `'static`, used for references pointing to static values. Of course the important thing here is not what the specific lifetime is, since we can use any identifier, but rather which of the given input lifetime will be used as output:
```rust
fn test<'a, 'b>(t: &'a str, u: &'b str) -> &'a str {
    t // OK
}
```

here we accept a parameter of a different lifetime `'b` than the returned one `'a`, but since in the code we are returning `t`, which is a `'a`, the compiler doesn't complain. We could also have omitted `'b` entirely, since it's not relevant to determine the lifetime of the return value.

```rust
fn test<'a, 'b>(t: &'a str, u: &'b str) -> &'b str {
    t // ERROR
}
```

here instead we are stating that a `'b` will be returned, but in fact the code is returning a `'a`, and the compiler rightfully complains.

```rust
fn test <'a, 'b>(t: &'a str, u: &'b str) -> &'a str {
    if t.len() > u.len() {
        t
    } else {
        u // ERROR
    }
}
```

here we are stating that a `'a` will be returned, while instead a `'b` could instead be returned, thus causing a compilation error.

Lifetime declarations are also needed inside struct definitions, in case their fields are references. In this case the problems are exactly the same we saw with functions, simply because the constructor is the function where existing values are taken, and assigned to reference fields, and thus there's the risk that those values will be dropped while still being used by the struct fields:
```rust
struct ImportantExcerpt<'a> {
    part: &'a str
}
```

### Resources
- https://doc.rust-lang.org/book/second-edition/ch10-03-lifetime-syntax.html


## Mutable references

When we allow a reference to be mutable, we introduce yet another problem. This may seem a simple issue if we are thinking of primitive values like `42`, but if we just think about a dynamic data structure like a string, when we want to change it, for example causing it to become bigger, it may not fit anymore in the space reserved to it in memory, and thus a new block of memory large enough must be allocated; however, this would mean that the address of the value would change, and then all other references to it would become dangling pointers!

Rust prevents this situation by making it so that when a mutable reference is created for a value, no other references to its value, mutable or not, can be used:
```rust
let a = "Test".to_string();
let b = &a;
let c = &mut a; // ERROR: cannot borrow immutable local variable `a` as mutable
```

here we are trying to create a mutable reference to the `a` resource, while another immutable one existed. If we were allowed to do that, we could have caused a reallocation of `a` to a different address, making `b` a dangling pointer.

```rust
let mut a = "Test".to_string();
let b = &mut a;
let c = &a; // ERROR: cannot borrow `a` as immutable because it is also borrowed as mutable
```

here we are trying to read `a` through an immutable reference, while a mutable reference to it already existed. If we were allowed to do that, we could have ended up trying to read the object with `c`, after it had been reallocated by some action performed by `b`, getting again to a dangling pointer situation.

The solution to this is that in Rust references can never be used to mutate values. Then, it's possible to use mutable references to change values through a reference, but as soon as a mutable reference to a value is created, all other references to that value are unusable (neither to read or write), and only one mutable reference can exist at the same time for a given value.


## Ownership

Memory is no different than any other resource, like files, database connections, or open sockets, and as such must be properly handled. The C++ language helped popularize the concept of RAII (Resource Acquisition Is Initialization): a common bug found in programs dealing with resources (including memory) is that the program acquires a resource at a certain time, but then forgets to release it, creating memory leaks, or release it too early, creating dangling pointers. RAII elevates resource management to an higher semantic level, stating that a resource connection should be encapsulated into its own type, where the instantiation of the type corresponds to the acquisition of the resource, and the destruction of the instance corresponds to the freeing of the resource. We say, thus, that that object *owns* the resource, and as such is the only one allowed to make operations with the resource.

Rust implements RAII by default:
```rust
let mut a = 42;
let mut b = a;
```

here, the `a` object is created at the same time the `42` value is allocated in memory, so the memory resource is acquired during initialization. At that point, `a` owns that `42` value, and no other object can act upon it. When `a` will go out of scope, it will be deleted, and the resource for the `42` value will be freed as part of the destruction procedure. When the assignment `b = a` is performed, as we know, a copy of the resource is performed, creating thus another, different, resource, which happen to have the same value as the first one, and this second copy is instead owned by `b`, with the same ownership rules.

The previous example, however, works only with values implementing `Copy`, which in general are primitive values. With reference values, the assignment operation usually causes *ownership move*, meaning the transfer of ownership of a resource from one object to another:
```rust
struct Alien {
    planet: String,
    n_tentacles: u32
}

let mut klaatu = Alien { planet: "Venus".to_string(), n_tentacles: 15 };
let kl2 = klaatu;
println!("{}", klaatu.planet); // ERROR: use of moved value 'klaatu.planet'
```

The assignment `let kl2 = klaatu` moves the ownership on the `Alien` value from `klaatu` to `kl2`. Since the variables are mutable, we cannot allow two references to mutate the same object, because one of them could destroy it, or cause it to be reallocated to a different address, making the other a dangling pointer. Thus, once the ownership have been moved, Rust forbids us to use the previous owner in any way.

Instead of moving the ownership, though, we can *borrow* it, using a reference:
```rust
let kl2 = &mut klaatu;
```

While a move permanently moves the ownership to a new reference, a borrow is only temporary, because after the reference holding the borrow goes out of scope, the ownership returns to the original owner, which can resume using the resource normally. While the resource is borrowed, however, the original owner is still prevented from using it.
```rust
let mut klaatu = Alien { planet: "Venus".to_string(), n_tentacles: 15 };
{
    let kl2 = &mut klaatu;
    kl2.n_tentacles = 14;
    println!("{} - {}", kl2.planet, kl2.n_tentacles); // prints: Venus - 14
}
klaatu.planet = "Pluto".to_string();
println!("{} - {}", klaatu.planet, klaatu.n_tentacles); // prints: Pluto - 14
```

in this example, after the `kl2` references has been destroyed at the end of the inner block, the borrowed resource is owned again by the original owner `klaatu` which is free again to use it.


### Ownership and `self`

Struct's methods can be passed the `self` parameter, representing the current invocation object. Like any other parameter, it can be passed by value, by reference, by mutable value, or by mutable reference. Let's see what happens when we pass `self` by value or by reference:

```rust
struct Test {}

impl Test {
    fn test(self) {
        println!("Pass by value: {:p}", &self);
    }

    fn test1(&self) {
        println!("Pass by reference: {:p}", self);
    }
}

let test = Test{};
println!("Local variable: {:p}", &test);    // Local variable: 0x7ffdd68320a0
test.test1();                               // Pass by reference: 0x7ffdd68320a0
test.test();                                // Pass by value: 0x7ffdd6832010
```

We see here that when `self` is passed by value, a copy of the invocation object is actually passed to the method! This means that in `test.test()`, `self` is not the `test` invocation object, but a copy of it, meaning that if that method was modifying the object, the invocation object wouldn't be the one being modified.

When we use a reference to the invocation object, we must deal with ownership issues. For example:
```rust
struct Test {
    test: String
}

impl Test {
    fn new(test: String) -> Self {
        Self{ test }
    }

    fn test(&self) -> String {
        self.test // ERROR: cannot move out of borrowed content
    }
}
```

Here the `test()` method takes `self` by reference: this means that `self` is actually just borrowed by the body of the `test()` method, which in turn means that the invocation object is not owned by the `test()` method. Now, what this method is trying to do, is moving the `test` field to the calling client, outside of the method body itself: this, however, cannot be done, because `test()` doesn't have ownership on `self.test`.

To fix this code we could do basically two things: the first is copying or cloning the `test` field, so that the new copy would be owned by the `test()` method, which can then do whatever it wishes with it. If we want to avoid wasting memory, and keep working with the original object, instead, we can return a reference to the field, instead of a value, because references don't take ownership:
```rust
fn test(&self) -> &String {
    &self.test
}
```


### Mutably borrowing

Let's consider the following example:
```rust
struct Response {
    value: String
}

impl Response {
    fn new(value: String) -> Self {
        Self{ value }
    }

    fn json(&mut self) -> &String {
        &self.value
    }
}

struct Client {
    response: Response
}

impl Client {
    fn new(response: Response) -> Self {
        Self{ response }
    }

    fn request(&self) -> &Response {
        &self.response
    }
}

let response = Response::new("Hey, Joe".to_string());
let client = Client::new(response);
println!("{}", client.request().json()); // ERROR: Cannot mutably borrow immutable content
```

Here, the `Response` struct is taking `value` with a move in `new()`, thus taking ownership on it, and then borrowing it to callers in `json()`. The same way, `Client` is taking ownership on `response` in `new()`, and borrowing it in `request()`. The spin here is that now `json()` needs to mutate the invocation object, thus taking `self` as `&mut`: this makes the `Response` borrowed by `request()` unfit to be called `json()` on, since it's borrowed immutably.

Since `request()` is doing a borrow, we cannot hope of changing the mutability of the `response` object; at this point the only two things we can do are either cloning the `Response` object into one we have ownership on (and that can thus be made mutable), or changing `request()` so that it knows that it has to return a mutable reference:
```rust
...
impl Client {
    ...
    fn request(&mut self) -> &mut Response {
        &mut self.response
    }
}
...
let mut client = Client::new(response);
...
```
