# Syntax

- [Use literal suffixes wherever possible](#use-literal-suffixes-wherever-possible)
- [Use range-based `for` loops or `for_each`](#use-range-based-for-loops-or-for_each)


## Use literal suffixes wherever possible

With C++11 comes the possibility of defining custom literals' suffixes: these can be useful to add type information to literal values that may instead be too generic:

```c++
change_speed(2.3);        // error: no unit
change_speed(23m / 10s);  // meters per second
change_speed(2.3m_s);
```

### References
- http://en.cppreference.com/w/cpp/language/user_literal


## Use range-based `for` loops or `for_each`

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

and also allows us to use `const` references to the elements of the container, while with a regular `for` or `while` loop instead, we would always have full write access.

If you want more control over the range over which to loop, you can use `std::for_each`, which takes two iterators and a callback, and applies the callback to all values of the range defined by the two iterators:

```c++
std::vector<int> nums{3, 4, 2, 8, 15, 267};
std::for_each(nums.begin(), nums.end(), [](int &n){ n++; });
```
This also allows you to use named algorithms as callbacks, gaining in readability.

### References
- http://en.cppreference.com/w/cpp/language/range-for
- http://en.cppreference.com/w/cpp/algorithm/for_each
