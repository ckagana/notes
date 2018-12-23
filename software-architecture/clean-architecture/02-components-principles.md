# Components Principles


## Component Cohesion

### The Reuse-Release Equivalence Principle

The Reuse/Release Equivalence Principle states that to be reused, software components must be tracked through a release process. This is needed first because this way users will know if and when components are changed, and what are the possible compatibility issues.

The problem with releases is that we need to create releasable artifacts containing software elements that are somewhat cohesive: the worst case scenario is when different components must be released at the same time because elements that change together are scattered among different components; in this case elements that are releasable together are not part of the same releasable artifact.


### The Common Closure Principle

The Common Closure Principle is the equivalent of the Single Responsibility Principle, applied to components instead of low-level elements. When an application has to change, it's of course better if all changes occur in only one component, instead of being scattered through multiple components: this way we need to re-compile and re-deploy only one component.


### The Common Reuse Principle

If the Common Closure Principle states that classes that tend to change for the same reason should be put in the same component, the Common Reuse Principle states that classes that tend to be reused together should belong to the same component. Typically, classes that are reused together are dependent on each other, so it makes sense that they are left together.

A typical violation of this principle is when a class `A` depends on another class `B`, which is included in a component `C`. When `B` is changed, typically also `A` needs to be changed, or at least re-deployed. However, if another class `D` of `C`, that is not cohesive with `B`, and as such is not needed by `A`, is changed, the component `C` still needs to be re-deployed, and thus `A` needs to be re-deployed as well, because it depends on that component. The end result is that `A` would need to be re-deployed because of completely irrelevant changes made to `C`.

Thus, we can see the Common Reuse Principle as a generalization of the Interface Segregation Principle, in the sense that it states that we shouldn't depend on things that we don't need, and to achieve this we should put into a component only classes that are strictly bound to each other, so that if we depend on one of them, obviously we depend also on every other one, and it will never happen that we need to re-deploy because of changes to unrelated classes in the component.


## Tension of Components

The previous principles apply different, and opposite forces to the process of components design. The Reuse/Release Equivalence Principle and the Common Closure Principle tend to put more stuff into components, making them larger, to avoid doing unnecessary releases, and to avoid doing unnecessary deployments; the Common Reuse Principle, instead, tend to take stuff away from components, making them smaller, to avoid unnecessary dependencies.

It's clear that it's not possible to satisfy all these principles at the same time, and that some trade-off needs to be done when designing the components of an architecture. If we focus mostly on maximizing reuse and minimizing dependencies, we end up with a system hard to maintain, because requires too many components changes and re-deploys; if we focus on reuse and maintenance, we end up with a system with too many dependencies, and thus releases; if we focus on maintenance and dependencies, we end up with a system which is hard to reuse.

Generally, at the beginning of a project, reusability is less important than maintainability, so we should focus more on the Common Closure Principle, and on the Common Reuse Principle; as the project matures, we will be able to put more effort on reusability, mostly because cohesive components will have emerged in the meanwhile.


## Component Coupling

### The Acyclic Dependencies Principle

We can draw a diagram of the components structure of an application, where components are linked by arrows showing dependency relationships, so that a component that depend on another has an arrow going from itself to its dependency. For example, the `Main` component depends on many other components, but no component depend on it: thus, `Main` would have many arrows going to other components in the diagram, and no arrow coming to it.

In a typical application structure, we'd have `Interactors` depending on `Entities`, `Database` depending on `Interactors` and `Entities`, `Authorizer` depending on `Interactors`, `Presenters` and `Controllers` depending on `Interactors`, and `Views` depending on `Presenters`; furthermore, `Main` depends on `Presenters`, `View`, `Interactors`, `Controllers`, `Database` and `Authorizer`. Let's say the team responsible for `Presenters` makes a new release: looking at the diagram, it's very easy to see which other teams are affected by this release, that is `Main` and `View`, meaning that the developers of these two components will have to decide if they want to update their components with the new version of `Presenters`, or if they want to keep using the old one. As another example, no component depends on `Main`, and thus we can make any change to `Main` without worrying of affecting any other team.

It's important that the components structure diagram be a *directed acyclic graph*, meaning that is has no cycles in it: it's impossible to travel from one component, following the dependency relationships, and ending up in the same starting component. For example, let's say that `Entities` is changed so that it now depends on `Authorizer`: suddenly, a dependency cycle is formed, because `Interactors` depends on `Entities`, which depends on `Authorizer`, which depend on `Interactors` itself. This way, `Interactors`, `Entities` and `Authorizer` have become a single, big, component, where all teams must always use the same versions to avoid merge conflicts, unit testing components depending on this cycle is very difficult, and there is no correct order to build these components, because they depend on each other.

The Acyclic Dependencies Principle states that there should be no cycles in the application's dependency structure. To break cycles, we could apply the Dependency Inversion Principle, for example making `Entities` depend on an interface that is implemented by `Authorizer` instead; or we might create a new component that both `Entities` and `Authorizer` depend on (which is the same as the previous solution, only at a higher architectural level).


### The Stable Dependencies Principle

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


### The Stable Abstractions Principle

High level software policies should not change in response to change to low-level details, like the user interface. This means that high level policies should be put into the most stable components of the system, even maximally stable ones with *I* equals to 0. However, even though it's good for these components to be stable, as far as dependencies are concerned, they should still be able to change, for example because some business logic has to change. To be able to change the logic without changing the component, we make use of the Open-Closed Principle, where instead of changing the high level classes, we extend them: for this to work, then, we need these high-level policy classes to be abstract.

The Stable Abstract Principle states that stable components should be abstract, so that their stability doesn't prevent them from being extended, and that an unstable component should be concrete, because its instability allows the concrete code to be easily changed.

We can define Abstractness *A* as the ratio between the number of abstract classes and interfaces in the component *Na*, and the total number of classes in the component *Nc*, so that *A* = *Na*/*Nc*. We can then plot each component on a *I*, *A* graph, where the coordinates of each component are their measures of *I* and *A*.

Components falling near the origin (0, 0) are not desirable, because they are at the same time stable and concrete, thus very painful to change: however, some components inevitably fall in that area, like database schemas (which are notoriously problematic to change), or utility library classes, like `String`, which is not going to change anyway.

Components falling near (1, 1) are said to be useless, because they are very abstract, but few or no other classes depend on them: they might appear after a refactoring where some abstract class or interface is left unused in the system.

The ideal area where components should fall is the one around the segment connecting (0, 1) to (1, 0), called the *Main Sequence*, where components are either abstract and stable, or volatile and concrete, as the Stable Abstractions Principle requires. We can measure the distance of a component from the Main Sequence, as *D* = |*A* + *I* - 1|. Components with a *D* of 0 fall directly on the Main Sequence, while those with a *D* of 1 are as far away as possible from it. A good architectural design has most components with small values of *D*.
