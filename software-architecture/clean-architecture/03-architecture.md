# Architecture

A good architecture is judged from several key indicators. First of all a good architecture should take care of *development* issues. The system should be easy enough for developers to work with: a small team of developers can more easily work on a monolith system without well defined components or interfaces; on the other hand, a system being developed by several teams concurrently will need a proper separation in well-defined components with stable interfaces.

The second element architectures should take into account is *deployment*: the goal to aim for is being able to deploy the entire system with a single action. If a system is easy to develop, it doesn't necessarily mean that it's also easy to deploy: for example, a micro-service architecture at the early stages of development will probably not bring sensible advantages for development, while at the same time will definitely make the system very expensive to deploy.

Another factor that comes into play is *use cases* and *operation*. Usually inefficient architectures are made up for just throwing more hardware at them, like more storage or more servers, because hardware is cheaper than development time. This in fact means that operation costs are lower than development and deployment costs. However, a good architecture should still strive to make clear what the operational needs of the system are, and it does this clearly showing what the use cases of the system are, so that the intent of the system is clear.

Finally, the last thing to consider is *maintenance*, which is the most expensive aspect of a software system. Of course the architecture of the system should make it easier to maintain, meaning to make changes to it. This in turn means that the system should be easy to inspect, to figure out what's the best place to perform the change, and easy to change, meaning that changes won't inadvertently cause other parts of the system to break.


## Decoupling into layers and use cases

Balancing all these factors is of course hard, for many reasons. Use cases may not be fully known in advance, nor may be operational constraints or deployment requirements; additionally, these factors may change during the system's life cycle. The key to let the system be easy to change, is leave as many options as possible open, so to defer taking most decisions at a later time. To achieve this, we need to decouple elements of the system that change for different reasons, and at different rates.

Layers usually are defined to contain elements that change for the same reasons, and at the same rate, so they're one obvious way to decouple a system. Even if not all use cases are known in advance, we know that, whatever the system might be, user interfaces change for different reasons than business rules, as does storage systems.

In addition to layers, use cases themselves change for different reasons: for example, the use case for adding an order will likely change for different reason than the use case for deleting an order; thus, use cases are another natural way of decomposing the system. We can think that use cases are vertical slices that cut the horizontal layers, so that each layer is further divided in use case portions: for example, we'd have the UI for the add-order use case, separated from the UI for the delete-order use case. This way, we can continue adding new use cases without interfering with old ones.

Once we have separated each use case in the elements belonging to each layer, we can already decide to run different elements at different throughput, when the need arises: if UI and database have been separated for a use case, they can be run on different servers, and those needing more bandwidth can be replicated in many servers.

Additionally, when components have been decoupled, also development and deployment are greatly improved, because teams can be assigned to the development of different components, provided the interfaces between each other have been properly designed, and the deployment can be done separately for each.

When decoupling use cases, it may be the case that some code will seemingly need to be duplicated among different components, so there may be the temptation to isolate this code into its own library, that will then become a dependency of both components. Now, it's important to understand that if two pieces of code are very similar, but they have different responsibility, meaning that they change for different reasons and at different rates, then they're not the same thing, and they're also not duplicated. It would be a troublesome mistake to make two components depend on the same shared library only for the sake of reusing code that at a certain moment looked similar, just to discover later on that the two use cases demand that that library change for different reasons and at different rates. The same can be said for the layers decomposition: even if some UI code looks similar to database code, they are still different, because they have different responsibilities.


## Decoupling modes

Layers and use cases can be decoupled at source code level, at deployment level, and at service level. To decouple at source code level we can control dependencies between source code modules so that changes in one don't force changes or recompilation of others; still, all components execute in the same address space, communicate with simple function calls, and there's a single executable loaded into memory: this is called the monolithic structure.

We can decouple at deployment level, making sure that different deployable units can be changed without requiring re-deploying of others. Still, many components can live in the same address space, but this time we have the freedom to move some components to different processes or servers.

