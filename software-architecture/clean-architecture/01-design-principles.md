# Design Principles

## The Single Responsibility Principle

The meaning of the Single Responsibility Principle is not, as often quoted, that a single programming element (function, class, module, etc.) should do only one thing. This is actually another principle, popularized by UNIX, that a single function should do one thing, and do it well, but the SRP says something quite different.

Software systems are recognized to be subject to change in time, not only because of errors needing to be fixed, but mostly because changes are requested regarding their features. The design of software systems should strive to accommodate future changes, to bring the expense caused by those changes to a minimum. For this reason, it's important to understand where do changes come from.

Changes to software systems originate from whichever party decides what the requisites of the systems are: usually they are users or stakeholders. After a first version of the software is delivered, stakeholders may ask for additional features, or for changes to existing ones.

When a change needs to be implemented, following stakeholders requests, our interest as developers is to keep to a minimum the number of modules that need to change because of that request. If every time a minor change requires fixing, releasing and deploying dozens of modules, we have a problem.

From this acknowledgment, the Single Responsibility Principle tells us to keep separated modules that change for different reasons, and at different rates, which means keeping separated modules belonging to different stakeholders. If a module depends instead on different stakeholders, it doesn't matter if many of these stakeholders rarely require any change: it suffices that a single stakeholder requires changes frequently, to force us to make new releases and deployments of the entire module.

In this context, we are actually referring to group of stakeholders having the same intent, or purposes, when it comes to using the system. Thus, we might call these groups of similar stakeholders, "actors", and say that a module should be responsible to one actor only.

Let's think of a payroll application, featuring the methods `calculatePay()`, `reportHours()` and `save()`. This application will be responsible to three actors:
- the CFO is interested in getting the amount of money to pay employees, so he's concerned with `calculatePay()`
- the COO (human resources) has control over the amount of hours worked by employees, so it may need to require changes to `reportHours()`
- the DBAs are in charge of specifying how the `save()` method works

If these three methods were put in the same class, some actors could be affected by the actions of another actor. For example, let's say that `calculatePay()` and `reportHours()` share the same algorithm for calculating non-overtime hours: `regularHours()`. Now, if the CFO requires this algorithm to be tweaked for its own purposes, and the change is made to the common `regularHours()` function, suddenly the `reportHours()` method used by human resources starts to contain incorrect hours, without the COO realizing that a change has been made. The problem in this case is that code that different actors depend upon is not properly decoupled (in this example it's actually even shared). 

If the same class is responsible for different actors, there's also a higher chance that two independent groups of developers will be requested to apply changes to the same codebase at the same time, which will produce merge conflicts, that are not always safe to resolve.

To fix this problem we need to decouple the code belonging to different actors, that is, code that has different reasons to change. For example, we could create `PayCalculator`, `HourReporter` and `EmployeeSaver` classes, all depending on `EmployeeData`. To avoid the inconvenience of having three different classes to deal with, we can put a `EmployeeFacade` in front of them, that just instantiates and delegates to the other classes.

Either way, the important point is that each of these classes must be responsible to only one actor, that is, must have only one reason to change, which is what the Single Responsibility Principle states.


## The Open-Closed Principle

The Open-Closed Principle states that a software artifact should be easy to extend, without having to modify it. This, again, takes care of the change that all software constantly risk of undergoing. If a change to an existing feature is requested, it's of course better if we can implement it just adding new code, without changing existing one.

The first step to achieve this result, is protecting software components from changes that occur on one another. If component A depends on component B, and we change component B, then also component A will need to be changed, and this for two reasons:
- if we changed the interface of component B, then we need to change the places of component A using component B, to conform to the new interface
- if component B only changed its implementation, we still need to re-release and re-deploy component B; however, since A depends on B, we'll also need to update A so that it points to the new version of B, which means that we need to re-release and re-deploy A even if no code has been changed there

