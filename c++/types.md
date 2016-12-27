# Types

- [Use precise types](#use-precise-types)
- [Use `const` wherever possible](#use-const-wherever-possible)


## Use precise types

Avoid abusing primitive types like `int`, where you can be more specific about the semantic of the object you're defining:

```c++
class Date
{
public:
    Month month() const;    // do
    int month();            // don't
};
```

In other words, if you're not doing arithmetic, maybe `int` is not a very descriptive type to use. The same can be said for `double`'s:

```c++
change_speed(double s);   // bad: what does s signify?
// ...
change_speed(2.3);
```

Here that `s` could mean either a new speed, or a speed increase: rather than relying on comments, express this meaning directly in the code:

```c++
change_speed(Speed s);    // better: the meaning of s is specified
// ...
change_speed(2.3);        // error: no unit
change_speed(23m / 10s);  // meters per second
```

### References
- https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md


## Use `const` wherever possible

Unexpected side effects are one of the main source of bugs in programs. C++ comes with the ability of enforcing immutability of objects at compile time. This is the case not only for regular variable definitions and return types:

```c++
const user& get_user(const string& user_name)
{
    const some_type local_variable;
    // ...
}
```

but also for invocation objects, in methods' definitions:

```c++
void some_type::some_method() const
{
    // ...
}
```

Here we are declaring that this method will never change the invocation object, and thus will never cause side effects. This declaration instructs the compiler to do a formal verification of this statement: in this way, if we forget that this method shouldn't cause side effects, and add some statement changing `this`, the compiler would catch it and raise an error.

Given the importance of limiting side effects as much as possible, it's a good practice of using `const` everywhere by default, and only removing it where we are aware that we need a mutable object.

### References
- https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md