Deployable units are typically artifact created by building a module, including configuration, assets, etc. in addition to source-code compilation. Of course we want to reuse the already built artifacts as much as possible, and not having to go through the whole build process every time some dependency changes. If a class inside the module changes, the whole module must be redeployed, because we cannot reuse the artifact previously built for that module: for example, if the module contains class `A` depending on `B`, but only using a few features from it, when some of the unused features of `B` change, this means that `B` has changed, and the whole module must be redeployed, even if the changes weren't really meaningful for the module. If instead `A` depended on an interface `B'`, and concrete `B` implementing `B'` was injected into the module from the outside, at the time when the final system using the module is built, then changes to `B` won't cause any re-deployment of the module.*

Finally, decoupling at the service level means that modules are deployed to different servers, and communicate with each other solely through the network.

Typically, projects start with the monolith structure, and are decomposed later on into different deployable units, or different services, as the need arises. The important thing is that components should be designed to be decoupled either way, even if at the beginning we think that the project will always remain a monolith: this way, it'll be easier to move components to their own processes or servers will that be ever necessary. It's also true that over time a system deployed into multiple services may need to be simplified back to a multi-process, or even monolith system. If the architecture has been properly designed, this would not be a problem, because the majority of code will be protected from these changes.


## Boundaries

Components are defined by the boundaries that separate them. Relations between different components come in the form of functions on one side of a boundary calling functions on the other side. Given this relation, when one side of the boundary changes, the other one may be forced to change as well, or to be recompiled. Since the ultimate purpose of partitioning applications into components is to control change, the key to define boundaries, and thus components, is managing source code dependencies.

The simplest situation is that of the *monolith*. In this case the application is a single executable file executing in a single process and address space (threads can still be used, as they cannot act as architectural boundaries). We could distinguish here between statically linked binaries, and dynamically linked ones, because in the second case we could use external binaries as a form of architectural boundary. Except for this case, in the monolith situation there are no physical boundaries between components, which means that developers must be disciplined enough to stick to their design principles when it comes to keep components decoupled.

Communication between monolith's components is very easy, because it's always just a function call. This means that components in a monolith risk to be very chatty, and thus that there might be too many points of contact between components.

The first kind of physical architectural boundary is the *local process*. Local processes are created from the command line, or system calls in general; they run in the same processor, or in the same set of processors in a multicore, as the original process, but in a different address space. Code running in different address spaces don't share memory, even though it's possible to use shared memory partitions to overcome this limit.

When there's no shared memory, different processes can communicate with each other via system-level facilities, usually sockets, but also mailboxes or message queues. Each process usually contains a statically or dynamically linked monolith (in the second case the same components used by each process can be compiled and deployed only once, since they're shared).

Communication in this case involves marshaling and decoding data, and interprocess context switches, which limits the chattiness of components. Still, low-level processes are plugins to higher-level processes, because dependencies should always go from low-level processes to high-level ones.

The strongest kind of physical architectural boundary is the *service*. A service works like a local process, with the difference that it's not necessarily local anymore: in fact a service can be placed both locally, and on another node in a network. For this reason, services always assume they're communicating over a network, being the worst case possible.

Communication between services are the slowest, thus chat should be avoided wherever possible.

Of course, in a realistic situation many different kinds of boundaries might be used: for example a service might use local processes.


## The Screaming Architecture

