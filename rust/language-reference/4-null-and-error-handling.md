# `null` and Error Handling

- [`null` and `Option`](#null-and-option)
- [`Result`](#result)
- [Error handling](#error-handling)
- [Re-throwing errors](#re-throwing-errors)


## `null` and `Option`

Rust doesn't support the concept of `null`: instead, it provides the union type `Option`, which represents a value that can be present or not:
```rust
enum Option<T> {
    Some(T),
    None
}
```

a function that might return a value or not, will return an Option. If a value is present, the Option will be an instance of `Some`; if no value is present, the Option will be `None`.

For example, the `next()` method of iterators returns an Option, because if there is a next element to fetch, relatively to the current position of the iterator cursor, then a `Some` will be returned, containing the actual value; on the other hand, if the iterator's cursor has reached the last element, calling `next()` will produce `None`, indicating that there is no value anymore.

Another common scenario where to use Option is with user input: since we cannot control the user behaviour, we must account for the chance that the user won't insert a value in the user interface.

For example:
```rust
fn parse_value(value: i32) -> Option<i32> {
    if value < 0 {
        None
    } else {
        Some(value * 2)
    }
}
```

when calling this function like `parse_value(3)`, we would get an `Option` object in return. This could contain an `Ok` value, meaning that there is indeed some value, or `None`, meaning that the operation returned nothing (replacing `null`). To get the actual `i32` value, we would call `unwrap()`:
```rust
let value:i32 = parse_value(3).unwrap();
```

Now, if we called `unwrap()` on a `None`, we would get a program crash with a panic, so `unwrap()` should be called with great care. Usually what we want to do is to check the return value, perhaps with a `match`:
```rust
fn use_value(value: i32) -> Option<i32> {
    match parse_value(value) {
        Some(value) => Some(value + 3),
        None => None
    }
}
```

A better way to handle such a case, though, would be using the `map` method:
```rust
fn use_value(value: i32) -> Option<i32> {
    parse_value(value).map(|v| v + 3)
}
```

this would automatically return `None` without going through `map()` if the original result was `None`, otherwise it would wrap the new result produced by the given lambda in a new `Option`.


## `Result`

`Result` represents the result of a computation, that can be either successful, or an error:
```rust
enum Result<T, E> {
	Ok(T),
	Err(E)
}
```

if the computation was successful, its result (of type `T`) will be wrapped inside an instance of `Ok`. If the computation produced an error instead, the error will be returned, wrapped in an instance of `Err`, having type `E` (usually a string with the error message).

Rust uses Results as replacements for the exception mechanism that other languages use. This means that when we define a function that might throw an error, we should return a `Result` instead of the raw return type:
```rust
fn sqroot(r: f32) -> Result<f32, String> {
	if r < 0.0 {
		Err("Number cannot be negative!".to_string())
	} else {
		Ok(f32::sqrt(r))
	}
}
```

`Result` works very similarly to `Option`, in particular supporting `unwrap()` and `map()`.


## Error handling

When calling a function that returns a `Result`, we may want to check if an error happened:
```rust
fn myFunction(v: f32) -> f32 {
    let value = match sqroot(v) {
        Ok(result) => result,
        Err(_) => 0.0
    };

    0.2 * value
}
```

in a language with exceptions, the previous code would have looked something like:
```java
public float myFunction(float v)
{
    float value = null;
    try {
        value = sqroot(v);
    } catch (Exception e) {
        value = 0.0;
    }

    return 0.2 * value;
}
```

Here, in case of error we swallow the message and return a default `0.0`. But what if our function doesn't know how to handle the error? In this case it must let it bubble up to its own caller. In languages supporting exceptions, this is usually just a matter of not catching the exception, because the default behavior is to let exceptions bubble up the stack. In Rust, we need our function to return a `Result` as well instead:
```rust
fn myFunction(v: f32) -> Result<f32, String> {
    let value = match sqroot(v) {
        Ok(result) -> result,
        Err(error) -> return Err(error)
    };

    Ok(0.2 * value)
}
```

Now, this exact pattern is so common that Rust has a built-in language feature that does exactly this: the `?` operator:
```rust
fn myFunction(v: f32) -> Result<f32, String> {
    let value = sqroot(v)?;
    Ok(0.2 * value)
}
```

What `?` actually does, is calling the method that returns a `Result`, and if the result is an `Ok` it unwraps it and use the content as value of the expression, otherwise it returns the given `Err` from the function. Thus, we can use `?` only inside functions returning a `Result`, because `?` will always return an `Err` in case of error. Also, the `Result` returned by the function must match the type of `Err` that `?` is going to return: this means that if `sqroot` returns `Result<f32, String>`, our function inside which we are using `?` can return `Result<SomeType, String>`, but not `Result<SomeType, SomeErrorType>`, because `?` is trying to return `Err<String>` instead. This gets particularly handy when we need to chain method calls:
```rust
fn myFunction(v: f32) -> Result<f32, String> {
    let value = sqroot(v)?.floor();
    Ok(floor)
}
```

If we don't need to do additional computation on the `Result` returned by a function (like `sqroot` in this case), it's pointless to use `?`:
```rust
fn myFunction(v: f32) -> Result<f32, String> {
    // Ok(sqroot(v)?) pointless, do just this instead:
    sqroot(v)
}
```


## Re-throwing errors

A common technique in languages supporting exceptions is to re-throw them, either because the caller has more information, or because we want to wrap low-level exceptions with others belonging to the current module:
```java
public void myFunction() throws MyException
{
    try {
        lowLevelMethod();
    } catch (LowLevelException e) {
        throw new MyException(e.getMessage(), e);
    }
}
```

In Rust we can achieve the same result by using enumerations of errors:
```rust
// This would be defined in a low-level library
enum LowLevelErrors {
    LowLevelError(String),
}

fn low_level_function() -> Result<String, LowLevelErrors> {
    if (1 > 2) {
        Ok("Success".to_string())
    } else {
        Err(LowLevelErrors::LowLevelError("Error".to_string()))
    }
}
```
```rust
// This is our own library, using the low-level one
enum MyErrors {
    MyError(LowLevelErrors),
}

fn my_function() -> Result<String, MyErrors> {
    let message = low_level_function().map_err(MyErrors::MyError)?;
    Ok(format!("Message was: {}", message))
}
```

In a more realistic program we would've defined more than one error kind in both libraries. The method `map_err()` of `Result` is used to replace the original `Result` with a new one having a different error kind: this is where the re-throwing actually happens, because the original exception is replaced with the new one. Additionally, from the definition of `MyError` we see that it takes as a construction argument the previous error of type `LowLevelErrors`, so we are also preserving the original error. The `?` operator is again used to unwrap the result if the operation was successful, or return the error from the function otherwise.

Define multiple error types inside a common enumeration is also useful when a function may return multiple different error types: since a `Result` can only define one error type, we can use an enumerated type as error type, while the `Result` will be able instead to be returned with different actual error types.