In general, when component A depends on component B, we say that component B is protected from changes to component A, because A knows about B, but B knows nothing about A, and as such it's untouched by changes to it.

The rationale we use to choose which component should depend on which other, is based on how likely a component will change. This, however, might be in contrast with the direction of the data flow.

For example, an user interface component is more likely to change than a domain logic one, so the user interface should depend on the domain logic. Luckily, this is also the direction of the data flow, because data is submit from the user in the user interface component, and then is sent to the domain logic: in this case the direction of the dependency matches the direction of the data.

Instead, the domain logic component still changes less frequently than the database component, so we want the database to depend on the domain logic: however, in this case the data flows in the opposite direction, because it's the domain logic that decides when it's time to write to the database. To fix this, we have to *invert the dependency* of the domain logic to the database (check the Dependency Inversion Principle), so that it's the database that depends on the domain logic instead. To do this, we define a data gateway interface in the domain logic, that knows nothing about the specific implementation used to write data to storage systems, and then at startup we inject in the domain logic a specific implementation of that interface, that knows how to deal with the database: this way, the domain logic will pass the data into the data gateway interface, but being independent from the actual database system used.

Components that are more likely to change, are so because they're more concrete, meaning that they carry a lot of implementation details: these are called lower-level components. On the other hand, components that are less likely to change, are so because they're more abstract: these are called higher-level components. Thus, to minimize change propagation, lower-level components must depend on higher-level ones, and higher-level components must never depend on lower-level ones.

Although lower-level components can depend on higher-level ones, meaning that we allow changes to abstract components to cause changes to concrete ones, we can still limit the propagation of these changes. A general rule states that software entities should not depend on things that they don't need, and this calls for removing transitive dependencies. If a concrete component B depends on an abstract component A, which itself depends on other more abstract component C, then changes to C will transitively propagate to B, requiring it to change as well, even though B doesn't really know of the existence of C. To prevent this from happening, we can add an abstraction layer between B and A, making it so that B depends on an interface A', implemented by A, instead of directly on A. This way, when C changes, and consequently A also changes, B is not interested by any change, as long as this change doesn't involve the interface A'. Only when a significant change to A, which means a change to its interface A', is requested, then B will need to change as well.

With the first step, we limited as much as possible the propagation of changes among components. However, we still didn't achieve the result stated by the Open-Closed Principle, that new code is added instead of old code being changed. This can be achieved in different ways according to the scenario:
- if the current situation is that concrete component B uses abstract component A, and we need to have a different behavior at the level of component B, instead of changing B we can just replace B with another component C, implementing the new desired behavior: since A didn't know about B in the first place, no code needs to be changed
- if again B uses A, but this time A needs to be changed, if the dependencies from B to A was going through the intermediate interface A', we can just replace A with C, still implementing A', and B won't see any difference, while we still didn't make any code change (just writing new code)
- the same thing can be done for the reversed dependencies, due to the fact that interfaces are involved


## The Liskov Substitution Principle

The Liskov Substitution Principle states that wherever an object of type T is requested, any object of type S that is subtype of T can be used there instead.

To see how this principle can be violated, let's consider a `Rectangle` class, that has `setH()` and `setW()` methods. Imagine, then, that we create a subclass of `Rectangle`, called `Square`, with the additional method `setSide()`. Even though `Square` is a subclass of `Rectangle` as far as object-oriented languages are concerned, `Square` is not a proper *subtype* of `Rectangle`. Wherever a `Rectangle` is used, the user will think of being able to change the width and the height independently, for example with code like this:
```java
Rectangle r = getRectangle();
r.setW(5);
r.setH(2);
assert(r.area() == 10);
```

Now imagine if `getRectangle()` returned a `Square` instead: this wouldn't violate any type check, because `Square` is still a subclass of `Rectangle`, but suddenly `r.area()` will return `4`, because the last `r.setH(2)` call will have set the side of the square to `2`, making the assert fail. The only way to fix this is adding code to `Rectangle` to check if the current implementation is actually a `Square`, making the more abstract class know about the existence of the more concrete one, and in general making those two types not substitutable.

