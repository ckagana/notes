# Domain-driven Design

- [Layered architecture](#layered-architecture)
- [Entities](#entities)
- [Value Objects](#value-objects)
- [Services](#services)
- [Modules](#modules)
- [Aggregates](#aggregates)
- [Factories](#factories)
- [Repositories](#repositories)
- [Constraints, processes and specifications](#constraints-processes-and-specifications)

## Layered architecture

Usually only a limited part of a software is related to the domain model. Many components provide functionality to help the main features mapped to the domain; such components deal for instance with database access, file access, user interfaces, etc. This distinction between main features and helper components is built upon the realization that functionality such as database access or user interface don't bear a semantic related to the domain, but instead they solve more generic problems. Following the Single Responsibility Principle, components belonging to the domain model should be kept separated from those providing more generic functionality.

This leads to the *Layered Architecture* pattern, where a program is partitioned into layers, which are cohesive (meaning that components inside a single layer share common semantics), and where each level depends only on the layers below (because more concrete layers should depend on more abstract ones, and not the other way around).

A common set of layers is the following:
- **User Interface (Presentation Layer)** Presents information to the user, and interprets its commands.
- **Application Layer** Thin layer orchestrating components located in other layers to provide business logic. Usually it does not hold any object state, except for object representing tasks being executed. It can expose an API of services.
- **Domain Layer** Contains object related to the actual domain model. Contains objects states but is not involved with their persistence.
- **Infrastructure Layer** Contains components providing support functionality like persistence.

This is a sample scenario involving this kind of layered architecture:
1. An user wants to book a flights route, and for this it uses a service exposed by the application layer.
2. The application layer uses the infrastructure layer to construct the relevant domain objects (for instance loading them from the persistence).
3. The application layer calls methods provided by the domain object just retrieved to meet the user needs. Notice that domain objects aren't concerned with knowing how to deal with various usage scenarios: this is responsibility of the application layer.
4. Once the computation done by domain objects is done, the application layer uses the infrastructure layer to persist domain objects again, that may have changed. It also notify the user of the result of the computation.

## Entities
Certain kinds of objects have an identity, which can be defined by a specific attribute, or a combination of attributes. The identity never changes, and no two objects have the same identity (or if it happens, they should be considered as the same object); it identifies an object even through operations that remove the object from memory, like sending the object through the network, or persisting it and then terminating the application. Objects of this kind are called *entities*.

The purpose of entities is specifically to maintain their identity. This means that entity objects shouldn't do more that exposing their identity, and ensuring the continuity of their life cycle. Their structure should be simple, and it should be easy to distinguish one from another, and tell if two objects are the same thing.

An important trait of entity objects is that they have a *history*, meaning that their current status is the result of the application of different operations to them during the course of time. Objects without an identity can be changed and become completely different objects, without this change having any meaning at all, and thus not developing a history for that object.

For instance, a `Customer` object may start having a `balance` of `10.00`, and then, after some operation is performed, it happens to have a `balance` of `20.00`. Although changed, this object always represented the same thing, the same customer than before, because it has an identity, and thus these different attributes it happened to have were part of his history.

## Value objects
Entity values come with the additional costs of increased complexity due to the need of defining an identity and making so that no two objects have the same identity, and in addition to that it needs to be created a distinct instance each time a new object is needed, even if they have exactly the same attributes' values: this is because entities have a history, so if at a certain moment two entities have the same properties, they may become different later.

On the other hand, a `Point` object representing a two-dimensional point in a plane, may have coordinates of `10,20`, thus referencing a specific point in the plane. Now, it doesn't make any sense to think of changing its coordinates later on, because points of a plane don't "move": the point of such an object is just to memorize a certain set of attribute values. This also means that, wherever in the application is needed a `10,20` point, the exact same object instance can be used, because what's being requested is always the same point, not a new point which now happens to have that coordinates, but which may change later on.

Objects that are interesting only for their properties' values, and that don't have identities, are called *Value Objects*. Since value objects are quite simpler that entities, it's a best practice to create entities only when strictly necessary, and make value objects in all other cases. Since value objects don't have an identity (nor a history), they can be created and discarded at will. Furthermore, since value objects are meant to represent a specific set of values, they should be immutable, and thus the same instances can be shared.

For instance, imagine that two clients book a flight for the same destination. Two distinct reservation objects are created (entities), one for each client: reservations have the flight code as a property (value object), which is shared between the two, because it's the same flight. Now imagine that the flight code is not immutable: if one client changes destination later on, the system changes the value of the flight code (being non immutable), instead of creating a new value object with the new flight code, and, being a shared object, also the other client sees his flight code change!

So, value objects that can be shared should always be immutable, and new value objects should be created each time a new value is needed. Value objects' attributes can be other value objects, or even references to entities, since even if the entities can change, they have an identity, and the value object is always pointing to the same entity (with the same identity).

## Services
While many actions being performed during software execution belong to specific objects, mainly because they need knowledge incapsulated in those objects to be performed, there are often actions that aren't related to any particular type of object. These are actions that don't need access to a specific set of information to be performed, rather, they may just reference whole objects, but without needing to know details incapsulated inside them. For instance, the action of transferring money from an account to another, involves two objects: a sender and a receiver, and there's really no reason to prefer `account1.sendTo(account2)`, to `account2.receiveFrom(account1)`.

These kinds of action should be located in *Service* objects. Services don't have an internal state, because they perform actions that don't need to know a specific set of information to work. To say it differently, if an action needs specific knowledge to be performed, it shouldn't belong to a service, but instead to the object that has that knowledge.

In a service, the service object is merely a device required by the fact that in object-oriented programming every operation must belong to an object: they might as well have been pure functions. What's important in services are the objects the operations are performed on. Furthermore, the fact that this kind of operations don't belong to any specific object, means that they are used to interconnect multiple objects among each other. Thus, services act like point of connection for many objects. Now, if such operations were located inside domain objects, we would end up with domain objects highly coupled to each other, while instead they should be loosely coupled.

Services are meant to act on domain objects, but this doesn't mean that all services should be located in the domain layer. Instead, there are usually many services that belong to the infrastructure or application layer. The important thing to consider when choosing the layer where to place a service, is not what objects it's acting on (because they are almost always domain objects), rather what's the purpose of the operation being performed by the service.

## Modules
When a domain object initially described with a single class start growing, and being given several different responsibilities, it's natural to refactor it extracting new classes. In this situation, the original object is not being described by a single class anymore, but instead by a collection of classes tightly related to one another: this is a *Module*, and by its nature it manifests high cohesion and loose coupling, as well as classes do.

Modules can be created not only extracting classes from an original class, but also grouping together related pre-existent classes. The important thing in both cases is that modules' elements are cohesive. From this perspective, two kinds of cohesion can be identified: *communicational cohesion* and *functional cohesion*.

We have communicational cohesion when different parts of the module work on the same data. Functional cohesion is achieved when different parts of the module are cooperating to achieve the same task.

Modules should expose only one interface. When a client needs to perform a task belonging to a module, instead of letting it use three different objects, it's better if it can just access a single interface: in this way we are reducing coupling, because the client now only needs to know one interface, instead of three different objects, and in the future it will be easier to apply changes to one single point (the interface) instead of to three different objects.

A module should represent one, and only one, specific concepts. The concept represented by a module should be easy to reason about, independently of other concepts (loose coupling between modules). If understanding a module requires to continuously refer to concept belonging to other modules, it means that the two modules are strongly coupled, and they should be redesigned.

## Aggregates
In every software model, objects are associated with one another, ending up in a complex net of relationships. For every association used in the model, there must be a way in the code to enforce it. Sometimes, associations can be enforced even in the database (for example with foreign keys).

For instance, a one-to-one relationship between a customer and a bank account (a bank account *has a* customer, and a customer *has a* bank account) is enforced in the code using references (properties) between the two objects (another enforcement in the database, by means of foreign keys, may or may not be used).

Anyway, it's very important that relationships in the model are reduced as much as possible, for the sake of simplicity (even if the domain has many more). To do this, we need to:
- identify domain relationships that are not necessary to the solution, and take them out of the model;
- try to impose constraints to relationships, in order to reduce the number of objects that satisfy that relationship;
- transform bidirectional associations into unidirectional ones (every customer has an account, and every account has a customer, but we can consider that in reality it's the customer who's owning the account, thus simplifying the relationship).

Even after having simplified the model as much as possible, we often end up with a lot of interrelated objects. This has several drawbacks: for instance, when an object is deleted, each object owned by it must be removed as well, but references to those objects may be kept by other objects as well. So, we need to check all the references to the objects we need to delete, scattered around the code; we face a similar problem when data of an object change, thus requiring that that object be properly updated throughout the system, ensuring data integrity; then, invariants must be enforced each time data change.

To ease these kinds of tasks, we use the *Aggregate* pattern. An aggregate is a group of objects that are considered a unit with regard to data changes, meaning that each time an object of the aggregate is changed, all other objects need to be changed as well, because of the nature of their relationships.

Each aggregate has one *root*, which is an entity, and it's the only object of the aggregate accessible from the outside. There shouldn't be other entities inside the aggregate, unless their identity is local, meaning that it makes sense only inside the aggregate (in other words, from the outside of the aggregate those entities don't exist, or have no meaning).

Since objects out of the aggregate can hold references only to the root, there's no way for them to change internal objects. This means that every change to them is forced to pass through the root, which can thus enforce the appropriate invariants.

It's possible to expose internal objects out of the aggregate, with the condition that they are not changed, or that references to them are not hold outside of the aggregate: a simple way to do so is to pass copies of the value objects.

If internal objects are stored in a database, only the root can be obtained through queries, and the other objects will then be retrieved through the root.

For example, consider an aggregate meant to manage customers' data. The root entity will be `Customer`, having a `customerID` (identity) and a `name`. Then other data of the user are stored in value objects, so that they can be reused for other customers. `ContactInfo` and `Address` are two such value objects. When an outside object needs to change the address of the customer, instead of accessing the `Address` object directly, it asks the customer, and it will perform the requested operation. This is also complying to the *tell, don't ask* principle.

## Factories
In the easiest case, when a client wants to construct an object, it will call its constructor, possibly passing some parameter to it. However, when a complex entity or aggregate needs to be constructed, the construction procedure is often very complex, involving creating multiple additional objects and setting up relationships among them. If the client had to be able to do this construction, it would need to know a lot of specific details about the internal structure of the entity or aggregate.

Complex object creation must then be responsibility of a *Factory*. They encapsulate the knowledge necessary for object creation, expecially aggregates. In particular, object creation should be an atomic procedure, meaning that if one component of the object cannot be created, an exception should be raised, such that an invalid object (half built) is never returned.

A *factory method* is a method belonging to an object, that encapsulates all the knowledge needed to create another variant of that object. This is expecially useful with aggregates: a factory method can be defined in the root, to create all value objects the aggregate is made of, enforcing the invariants, and return the finally built object. For instance, say we have a `Container` aggregate, representing a collection of `Component`s. The root is a `Container` object, featuring a `createComponent` method, that takes care of all details needed to create a new `Component`, and then returns a reference to it, or to a copy of it (to ensure that objects external to the aggregate aren't able to change internal elements).

When the creation of an object is more complex, or several other objects need to be created as well, the factory method is usually not enough anymore. For instance, consider what we must do to create the actual `Container` aggregate (and not just a new `Component`). In this case the factory, rather than being a method of `Container`, is a completely distinct object, dedicated to this task.

Creating an entity is different than creating a value object. Value objects must always be in their valid states, right from their construction: this means that we can't create a value object if we don't know all the details of its valid state (i.e. of his "value"). On the other hand, entities aren't immutable, so they can have only some of their attributes defined at creation, because the other can be set later on; however, identites need to be generated for each entity.

These are the cases when a factory is not needed, and a constructor is enough:
- The construction is simple.
- Creating the object does not involve creating other objects, and all attributes needed are passed via the constructor.
- The client needs to be aware of the specific implementation, for instance choosing the Strategy.
- The type we want to create is exactly the class we are using, so there's no class hierarchy, and no selection of the concrete type to be done.

## Repositories
After an object has been created, references to it need to be available to the clients that need to use that object. For instance, if the object is an entity, a reference to it may be retrieved from the root of an aggregate; or, in the case of a value object, it can be retrieved as a property of an Entity.

Without any sistematic way of organizing object references, client code risks of quickly becoming littered with references, making it highly coupled to various different objects.

A simple solution one could think of, is letting clients retrieve objects from the database: since entities are likely stored in the database, and value objects are usually parts of entities anyway, the greatest part of the objects a client will need can be fetched from the database. However, this solution has the downside that client code becomes strongly coupled with infrastructure implementation, making it fragile to changes, and even worse, through the database the client can access aggregates' private objects, breaking incapsulation. When most of the logic to retrieve objects is directly provided by the database, entities and value objects become mere data containers, making the domain model meaningless and irrelevant.

*Repositories* are objects used to encapsulate all the logic needed to obtain object references. Clients request objects to the repository, which may already have that object, or it could restore it from the persistence, and save it to reuse it later. Using repositories makes also easier to implement strategies, for instance choosing different persistence storages according to different policies.

For each type of object that needs global access, create an object that can provide the illusion of an in-memory collection of all objects of that type. This object will be accessed through a global interface, providing methods to add and remove objects, encapsulating the details of actually inserting or removing them to and from the persistence. Methods that allow the client to retrieve collections of objects based on certain criteria can also be provided.

The important thing to highlight here, is that repositories are needed only for those objects that need to be globally available. Objects that are needed only locally don't need a repository to be accessed.

Repositories can be confused with factories, because repositories are seen as creating objects as well. The important difference is that repositories reconstitute already existing objects from a storage, while factories create new ones from scratch. When a new object is to be stored, it is first created by a factory (since it's brand new), then passed to a repository to be persisted. Furthermore, factories are completely contained in the domain layer, so they have no knowledge of the infrastructure, while repositories need to interact with the persistence. Repositories and factories never directly communicate: the client asks the factory to create a new object, which is returned to the client, which then pass it to the repositories to be persisted: the repository doesn't know how the object was created.

## Constraints, processes and specifications
A *constraint* is used to express an invariant, which is a property of an object or computation that must be always valid, no matter what happens.

Let's consider a bookshelf: it has a collection of books and a capacity (maximum number of books that can be stored). The invariant here is that the size of the collection must always be less or equal than the capacity.

A simple way to implement this invariant is adding checks wherever that condition may be invalidated: for instance, inside the `add(Book book)` method, we can check if there's still at least one place left for a new book, otherwise throw an exception.

However, it's easy to see how this can lead to repetition of the same checks all over the place. A better design would involve extracting the invariant in its own method, like `isSpaceAvailable()`. In this way, expecially, we are clearly stating that this is an invariant.

A *process* is sequences of computations to be performed; they are usually expressed as procedures (methods). When their logic become quite complex, it could be a wiser choice to express them with services instead. If there are multiple ways to carry out the process, this means that the client will only be faced with the process interface, and different strategies will be chosen to implement the actual process.

A *specification* is used to test an object to see if it satisfies a certain criteria. For instance, checking if a customer is eligible to a discount, may involve doing several domain-related computations, and accessing a lot of data. All the code needed to perform these operations will make the customer object bloated: for this reason a new object should be created, containing a series of boolean methods checking if the customer is eligible for a discount or not. Each method performs a single, specific, test, and all methods combined can answer the original question. A specification can also be used to select a certain object from a collection, using complex choice criteria, or as a condition during the creation of an object. Often specifications can be combined in composite ones like this:

```java
Customer customer = customerRepository.findCustomer(customerIdentity);
Specification customerEligibleForRefund = new Specification(
    new CustomerPaidHisDebtsInThePast(),
    new CustomerHasNoOutstandingBalances()
);
if (customerEligibleForRefund.isSatisfiedBy(customer)) {
    refundService.issueRefundTo(customer);
}
```