Much like the intent of a building can be clearly guessed by just looking at its blueprints (whether it's a house, a library, etc.), thus the intent of a software system should be clearly stated by its top level modules. The architecture of a system should "scream" its intent, whether it's an accounting system, a health care system, etc. rather than what style of architecture or framework it's using (MVC, Spring, etc.).

The intent of a software system is described by its use cases. Going back to the buildings analogy, the intent of a building is stated by, among other things, the kind of rooms that are used (which are usually related to the "use cases" of the building, e.g. what the building will be used for), so that for example a building with a kitchen, a living room, a restroom, etc., can be safely identified as a house.

In software systems, use cases play the same role as room types, so that just looking at the use cases fulfilled by the system you should be able to guess the intent of the system itself, for example that it's an accounting system.

Furthermore, as in the buildings analogy, where the design is made so the actual construction materials can be decided later, after the design is finished and has been approved, the use cases must be completely independent from the actual frameworks and tools that will be used to implement them, so that the choice of which technology to use can be made later on, without affecting the original design.


## The Clean Architecture

Hexagonal architecture is only one of several architectural styles that have emerged in the latest years, among which clean architecture and onion archtecture, that are all trying to address the same problem: decoupling application and business code from infrastructure code.

All these kinds of architecture, also, suggest to respect the dependency inversion principle, meaning that the dependencies relations should only go from the outside in: outer components are more concrete than, and depend on, the inner ones, which on the other hand are more abstract and independent.


### Domain layer

At the innermost layer, are located *entities*, which encapsulate enterprise business rules, which are the most generic and high level of the entire application, and the least likely to change. They have no knowledge of application navigation, security or operations, for example.


### Application layer

Surrounding the entity layer we found *use cases*, containing application specific business rules. These are all the use cases of the system, necessary to orchestrate the flow of data to and from entities, so that entities can use their enterprise business rules to achieve the goals of the use cases. Since entities are independent from use cases, changes in use cases don't affect entities. Also, use cases are independent from external concerns, like routing and databases.

Use cases are implemented by *interactors*, defining how external clients can interact with the application. An interactor can be an object, like `CreateOrder`, having an `execute()` method that triggers the execution of the use case. This can very easily be seen as an implementation of the Command Pattern, and as such can be implemented in different ways with the same effect.

An interactor has *input boundaries* and *output boundaries*. In particular, the interactor implements the input boundary interface, because the input will be provided using the interactor itself. On the other hand, the output boundary is defined by the interactor, but implemented by device adapters, which are typically injected in the interactor, which uses them to provide output: being injected, the interactor never knows which specific adapter is being used each time.


### Infrastructure layer

The next layer, just outside the use cases layer, contains adapters used to convert data from the format used by the infrastructure (the Web, databases, etc.) to the format required by use cases and entities. This, for example, is the layer where all MVC components will be implemented. No code inward of this layer should know anything about the structural details: if some SQL is used, all of it will be located inside the adapters, and no SQL knowledge of any kind should creep into the inner layers.

When the end user performs some action on an adapter (for example filling a form or pushing a button), this action is caught by the *controller*, which creates a *request model*, which can be either a list of arguments to a method of the input boundary, or a full-fledged DTO, which is deployed through the input interface, into the interactor.

The interactor uses the data contained in the request model to perform its use case, collects the data that is possibly produced and needs to be output into a *response model* (again a DTO or a list of arguments to a method of the output boundary), and passes it back to the adapter through the output boundary.

The object, belonging to the adapter, that actually implements the output boundary is called a *presenter*. The presenter, when called by the interactor, outputs a *view model*, which is another DTO containing all the information needed to perform the rendering of the response in the adapter.

The *view* then takes the view model and renders the response, according to the instructions contained in the view model itself. The view has no functionality itself, besides the ability to follow the instructions of the view model, and as such doesn't usually need to be tested.

It's important to highlight that the view must be decoupled by the view model, and that's because there can be multiple views for the same view model. A primary adapter needs to send the response to the calling device, using the requested format; to switch formats, we abstract the concept of view, meaning that a view is responsible to render the response, but each different concrete implementation of a view will do it with a different format. Unlike the usage of interfaces done in other contexts, where it's used to receive concrete implementations from different components, so that the component using the interface knows nothing of the concrete implementations, instead when it comes to views, the current component contains already several different view implementations, but the one to be used is selected according to the client request.

When it's the application that initiates the communication instead, like when it has to use the database, the interactor uses a *gateway interface* to retrieve entities, composed with the data coming from the specific storage (secondary adapter), which is unknown by the interactor. The gateway interface is implemented by another adapter, written for the specific gateway (for example MySQL). This adapter knows all the technical details of the technology to which the application needs to talk.


### External layer

The outermost layer typically contains third party components, like a framework or libraries, and implements every detail of the application: the fact that the application supports a Web adapter is just a detail, like the fact that a specific RDBMS is used, and inward layers should not be aware of any of this. Of course, these are also the elements of the application that are most likely to change.

The communication between boundaries must be done solely by anemic data structures, like DTO's. The important thing is that no assumption or knowledge about external layers is ever contained in these objects. For example, if the database library wraps the data returned from a query in a row-like structure, we should never pass this object as-is to the inner layers, because this object would carry the information that a database with a concept of a row is being used, and at that point the inner layers would be tightly coupled to the application details, breaking the dependency inversion principle.