The problem here is that the Liskov Substitution Principle has little to do with programming languages constructs such as classes, and more with the abstract concept of software interfaces. In the previous example, the interface we were adhering to was that the `Rectangle` type had a width and a height, that should be independently changeable. The `Square` type, instead, is defined so that width and height are always the same, and as such it's not a subtype of `Rectangle`, even if the programming language allows us to use an instance of `Square` in place of an instance of `Rectangle`.

To further highlight the fact that this isn't about programming language constructs, let's just think about the fact that types and interfaces can be implemented with REST endpoints. If my software system gets weather information from a service exposing a REST API such as `temp.service.com/temperature/?city=SomeCity`, then the interface of this "weather service" type is that the `temperature` endpoint should be used with the `city` parameter. If in the future a better service comes up, which still has the `temperature` endpoint, but taking `place` as parameter, instead of `city`, I will have to change my code to accommodate this difference: the two types aren't substitutable, and as such they violate the Liskov Substitution Principle. Had the new service been a subtype of the old one (for example adding new endpoints and parameters, but keeping that `/temperature/?city`), I could've used the new one in place of the old.


## The Interface Segregation Principle

The Interface Segregation Principle also tackles the need to prevent unnecessary coupling between components. Let's assume we have a class:
```
OPS
---
+ op1
+ op2
+ op3
```

and three different users of this class, `User1` who only uses `op1`, `User2` who only uses `op2`, and `User3` who only uses `op3`. In this situation, the source code of `User1` only needs `op1`, but since `op1` is part of `OPS`, `User1` might end up to unnecessarily depend also on `op2` and `op3`.

In fact, if using statically typed compiled languages, making changes to `OPS`, to keep up with requirements coming from another class like `User2`, will cause also `User1` to need to be recompiled, for these reasons:
- after changing `OPS`, the compiler has to make sure that `User1` is still properly using the interface of `OPS`, which might have changed
- in languages like C++, changing `OPS` might require the compiler to change the technical layout of the class
This problem is usually mitigated when using dynamically typed languages, though.*[[ref](https://stackoverflow.com/questions/53575779)]

To fix this situation, we need to segregate the interfaces that are responsible to different actors: `OPS` now will implement three different interfaces, `U1Ops` providing `op1`, `U2Ops` providing `op2` and `U3Ops` providing `op3`. Then, `User1` will depend just on `U1Ops`, instead of on the whole `OPS`, `User2` will depend on `U2Ops`, and `User3` will depend on `U3Ops`. This way, if `op2` is changed, `User1` won't notice, and won't need to be recompiled and redeployed.


## The Dependency Inversion Principle

Like previewed when talking about the Open-Closed Principle, the Dependency Inversion Principle states that dependency should be designed so that the more concrete component depends on the more abstract one, and never the other way around. In statically-typed languages we could use the heuristic that source code files should only contain `import` statements referring to interfaces or abstract classes (or other kinds of abstract definition), or to concrete, but extremely stable, classes (like `String`). 

The difference between concrete types and abstract types lies in their stability. Concrete types, representing technological details, are by their very nature volatile, unstable, meaning that they can change quite frequently; on the other hand, abstract types are less bound to details, and as such less likely to change, thus more stable.

To implement the Dependency Inversion Principle, then, we should rely on these coding practices:
- *Don't refer to volatile concrete classes*, but to abstract interfaces instead, for example using Abstract Factories
- *Don't derive from volatile concrete classes*, for the same reason as the rule above, and also because inheritance is the strongest type of dependency
- *Don't override concrete functions*, because concrete functions carry dependencies on concrete elements, which are inherited by the overriding function
- *Never mention the name of anything concrete and volatile*, as a general summary of the principle
