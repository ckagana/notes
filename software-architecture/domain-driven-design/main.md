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