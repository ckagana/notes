### Use precise types
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
[2]

### Use literal suffixes wherever possible
With C++11 comes the possibility of defining custom literals' suffixes: these can be useful to add type information to literal values that may instead be too generic:

```c++
change_speed(2.3);        // error: no unit
change_speed(23m / 10s);  // meters per second
change_speed(2.3m_s);
```
[3]

### Use `const` wherever possible
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

Given the importance of limiting side effects as much as possible, it's a good practice of using `const` everywhere by default, and only removing it where we are aware that we need a mutable object. [2]

### Use range-based `for` loops or `for_each`
The classical low-level `for` and `while` loops should be avoided wherever possible, since they can't express any intent directly in code, and they expose too many unnecessary implementation details:

```c++
int i = 0;
while (i < v.size()) {
    // ...do something with v[i]
}
```

Here it's not immediately clear that we just want to loop over an array. Besides that, the tool we are using is way too powerful for our needs, since it allows us to do much more than simply looping over the array. Finally, that `i` variable, which is an implementation detail of our loop, is visible and available also to the code that will come after, and this may be dangerous, other than unnecessary.

The scope-based `for` loop directly expresses the idea of just looping over a container:

```c++
for (const auto& x : v) {
    // do something with x
}
```

and also allows us to use `const` references to the elements of the container, while with a regular `for` or `while` loop instead, we would always have full write access. [4]

If you want more control over the range over which to loop, you can use `std::for_each`, which takes two iterators and a callback, and applies the callback to all values of the range defined by the two iterators:

```c++
std::vector<int> nums{3, 4, 2, 8, 15, 267};
std::for_each(nums.begin(), nums.end(), [](int &n){ n++; });
```
This also allows you to use named algorithms as callbacks, gaining in readability. [5]

### Use library functions
Avoid reinventing the wheel when it comes to simple algorithms, for example:

```c++
void f(vector<string>& v)
{
    string val;
    cin >> val;
    // ...
    int index = -1;
    for (int i = 0; i < v.size(); ++i) {
        if (v[i] == val) {
            index = i;
            break;
        }
    }

    // ...
}
```

You can do the same just using library function `std::find`:

```c++
void f(vector<string>& v)
{
    string val;
    cin >> val;
    // ...
    auto p = find(begin(v), end(v), val);
    // ...
}
```
[2,6]

### Always consider objects' ownership
The biggest problem we face when it comes to memory management is forgetting to free memory, or to release resources (like leaving file connection opened). Actually, the two cases boil down to the same problem of releasing resources, since memory is a resource, and freeing memory means to release it. C++ has a standard pattern to tackle this problem, which is *Resource Acquisition is Initialization (RAII)*: to apply this pattern means that every time we have to use an external resource (even only memory, i.e. when using pointers), we have to wrap it in an object so that the resource is acquired in the constructor, and released in the destructor. Since in C++ there's no garbage collector, we are always sure of the exact time when an object will be destroyed, and this always happens when intended, even if exceptions are thrown in the meanwhile.

When you create an object that wraps a resource in such a way, you say that that object *has ownership* on that resource, because it, on no-one else, has the responsibility to release it. It's important to highlight that here we are talking about real objects, instances, and not classes: this means that if an object, a specific instance, owns a resources, and then this object is copied, we end up with the same resource (think of a file handle) being owned by two different objects. This is very dangerous, because it may lead to duplicated resource releases (like trying to free two time the same pointer).

## References
1. "Effective Modern C++", Scott Meyers, 2015
2. ["C++ Core Guidelines"](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md), Bjarne Stroustrup, Herb Sutter, 2016
3. ["User-defined literals"](http://en.cppreference.com/w/cpp/language/user_literal)
4. ["Range-based for loop"](http://en.cppreference.com/w/cpp/language/range-for)
5. ["std::for_each"](http://en.cppreference.com/w/cpp/algorithm/for_each)
6. ["C++ Core Guidelines: The Standard Library"](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#S-stdlib)
