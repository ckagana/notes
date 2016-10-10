# Best practices

- [Use library functions](#use-library-functions)


## Use library functions

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

### References
- https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md
- https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#S-stdlib
