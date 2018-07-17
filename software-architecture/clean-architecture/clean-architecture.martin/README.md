# Clean Architecture

The following are notes taken while studying the book [Clean Architecture, A Craftman's Guide to Software Structure and Design, by R.C. Martin, 2018](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164).

- [Design Principles](#design-principles)
    - [The Single Responsibility Principle](#the-single-responsibility-principle)
    - [The Open-Closed Principle](#the-open-closed-principle)
    - [The Liskov Substitution Principle](#the-liskov-substitution-principle)
    - [The Interface Segregation Principle](#the-interface-segregation-principle)
    - [The Dependency Inversion Principle](#the-dependency-inversion-principle)
- [Component Principles](#component-principles)
    - [Component Cohesion](#component-cohesion)
        - [The Reuse/Release Equivalence Principle](#the-reuse-release-equivalence-principle)
        - [The Common Closure Principle](#the-common-closure-principle)
        - [The Common Reuse Principle](#the-common-reuse-principle)
    - [Tension of Components](#tension-of-components)
    - [Component Coupling](#component-coupling)
        - [The Acyclic Dependencies Principle](#the-acyclic-dependencies-principle)
        - [The Stable Dependencies Principle](#the-stable-dependencies-principle)
        - [The Stable Abstractions Principle](#the-stable-abstractions-principle)
- [Architecture](#architecture)
    - [Decoupling into layers and use cases](#decoupling-into-layers-and-use-cases)
    - [Decoupling modes](#decoupling-modes)
    - [Boundaries](#boundaries)
    - [The Screaming Architecture](#the-screaming-architecture)
    - [The Clean Architecture](#the-clean-architecture)

## Design Principles

### The Single Responsibility Principle

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


### The Open-Closed Principle

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


### The Liskov Substitution Principle

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


### The Interface Segregation Principle

The Interface Segregation Principle also tackles the need to prevent unnecessary coupling between components. Let's assume we have a class:
```
OPS
---
+ op1
+ op2
+ op3
```

and three different users of this class, `User1` who only uses `op1`, `User2` who only uses `op2`, and `User3` who only uses `op3`. In this situation, the source code of `User1` only needs `op1`, but since `op1` is part of `OPS`, `User1` ends up unnecessarily depending also on `op2` and `op3`, so that if these methods will have to change, `User1` will need to change too (or at least to be recompiled and redeployed).

To fix this situation, we need to segregate the interfaces that are responsible to different actors: `OPS` now will implement three different interfaces, `U1Ops` providing `op1`, `U2Ops` providing `op2` and `U3Ops` providing `op3`. Then, `User1` will depend just on `U1Ops`, instead of on the whole `OPS`, `User2` will depend on `U2Ops`, and `User3` will depend on `U3Ops`. This way, if `op2` is changed, `User1` won't notice, and won't need to be recompiled and redeployed.


### The Dependency Inversion Principle

Like previewed when talking about the Open-Closed Principle, the Dependency Inversion Principle states that dependency should be designed so that the more concrete component depends on the more abstract one, and never the other way around. In statically-typed languages we could use the heuristic that source code files should only contain `import` statements referring to interfaces or abstract classes (or other kinds of abstract definition), or to concrete, but extremely stable, classes (like `String`). 

The difference between concrete types and abstract types lies in their stability. Concrete types, representing technological details, are by their very nature volatile, unstable, meaning that they can change quite frequently; on the other hand, abstract types are less bound to details, and as such less likely to change, thus more stable.

To implement the Dependency Inversion Principle, then, we should rely on these coding practices:
- *Don't refer to volatile concrete classes*, but to abstract interfaces instead, for example using Abstract Factories
- *Don't derive from volatile concrete classes*, for the same reason as the rule above, and also because inheritance is the strongest type of dependency
- *Don't override concrete functions*, because concrete functions carry dependencies on concrete elements, which are inherited by the overriding function
- *Never mention the name of anything concrete and volatile*, as a general summary of the principle


## Component Principles

### Component Cohesion

#### The Reuse-Release Equivalence Principle

The Reuse/Release Equivalence Principle states that to be reused, software components must be tracked through a release process. This is needed first because this way users will know if and when components are changed, and what are the possible compatibility issues.

The problem with releases is that we need to create releasable artifacts containing software elements that are somewhat cohesive: the worst case scenario is when different components must be released at the same time because elements that change together are scattered among different components; in this case elements that are releasable together are not part of the same releasable artifact.

#### The Common Closure Principle

The Common Closure Principle is the equivalent of the Single Responsibility Principle, applied to components instead of low-level elements. When an application has to change, it's of course better if all changes occur in only one component, instead of being scattered through multiple components: this way we need to re-compile and re-deploy only one component.

#### The Common Reuse Principle

If the Common Closure Principle states that classes that tend to change for the same reason should be put in the same component, the Common Reuse Principle states that classes that tend to be reused together should belong to the same component. Typically, classes that are reused together are dependent on each other, so it makes sense that they are left together.

A typical violation of this principle is when a class `A` depends on another class `B`, which is included in a component `C`. When `B` is changed, typically also `A` needs to be changed, or at least re-deployed. However, if another class `D` of `C`, that is not cohesive with `B`, and as such is not needed by `A`, is changed, the component `C` still needs to be re-deployed, and thus `A` needs to be re-deployed as well, because it depends on that component. The end result is that `A` would need to be re-deployed because of completely irrelevant changes made to `C`.

Thus, we can see the Common Reuse Principle as a generalization of the Interface Segregation Principle, in the sense that it states that we shouldn't depend on things that we don't need, and to achieve this we should put into a component only classes that are strictly bound to each other, so that if we depend on one of them, obviously we depend also on every other one, and it will never happen that we need to re-deploy because of changes to unrelated classes in the component.


### Tension of Components

The previous principles apply different, and opposite forces to the process of components design. The Reuse/Release Equivalence Principle and the Common Closure Principle tend to put more stuff into components, making them larger, to avoid doing unnecessary releases, and to avoid doing unnecessary deployments; the Common Reuse Principle, instead, tend to take stuff away from components, making them smaller, to avoid unnecessary dependencies.

It's clear that it's not possible to satisfy all these principles at the same time, and that some trade-off needs to be done when designing the components of an architecture. If we focus mostly on maximizing reuse and minimizing dependencies, we end up with a system hard to maintain, because requires too many components changes and re-deploys; if we focus on reuse and maintenance, we end up with a system with too many dependencies, and thus releases; if we focus on maintenance and dependencies, we end up with a system which is hard to reuse.

Generally, at the beginning of a project, reusability is less important than maintainability, so we should focus more on the Common Closure Principle, and on the Common Reuse Principle; as the project matures, we will be able to put more effort on reusability, mostly because cohesive components will have emerged in the meanwhile.


### Component Coupling

#### The Acyclic Dependencies Principle

We can draw a diagram of the components structure of an application, where components are linked by arrows showing dependency relationships, so that a component that depend on another has an arrow going from itself to its dependency. For example, the `Main` component depends on many other components, but no component depend on it: thus, `Main` would have many arrows going to other components in the diagram, and no arrow coming to it.

In a typical application structure, we'd have `Interactors` depending on `Entities`, `Database` depending on `Interactors` and `Entities`, `Authorizer` depending on `Interactors`, `Presenters` and `Controllers` depending on `Interactors`, and `Views` depending on `Presenters`; furthermore, `Main` depends on `Presenters`, `View`, `Interactors`, `Controllers`, `Database` and `Authorizer`. Let's say the team responsible for `Presenters` makes a new release: looking at the diagram, it's very easy to see which other teams are affected by this release, that is `Main` and `View`, meaning that the developers of these two components will have to decide if they want to update their components with the new version of `Presenters`, or if they want to keep using the old one. As another example, no component depends on `Main`, and thus we can make any change to `Main` without worrying of affecting any other team.

It's important that the components structure diagram be a *directed acyclic graph*, meaning that is has no cycles in it: it's impossible to travel from one component, following the dependency relationships, and ending up in the same starting component. For example, let's say that `Entities` is changed so that it now depends on `Authorizer`: suddenly, a dependency cycle is formed, because `Interactors` depends on `Entities`, which depends on `Authorizer`, which depend on `Interactors` itself. This way, `Interactors`, `Entities` and `Authorizer` have become a single, big, component, where all teams must always use the same versions to avoid merge conflicts, unit testing components depending on this cycle is very difficult, and there is no correct order to build these components, because they depend on each other.

The Acyclic Dependencies Principle states that there should be no cycles in the application's dependency structure. To break cycles, we could apply the Dependency Inversion Principle, for example making `Entities` depend on an interface that is implemented by `Authorizer` instead; or we might create a new component that both `Entities` and `Authorizer` depend on (which is the same as the previous solution, only at a higher architectural level).

#### The Stable Dependencies Principle

Components can be *stable*, meaning unlikely to change, or *volatile*, meaning likely to change. A volatile component, that has been designed to be easy to change, should never be a dependency of a stable component, that is difficult to change, otherwise the volatile component will also be difficult to change.

A software component will be hard to change, or stable, if a lot of other components depend on it: in fact in this case even the slightest change to that component will require several other components to be updated; every incoming dependency is a reason for it not to change, a responsibility.

A software component will be easy to change, or unstable, if few other components (maybe no one) depend on it: the component has little responsibility, thus it can change more freely.

We can calculate the Instability *I* of a component as *Fan-out*/(*Fan-in* + *Fan-out*) where *Fan-out* is the number of outgoing dependencies, and *Fan-in* is the number of incoming dependencies. The Instability *I* ranges from 0, when the component is independent (it only has responsibilities, thus it maximally hard to change), to 1, when the component is irresponsible (it can change with maximum freedom).

For example, let's say we have the following components:
```
Ca [q, r]
Cb [s]
Cc [t, u]
Cd [v]
```

with the following class dependencies:
```
q -> t
r -> u
s -> u
u -> v
```

We want to calculate the stability of the component `Cc`. There are three dependencies incoming into `Cc`: `q` depending on `t`, `r` depending on `u`, and `s`depending on `u` as well. Then, there is only one dependency outgoing from `Cc`, which is `u` depending on `v`. Thus, `I` equals to 1/(3 + 1) = 0.25.

The Stable Dependencies Principle states that the Instability of a component should be larger than the Instability of the components it depends on: in other words, *I* should *decrease* in the direction of dependencies. If a `Stable` component depends on a `Flexible` component, it means that *I* is increasing following the dependency direction: since `Flexible` is changed more often than `Stable`, and `Stable` must be changed every time `Flexible` is changed, this means `Stable` less stable than intended; from another point of view, `Flexible` will be hard to change (as opposed as how it's intended to be) because changing `Flexible` would require changing `Stable` as well, which is not supposed to change that much.

#### The Stable Abstractions Principle

High level software policies should not change in response to change to low-level details, like the user interface. This means that high level policies should be put into the most stable components of the system, even maximally stable ones with *I* equals to 0. However, even though it's good for these components to be stable, as far as dependencies are concerned, they should still be able to change, for example because some business logic has to change. To be able to change the logic without changing the component, we make use of the Open-Closed Principle, where instead of changing the high level classes, we extend them: for this to work, then, we need these high-level policy classes to be abstract.

The Stable Abstract Principle states that stable components should be abstract, so that their stability doesn't prevent them from being extended, and that an unstable component should be concrete, because its instability allows the concrete code to be easily changed.

We can define Abstractness *A* as the ratio between the number of abstract classes and interfaces in the component *Na*, and the total number of classes in the component *Nc*, so that *A* = *Na*/*Nc*. We can then plot each component on a *I*, *A* graph, where the coordinates of each component are their measures of *I* and *A*.

Components falling near the origin (0, 0) are not desirable, because they are at the same time stable and concrete, thus very painful to change: however, some components inevitably fall in that area, like database schemas (which are notoriously problematic to change), or utility library classes, like `String`, which is not going to change anyway.

Components falling near (1, 1) are said to be useless, because they are very abstract, but few or no other classes depend on them: they might appear after a refactoring where some abstract class or interface is left unused in the system.

The ideal area where components should fall is the one around the segment connecting (0, 1) to (1, 0), called the *Main Sequence*, where components are either abstract and stable, or volatile and concrete, as the Stable Abstractions Principle requires. We can measure the distance of a component from the Main Sequence, as *D* = |*A* + *I* - 1|. Components with a *D* of 0 fall directly on the Main Sequence, while those with a *D* of 1 are as far away as possible from it. A good architectural design has most components with small values of *D*.


## Architecture

A good architecture is judged from several key indicators. First of all a good architecture should take care of *development* issues. The system should be easy enough for developers to work with: a small team of developers can more easily work on a monolith system without well defined components or interfaces; on the other hand, a system being developed by several teams concurrently will need a proper separation in well-defined components with stable interfaces.

The second element architectures should take into account is *deployment*: the goal to aim for is being able to deploy the entire system with a single action. If a system is easy to develop, it doesn't necessarily mean that it's also easy to deploy: for example, a micro-service architecture at the early stages of development will probably not bring sensible advantages for development, while at the same time will definitely make the system very expensive to deploy.

Another factor that comes into play is *use cases* and *operation*. Usually inefficient architectures are made up for just throwing more hardware at them, like more storage or more servers, because hardware is cheaper than development time. This in fact means that operation costs are lower than development and deployment costs. However, a good architecture should still strive to make clear what the operational needs of the system are, and it does this clearly showing what the use cases of the system are, so that the intent of the system is clear.

Finally, the last thing to consider is *maintenance*, which is the most expensive aspect of a software system. Of course the architecture of the system should make it easier to maintain, meaning to make changes to it. This in turn means that the system should be easy to inspect, to figure out what's the best place to perform the change, and easy to change, meaning that changes won't inadvertently cause other parts of the system to break.


### Decoupling into layers and use cases

Balancing all these factors is of course hard, for many reasons. Use cases may not be fully known in advance, nor may be operational constraints or deployment requirements; additionally, these factors may change during the system's life cycle. The key to let the system be easy to change, is leave as many options as possible open, so to defer taking most decisions at a later time. To achieve this, we need to decouple elements of the system that change for different reasons, and at different rates.

Layers usually are defined to contain elements that change for the same reasons, and at the same rate, so they're one obvious way to decouple a system. Even if not all use cases are known in advance, we know that, whatever the system might be, user interfaces change for different reasons than business rules, as does storage systems.

In addition to layers, use cases themselves change for different reasons: for example, the use case for adding an order will likely change for different reason than the use case for deleting an order; thus, use cases are another natural way of decomposing the system. We can think that use cases are vertical slices that cut the horizontal layers, so that each layer is further divided in use case portions: for example, we'd have the UI for the add-order use case, separated from the UI for the delete-order use case. This way, we can continue adding new use cases without interfering with old ones.

Once we have separated each use case in the elements belonging to each layer, we can already decide to run different elements at different throughput, when the need arises: if UI and database have been separated for a use case, they can be run on different servers, and those needing more bandwidth can be replicated in many servers.

Additionally, when components have been decoupled, also development and deployment are greatly improved, because teams can be assigned to the development of different components, provided the interfaces between each other have been properly designed, and the deployment can be done separately for each.

When decoupling use cases, it may be the case that some code will seemingly need to be duplicated among different components, so there may be the temptation to isolate this code into its own library, that will then become a dependency of both components. Now, it's important to understand that if two pieces of code are very similar, but they have different responsibility, meaning that they change for different reasons and at different rates, then they're not the same thing, and they're also not duplicated. It would be a troublesome mistake to make two components depend on the same shared library only for the sake of reusing code that at a certain moment looked similar, just to discover later on that the two use cases demand that that library change for different reasons and at different rates. The same can be said for the layers decomposition: even if some UI code looks similar to database code, they are still different, because they have different responsibilities.


### Decoupling modes

Layers and use cases can be decoupled at source code level, at deployment level, and at service level. To decouple at source code level we can control dependencies between source code modules so that changes in one don't force changes or recompilation of others; still, all components execute in the same address space, communicate with simple function calls, and there's a single executable loaded into memory: this is called the monolithic structure.

We can decouple at deployment level, making sure that different deployable units can be changed without requiring re-deploying of others. Still, many components can live in the same address space, but this time we have the freedom to move some components to different processes or servers.

Finally, decoupling at the service level means that modules are deployed to different servers, and communicate with each other solely through the network.

Typically, projects start with the monolith structure, and are decomposed later on into different deployable units, or different services, as the need arises. The important thing is that components should be designed to be decoupled either way, even if at the beginning we think that the project will always remain a monolith: this way, it'll be easier to move components to their own processes or servers will that be ever necessary. It's also true that over time a system deployed into multiple services may need to be simplified back to a multi-process, or even monolith system. If the architecture has been properly designed, this would not be a problem, because the majority of code will be protected from these changes.


### Boundaries

Components are defined by the boundaries that separate them. Relations between different components come in the form of functions on one side of a boundary calling functions on the other side. Given this relation, when one side of the boundary changes, the other one may be forced to change as well, or to be recompiled. Since the ultimate purpose of partitioning applications into components is to control change, the key to define boundaries, and thus components, is managing source code dependencies.

The simplest situation is that of the *monolith*. In this case the application is a single executable file executing in a single process and address space (threads can still be used, as they cannot act as architectural boundaries). We could distinguish here between statically linked binaries, and dynamically linked ones, because in the second case we could use external binaries as a form of architectural boundary. Except for this case, in the monolith situation there are no physical boundaries between components, which means that developers must be disciplined enough to stick to their design principles when it comes to keep components decoupled.

Communication between monolith's components is very easy, because it's always just a function call. This means that components in a monolith risk to be very chatty, and thus that there might be too many points of contact between components.

The first kind of physical architectural boundary is the *local process*. Local processes are created from the command line, or system calls in general; they run in the same processor, or in the same set of processors in a multicore, as the original process, but in a different address space. Code running in different address spaces don't share memory, even though it's possible to use shared memory partitions to overcome this limit.

When there's no shared memory, different processes can communicate with each other via system-level facilities, usually sockets, but also mailboxes or message queues. Each process usually contains a statically or dynamically linked monolith (in the second case the same components used by each process can be compiled and deployed only once, since they're shared).

Communication in this case involves marshaling and decoding data, and interprocess context switches, which limits the chattiness of components. Still, low-level processes are plugins to higher-level processes, because dependencies should always go from low-level processes to high-level ones.

The strongest kind of physical architectural boundary is the *service*. A service works like a local process, with the difference that it's not necessarily local anymore: in fact a service can be placed both locally, and on another node in a network. For this reason, services always assume they're communicating over a network, being the worst case possible.

Communication between services are the slowest, thus chat should be avoided wherever possible.

Of course, in a realistic situation many different kinds of boundaries might be used: for example a service might use local processes.


### The Screaming Architecture

Much like the intent of a building can be clearly guessed by just looking at its blueprints (whether it's a house, a library, etc.), thus the intent of a software system should be clearly stated by its top level modules. The architecture of a system should "scream" its intent, whether it's an accounting system, a health care system, etc. rather than what style of architecture or framework it's using (MVC, Spring, etc.).

The intent of a software system is described by its use cases. Going back to the buildings analogy, the intent of a building is stated by, among other things, the kind of rooms that are used (which are usually related to the "use cases" of the building, e.g. what the building will be used for), so that for example a building with a kitchen, a living room, a restroom, etc., can be safely identified as a house.

In software systems, use cases play the same role as room types, so that just looking at the use cases fulfilled by the system you should be able to guess the intent of the system itself, for example that it's an accounting system.

Furthermore, as in the buildings analogy, where the design is made so the actual construction materials can be decided later, after the design is finished and has been approved, the use cases must be completely independent from the actual frameworks and tools that will be used to implement them, so that the choice of which technology to use can be made later on, without affecting the original design.


### The Clean Architecture

Hexagonal architecture is only one of several architectural styles that have emerged in the latest years, among which clean architecture and onion archtecture, that are all trying to address the same problem: decoupling application and business code from infrastructure code.

All these kinds of architecture, also, suggest to respect the dependency inversion principle, meaning that the dependencies relations should only go from the outside in: outer components are more concrete than, and depend on, the inner ones, which on the other hand are more abstract and independent.

At the innermost layer, are located *entities*, which encapsulate enterprise business rules, which are the most generic and high level of the entire application, and the least likely to change. They have no knowledge of application navigation, security or operations, for example.

Surrounding the entity layer we found *use cases*, containing application specific business rules. These are all the use cases of the system, necessary to orchestrate the flow of data to and from entities, so that entities can use their enterprise business rules to achieve the goals of the use cases. Since entities are independent from use cases, changes in use cases don't affect entities. Also, use cases are independent from external concerns, like routing and databases.

The next layer, just outside the use cases layer, contains adapters used to convert data from the format used by the infrastructure (the Web, databases, etc.) to the format required by use cases and entities. This, for example, is the layer where all MVC components will be implemented. No code inward of this layer should know anything about the structural details: if some SQL is used, all of it will be located inside the adapters, and no SQL knowledge of any kind should creep into the inner layers.

The outermost layer typically contains third party components, like a framework or libraries, and implements every detail of the application: the fact that the application supports a Web adapter is just a detail, like the fact that a specific RDBMS is used, and inward layers should not be aware of any of this. Of course, these are also the elements of the application that are most likely to change.

The communication between boundaries must be done solely by anemic data structures, like DTO's. The important thing is that no assumption or knowledge about external layers is ever contained in these objects. For example, if the database library wraps the data returned from a query in a row-like structure, we should never pass this object as-is to the inner layers, because this object would carry the information that a database with a concept of a row is being used, and at that point the inner layers would be tightly coupled to the application details, breaking the dependency inversion principle.


## References

- "Clean Architecture, A Craftman's Guide to Software Structure and Design", Martin, 2018
