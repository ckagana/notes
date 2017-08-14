# Namespaces

- [Always use fully qualified names in header files](#always-use-fully-qualified-names-in-header-files)
- [Use unnamed namespaces instead of global `static`](#use-unnamed-namespaces-instead-of-global-static)
- [Use `using` instead of `typedef`](#use-using-instead-of-typedef)

## Always use fully qualified names in header files

### `using` directives
`using` *directives* allow to make all names declared inside a namespaces visible from the current namespace (the namespace the directive is placed in):

```c++
#import "MyLibrary.h"

namespace MyProject {
    using namespace MyLibrary;
    // ...
}
```

It's almost always a bad idea to use `using` directives, since namespaces are meant to avoid polluting the single global pool of names, that can be used to create identifiers, in the first place. Using namespaces allow to reuse the same name multiple times for different identifiers, without creating ambiguities and compiler errors due to name conflicts. It's clear, then, that importing all names defined in another namespace goes directly against this goal, because suddenly all those names (also the ones we don't need, but we are still importing) cannot be used any more in our code. This is even worse if we are doing this inside a header file, because this file may be included in other files, all of which won't be able to use those names any more.

### `using` declarations
`using` *declarations*, instead, are used to make only specific names belonging to another namespace, visible to the current one, or to create an alias for an existing name:

```c++
#import "MyLibrary.h"

namespace MyProject {
    using MyLibrary::MyClass;
    // ...
}
```

`using` declarations are usually preferred over `using` directives to import names into a namespace, because they give the least chance of namespace polluting. However, it's still not advisable to use them inside header files, because `using` declarations can be used to shadow names imported by other header files.

### `using` declarations make header include order matter
Let's consider this example, where we define a new class `MyProject::vector`. Since we are inside our custom namespace, we can use the name `vector` without fear that it could conflict with the more famous `std::vector`:

```c++
// base.h
#ifndef BASE_H
#define BASE_H

namespace MyProject
{
    class vector {};
}

#endif
```

In the next header, we include `base.h` and do something with our previously defined `vector` type:

```c++
// z.h
#ifndef Z_H
#define Z_H

#include "base.h"

namespace MyProject
{
    void useVector()
    {
        const vector v;
    }
}

#endif
```

Now, let's consider that there's also another header file, doing funny things with the `using` declaration:

```c++
// x.h
#ifndef X_H
#define X_H

#include <vector>

namespace MyProject
{
    using std::vector;
}

#endif
```

Here we are importing the `vector` name as defined in the `std` namespace, inside our `MyProject` namespace. What happens if we include both headers inside an implementation file?

```c++
// main.cpp
#include "x.h"
#include "z.h"

int main()
{
    MyProject::useVector();
}
```

What happens is that this code causes a compilation error, saying that `vector` needs a template argument to be specified. This is because the `x.h` header file is re-defining the `vector` name inside `MyProject`, specifying that this is an alias for `std::vector`, instead of our custom `vector` class. When `z.h` is then included, the `vector` name doesn't represent our custom class anymore, but the standard `vector` template container, and this is why the compiler complains.

If we swap the order with which the two headers are included, thus including `z.h` before `x.h`, `useVector()` won't create any trouble, because when it was defined `vector` still was our custom class, but if we were using the `vector` name after `x.h` inclusion, it would have represented `std::vector` instead. This is particularly bad, because it means that the behavior of `z.h` depends on the order with which it's included, while header files should always be self-contained, thus independent from how they are used.

It's interesting to note that a `using` directive (like `using namespace std;`) wouldn't have caused any compilation error, because, unlike `using` declarations, it doesn't shadow names already existing in the current namespace.

When we are using `using` declarations inside an implementation file, instead, this problem should never occur, because implementation files aren't meant to be included by other files, so if we add `using std::vector;`, we are changing the meaning of `vector` only in the current scope of the current file, and no unexpected side-effects can happen when our file is used.

### `using` declarations versus aliases
If we pay attention to the previous example, we'll notice that the actual error being thrown by the compiler isn't related to the same name being declared two times inside the same namespace, but to us trying to use `vector` without passing a template argument: this means that the compiler was really ok with our new declaration of the `vector` name. Usually, however, if we try to use `using` to redeclare an already existing name, we will get an error:

```c++
namespace MyProject {
    class string {};
}

namespace Other {
    class string {};
}

namespace MyProject {
    using Other::string;
}

namespace MyProject {
    void useString() {
        const string s;
    }
}
```

Here, the line `using Other::string` will cause a compilation error, since we are trying to declare the name `string`, which is already declared. However, if we do just a little change:

```c++
namespace MyProject {
    class string {};
}

namespace Other {
    class something {};
    using string = something;
}

namespace MyProject {
    using Other::string;
}

namespace MyProject {
    void useString() {
        const string s;
    }
}
```

At the time of writing, this code produces different effects according to which compiler is used: g++ 4.9 accepts the code, while Clang 3.8 raise a compilation error at the line `const string s` since `string` is ambiguous there. The difference with the previous code is that here the line `using string = something` is not technically a *declaration*, but an *alias*, and the C++ language definition forbids to declare a name which is already declared, but not to make an alias out of an already existing name. This, anyway, shows that also alias in header files can be dangerous, for the same reason while `using` declarations are.

### Summing up
The rule of thumb we can get from this is:
- never use `using` directives;
- never use `using` declarations inside header files;
- always use fully qualified identifiers (starting with the global namespace reference `::`), in header files;
- always use `using` declarations inside implementation files only, and *after* all header files have been included.

### References
- https://stackoverflow.com/questions/39799873/
- http://stackoverflow.com/questions/6175705/
- http://www.gotw.ca/gotw/053.htm


## Use unnamed namespaces instead of global `static`

In the C programming language is possible to define `static` global variables and functions, meaning that those will be available only in the current translation unit (the file plus all its includes). Now in C++ this usage of `static` has been deprecated because it causes confusion, since `static` usually has different meanings; however, it's possible to achieve the same effect using unnamed namespaces:

```c++
// file a.cpp
#include "SomeLib.h";

namespace {
    bool a = false;
    class MyClass {
        // ...
    };
}

void SomeClass::someMethod()
{
    // I can use a only within the current translation unit.
    someFunction(a);

    // I can use MyClass only within the current translation unit.
    MyClass object;
}
```

Unlike the old `static`, unnamed namespaces also allow to define types which are local to the current translation unit, in addition to variables and functions.

### References
- http://stackoverflow.com/questions/357404/


## Use `using` instead of `typedef`

As of C++ 11, defining aliases with `typedef` or `using` is almost the same, with the exception that `using` allows to define alias of templates.

### References
- http://en.cppreference.com/w/cpp/language/type_alias
