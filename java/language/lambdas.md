# Lambdas

## Lambdas, overloading and generics

The fact that type inference is used with the arguments of a lambda function poses some problems when we want to pass lambdas to overloaded methods using generic parameters. A lambda's expression type is inferred from the target type, thus in this example:
```java
interface A
{
    void call(String x);
}

abstract class B
{
    void call(int x);
}

class C
{
    public void apply(A a)
    {
        ...
    }

    public B apply (B b)
    {
        ...
    }
}

c.apply(x -> System.out.println(x));
```

in this case it's obvious that the `apply(A a)` overloading will be chosen, because `A` is a functional interface, while `B` is not. At this point we have the target type `A` for the lambda, and the type inference for `x` can proceed, resulting in `x` being considered a `String`. Now consider this example:
```java
class C
{
    public void apply(A a)
    {
        ...
    }

    public <T extends B> T apply(T t)
    {
        ...
    }
}
```

here, since the second overloading has generic types, the overload resolution can be done only after the concrete type of `T` will be determined, but for this to happen, we need first to determine the type of the lambda that is being passed, which we can't know until we have resolved the overloading! We could argue that in this specific case, since `T extends B` it's obvious that it can't possibly be a functional interface, but nevertheless the Java specification decided to refuse this kind of code anyway, because it would be too complicated to handle.

It's still possible to resolve the ambiguity, though, manually specifying the type of the lambda, during the call:
```java
c.apply((A)x -> System.out.println(x));
```

this way there will be no need to wait until we discover the type of the lambda before resolving the overloading, because we already know it.


#### References
- https://stackoverflow.com/questions/21905169/
- https://stackoverflow.com/questions/28231155/
