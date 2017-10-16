# Hexagonal Architecture

- [Shortcomings of layered architecture](#shortcomings-of-layered-architecture)
- [The layout of hexagonal architecture](#the-layout-of-hexagonal-architecture)
- [Ports and adapters](#ports-and-adapters)
- [References](#references)

The intent of hexagonal architecture is to "allow an application to equally be driven by users, programs, automated tests or batch scripts, and to be developed and tested in isolation from its eventual run-time devices and databases". Hexagonal architecture can be regarded as an attempt to improve on the classic layered architecture.


## Shortcomings of layered architecture

The classical implementation of layered architecture states that the presentation layer uses the domain layer (thus depending on it), which in turn uses the infrastructure layer (again, depending on it). This, however, breaks the dependency inversion principle, stating that abstract components should not depend on concrete ones, while instead details should depend on abstractions. In layered architecture, though, the domain components, which are by their nature quite abstract, depends on the technical details found in the infrastructure components.

If we apply the dependency inversion principle to the classical layered architecture, we get to a situation where both the presentation layer and the infrastructure layer depend on the domain layer (the abstract one). Since everything depends on the domain layer, while it itself depends on nothing, we have just achieved the layout that goes by the name of hexagonal architecture, where the domain layer is actually at the center, and dependencies are laid out around it.

Another problem with classical layered architecture is that several services might belong to different layers, while they must instead be put inside a specific one. For example, mediation, routing, orchestration, monitoring and auditing services could be associated to both the business layer, because they apply directly to entities of that layer, and to adjacent tiers (interface and data), because they are used to exchange information between them.

A better solution, instead, would be to recognize that these services belong to the boundary of the business layer, whatever other layer it is communicating with. Hexagonal architecture supports this kind of solution, since there the business layer takes the centermost position, and supporting services are relegated to the framework layer, surrounding it.


## The layout of hexagonal architecture

Hexagonal architecture, in contrast with layered architecture, particularly stresses the difference between the "inside" and the "outside" of an application. The inside of the application contains what layered architecture calls the application layer and the domain layer, which include all domain related objects (domain layer) plus all business services that define what can be done with the domain objects, e.g. use cases and business logic (application layer).

The outside of the application, also called the framework layer, contains all supporting services that are not directly related to the domain, nor to the application, like user interface, database access, communication, etc. This way use business entities can be completely independent from the delivery model, which is useful both to better understand the application domain, and to test it.

Applying changes to the domain layer means changing the essential meaning of the application, and thus will be done only in response to explicit client requests. Everything else, instead, is a service: it can be freely changed without modifying the purpose of the application. Example of services are database access, user interface, communication, etc. The core logic of the application should not depend on those services: this way services can be changed or replaced as easily as possible.

As anticipated, the innermost layer is the domain layer, which contains domain objects, business rules, business events and use cases (domain commands), and defines the interfaces through which the outer layer can communicate with it. Of course the domain layer is completely independent from any other layer. The domain layer is what differentiates different applications of the same type, because it defines the actual behaviour and constraints of that project. The domain layer doesn't contain any application logic, though, like controller logic, nor event/command handling logic.

The application layer orchestrates the use of the domain objects found in the domain layer. The application layer depends on the domain layer, and on nothing else. For example, the application layer is responsible for handling the requests coming from the framework layer, and for dispatching and handling domain events and commands.

The framework layer contains library and framework code that is not part of the application itself. It implements interfaces defined in the application layer, like the ones for the event dispatcher and the command bus.


## Ports and adapters

An alternative name for the hexagonal architecture is "ports and adapters", because all communication between layers is designed to happen by means of ports and adapters. Ports are interface elements exposed by the inner layers, like the domain layer, to be consumed by adapters defined in the outer layers (like the framework layer). Thus, a port is a way to accept requests into the application, or to send requests outside of it.

In general, ports are the way we use abstract types to encapsulate changes: to avoid having to change the application each time a change in some requirement or technical choice is needed, we encapsulate these changes into the concrete adapters, and let the application deal only with abstract interfaces that are not subject to changes (in the worst case wrappers may be created to bridge different interfaces).

*Primary ports* allow direct outside-in communication: they usually are simple methods exposed by domain objects, that are called by outer layers, for example to make changes to the domain objects themselves. Thus, *primary adapters* are objects of the outer layers that directly call primary ports.

*Secondary ports* allow indirect, inside-out communication: they are usually defined as interfaces in the domain layer. The *secondary adapters*, in this case, are implementations of those interfaces, defined in the outer layers, and injected into the domain objects. This way, domain objects can use services belonging to the outside layers (like storage services), without being coupled to them.

For example, the application can have a Storage port (interface), and the surrounding framework will provide adapters that fit that specific port. It's important to notice that the application doesn't know about the existence of the framework: it just requires an object implementing the Storage adapter interface: this would be the "port". The framework, when bootstrapping the application, will inject a concrete Storage adapter (the actual "adapter") into the components requiring it. This separation will allow us to freely swap the concrete adapter, and thus the Storage technology we want to use (from MySQL to SQL Server for example).

Adapters implemented at the outmost level represent both components of the classic presentation layer, and components of the classic infrastructure layer, because these layers have now been crammed into the same space. In addition to them, the framework layer will contain also other kinds of adapters, for example unit tests. Since it's important that adapters having different intents don't talk directly to each other (because they would by-pass important business logic), it's important to introduce a segregation mechanism between them, so that each adapter is forced to talk only to the inner side, and never to other adapters. For example, a service in the presentation layer should never use a storage service.


## Clean architecture

Hexagonal architecture is only one of several architectural styles that have emerged in the latest years, among which clean architecture and onion archtecture, that are all trying to address the same problem: decoupling application and business code from infrastructure code.

All these kinds of architecture, also, suggest to respect the dependency inversion principle, meaning that the dependencies relations should only go from the outside in: outer components are more concrete than, and depend on, the inner ones, which on the other hand are more abstract and independent.

At the innermost layer, are located *entities*, which encapsulate enterprise business rules, which are the most generic and high level of the entire application, and the least likely to change. They have no knowledge of application navigation, security or operations, for example.

Surrounding the entity layer we found *use cases*, containing application specific business rules. These are all the use cases of the system, necessary to orchestrate the flow of data to and from entities, so that entities can use their enterprise business rules to achieve the goals of the use cases. Since entities are independent from use cases, changes in use cases don't affect entities. Also, use cases are independent from external concerns, like routing and databases.

The next layer, just outside the use cases layer, contains adapters used to convert data from the format used by the infrastructure (the Web, databases, etc.) to the format required by use cases and entities. This, for example, is the layer where all MVC components will be implemented. No code inward of this layer should know anything about the structural details: if some SQL is used, all of it will be located inside the adapters, and no SQL knowledge of any kind should creep into the inner layers.

The outermost layer tipically contains third party components, like a framework or libraries, and implements every detail of the application: the fact that the application supports a Web adapter is just a detail, like the fact that a specific RDBMS is used, and inward layers should not be aware of any of this. Of course, these are also the elements of the application that are most likely to change.

The communication between boundaries must be done solely by anemic data structures, like DTO's. The important thing is that no assumption or knowledge about external layers is ever contained in these objects. For example, if the database library wraps the data returned from a query in a row-like structure, we should never pass this object as-is to the inner layers, because this object would carry the information that a database with a concept of a row is being used, and at that point the inner layers would be tightly coupled to the application details, breaking the dependency inversion principle.


### Use cases and screaming architecture

Much like the intent of a building can be clearly guessed by just looking at its blueprints (whether it's a house, a library, etc.), thus the intent of a software system should be clearly stated by its top level modules. The architecture of a system should "scream" its intent, whether it's an accounting system, a health care system, etc. rather than what style of architecture or framework it's using (MVC, Spring, etc.).

The intent of a software system is described by its use cases. Going back to the buildings analogy, the intent of a building is stated by, among other things, the kind of rooms that are used (which are usually related to the "use cases" of the building, e.g. what the building will be used for), so that for example a building with a kitchen, a living room, a restroom, etc., can be safely identified as a house. In software systems, use cases play the same role as room types, so that just looking at the use cases fulfilled by the system you should be able to guess the intent of the system itself, for example that it's an accounting system. Furthermore, as in the buildings analogy, where the design is made so the actual construction materials can be decided later, after the design is finished and has been approved, the use cases must be completely independent from the actual frameworks and tools that will be used to implement them, so that the choice of which technology to use can be made later on, without affecting the original design.


## Resources
- http://www.dossier-andreas.net/software_architecture/ports_and_adapters.html
- http://geekswithblogs.net/cyoung/archive/2014/12/20/hexagonal-architecturendashthe-great-reconciler.aspx
- https://dzone.com/articles/hexagonal-architecture-is-powerful
- http://blog.ploeh.dk/2013/12/03/layers-onions-ports-adapters-its-all-the-same/
- http://fideloper.com/hexagonal-architecture
- https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html
- https://8thlight.com/blog/uncle-bob/2011/09/30/Screaming-Architecture.html