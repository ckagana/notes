# Objects

- [Use initialization lists instead of initialization in constructors](#use-initialization-lists-instead-of-initialization-in-constructors)
- [Always consider objects' ownership](#always-consider-objects-ownership)


## Use initialization lists instead of initialization in constructors

Consider having to initialize some class' properties at construction:

```c++
class MyClass
{
    MyClass(SomeType a, OtherType b)
    {
        this->a = a;
        this->b = b;
    }

    SomeType a;
    OtherType b;
};
```

Here we are injecting objects `a` and `b` inside the two properties with the same name of `MyClass`. What's actually going on here is the following:
- Default constructors of a and b (thus of `SomeType` and `OtherType`) are called to initialize properties `a` and `b`;
- `MyClass` constructor is called.

Now, we know that we are never going to need the default values variables `a` and `b` are automatically initialized with, so why wasting time and memory doing that useless default initialization? To void doing that, we use *initialization lists*:

```c++
class MyClass
{
  MyClass(SomeType a, OtherType b): a\(a), b(b);
  SomeType a;
  OtherType b;
};
```

Initialization lists allow us to hook into the automatic initialization process, and choose which constructor to use. In this case, for example, we are stating that properties `a` and `b` shouldn't be initialized calling the default constructors of `SomeType` and `OtherType`, but rather their copy constructors, with the given objects as arguments. Actually, the difference with the previous case is that we are skipping `MyClass` constructor call, rather than properties' initialization.

### References
- http://www.cprogramming.com/tutorial/initialization-lists-c++.html


## Always consider objects' ownership

The biggest problem we face when it comes to memory management is forgetting to free memory, or to release resources (like leaving file connection opened). Actually, the two cases boil down to the same problem of releasing resources, since memory is a resource, and freeing memory means to release it. C++ has a standard pattern to tackle this problem, which is *Resource Acquisition is Initialization (RAII)*: to apply this pattern means that every time we have to use an external resource (even only memory, i.e. when using pointers), we have to wrap it in an object so that the resource is acquired in the constructor, and released in the destructor. Since in C++ there's no garbage collector, we are always sure of the exact time when an object will be destroyed, and this always happens when intended, even if exceptions are thrown in the meanwhile.

When you create an object that wraps a resource in such a way, you say that that object *has ownership* on that resource, because it, on no-one else, has the responsibility to release it. It's important to highlight that here we are talking about real objects, instances, and not classes: this means that if an object, a specific instance, owns a resources, and then this object is copied, we end up with the same resource (think of a file handle) being owned by two different objects. This is very dangerous, because it may lead to duplicated resource releases (like trying to free two time the same pointer).
