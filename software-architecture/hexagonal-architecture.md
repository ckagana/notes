# Hexagonal Architecture

- [Shortcomings of layered architecture](#shortcomings-of-layered-architecture)
- [The layout of hexagonal architecture](#the-layout-of-hexagonal-architecture)
- [Ports and adapters](#ports-and-adapters)
- [References](#references)
- [Example](#example)
- [Resources](#resources)

The intent of hexagonal architecture is to "allow an application to equally be driven by users, programs, automated tests or batch scripts, and to be developed and tested in isolation from its eventual run-time devices and databases". Hexagonal architecture can be regarded as an attempt to improve on the classic layered architecture.


## Shortcomings of layered architecture

The classical implementation of layered architecture states that the presentation layer uses the domain layer (thus depending on it), which in turn uses the infrastructure layer (again, depending on it). This, however, breaks the dependency inversion principle, stating that abstract components should not depend on concrete ones, while instead details should depend on abstractions. In layered architecture, though, the domain components, which are by their nature quite abstract, depends on the technical details found in the infrastructure components.

If we apply the dependency inversion principle to the classical layered architecture, we get to a situation where both the presentation layer and the infrastructure layer depend on the domain layer (the abstract one). Since everything depends on the domain layer, while it itself depends on nothing, we have just achieved the layout that goes by the name of hexagonal architecture, where the domain layer is actually at the center, and dependencies are laid out around it.

Another problem with classical layered architecture is that several services might belong to different layers, while they must instead be put inside a specific one. For example, mediation, routing, orchestration, monitoring and auditing services could be associated to both the business layer, because they apply directly to entities of that layer, and to adjacent tiers (interface and data), because they are used to exchange information between them.

A better solution, instead, would be to recognize that these services belong to the boundary of the business layer, whatever other layer it is communicating with. Hexagonal architecture supports this kind of solution, since there the business layer takes the centermost position, and supporting services are relegated to the framework layer, surrounding it.


## The layout of hexagonal architecture

Hexagonal architecture, in contrast with layered architecture, particularly stresses the difference between the "inside" and the "outside" of an application. The inside of the application contains what layered architecture calls the application layer and the domain layer, which include all domain related objects (domain layer) plus all business services that define what can be done with the domain objects, e.g. use cases and business logic (application layer).

The outside of the application, also called the framework layer, contains all supporting services that are not directly related to the domain, nor to the application, like user interface, database access, communication, etc. This way business entities can be completely independent from the delivery model, which is useful both to better understand the application domain, and to test it.

Applying changes to the domain layer means changing the essential meaning of the application, and thus will be done only in response to explicit client requests. Everything else, instead, is a service: it can be freely changed without modifying the purpose of the application. Example of services are database access, user interface, communication, etc. The core logic of the application should not depend on those services: this way services can be changed or replaced as easily as possible.

As anticipated, the innermost layer is the *domain layer*, which contains domain objects, business rules, business events and use cases (domain commands), and defines the interfaces through which the outer layer can communicate with it. Of course the domain layer is completely independent from any other layer. The domain layer is what differentiates different applications of the same type, because it defines the actual behaviour and constraints of that project. The domain layer doesn't contain any application logic, though, like controller logic, nor event/command handling logic.

The *application layer* orchestrates the use of the domain objects found in the domain layer. The application layer depends on the domain layer, and on nothing else. For example, the application layer is responsible for handling the requests coming from the framework layer, and for dispatching and handling domain events and commands.

The *framework layer* contains library and framework code that is not part of the application itself. It implements interfaces defined in the application layer, like the ones for the event dispatcher and the command bus.


## Ports and adapters

As stated in the initial description of Hexagonal Architecture, the main drive for this approach is to allow different kinds of actors to use the application, such as users, programs, tests, etc., without requiring changes to most part of the application when new actors need to be supported. It turns out that fixing the problems with the Dependency Inversion Principle that bug the traditional layered application, we get to a design that can easily support multiple actors.

Hexagonal Architecture is based on the premise that an application might be required to talk to different devices. In order for two devices to be able to communicate, they must understand the same *protocol*: for example, the application (which is a device in itself) may expose some kind of API, but then an HTTP browser client might not directly understand it (since it only understands raw HTTP), so some kind of adaptation needs to be put in place.

In the context of Hexagonal Architecture, a protocol is defined for a specific purpose of communication, and *not* for a specific device: this means that the protocol is defined on the application's side. For example, the application might be exposing an API for handling shopping carts, which can be consumed by more than one client, like a Web client to be used by human users, and an HTTP API client to be used by other machines. Here we have only one protocol in place, which is the one defining what can be done with shopping carts, defined in the application, and all devices that want to use it, must adhere to it: it's not the case that there is a different protocol for each different device (Web and API in this case).

In Hexagonal Architecture, the part of the application that defines a specific protocol is called a *port*: in our previous example, then, our application will have a Shopping Cart port. Of course multiple ports might be defined, because there might be multiple purposes of communication: for example there could be an Administration port as well.

Communication purposes are usually identified as *actors*, which would be the fictitious entities using the system to carry out a specific purpose. For example, in an e-commerce application the Customer could be the actor with the purpose of buying goods: this communication purpose will be supported by the Shopping Cart port. Thus, in the design phase we first identify the actors of the system, and to each of them we can then associate a specific port.

Being an implementation of a specific communication purpose, each port can also be seen as a way to group related use-cases together. However, these are use cases of the application itself, which are different from the use cases of the devices: in fact each device can support many additional use-cases that the application knows nothing about, or that don't precisely match the application ones. Thus, another important goal of device adapters is that of translating the device's use cases to the application's ones (which might imply to join multiple device use cases into a single application use case, or to split a single device use case into multiple application use cases), and dealing with other situations such as devices' use cases not matching any application's one, or vice-versa.

As we pointed out, several different devices might be used for the same purpose of communication (port). However, each device will only understand its own protocol, which will likely be different from the one implemented by the port it's trying to talk to. This means that we need to put an *adapter* between the device and the port, to translate the device protocol to the port's one. Thus, every port will have a specific adapter attached to it, for every device that wants to communicate with it.

A *primary port* has the purpose of letting users communicate with the application, while a *secondary port* is used to let the application communicate to external devices (for example a database). Most applications actually only have a primary port, supporting the main features of the application, and a secondary port, to talk to a database. Likewise, a *primary adapter* will be attached to a primary port, and could be a system-level test suite, a GUI, another application (remote or local), etc., while a *secondary adapter* will be attached to a secondary port, and could be a database, a system to send notifications, etc. In particular, when it comes to tests, if the primary adapter is a system-level test suite, the correspondent secondary adapters will be mock objects.

Focusing on communication purposes (i.e. ports) instead of specific devices (i.e. adapters), allows the application to be independent from technical details, which are instead related to devices. This way the more abstract application can avoid being changed each time a device needs to be changed, which happens much more frequently.

Once we defined ports and adapters, with their specific use cases, we are able to also write specification (functional) tests for them. To ensure that the system works in real-life (at least with the happy flow, or other main scenarios), we write tests that exercise the specific features throughout the whole sequence of actions involved, touching all the real devices and modules that are used in a realistic scenario: these would then be integration, end-to-end tests.

From the testing point of view, however, we can notice that there are two cases we might fall into, which depend on the specific way the system is designed to be deployed. Let's say we have an application with a service module providing domain models, for example Orders; this service will be consumed through an interface it defines, by one or more consumers, that for example are displaying this data to a Web page, or a REST API.

This is already a situation where we have a port (the service module interface), and adapters (Web or API consumers): if we are deploying this application as a single, monolithic, bundle, containing the two adapters along with the internal service, we can write end-to-end tests that exercise the whole system, from the requests routed to the Web or API, through the internal service, to the responses returned back by the same adapters. We could call these, then, *internal ports* and *internal adapters*, where *internal* is from the perspective of the deployment strategy.

However, we could also decide to deploy the module providing Orders to its own process/service, and the Web and API clients separately to their own services. In this case, the Orders service would be regarded as an independent service provider, that should be guaranteed to work properly independently of which adapter client will connect to it: for this reason, in this case we need to write end-to-end tests for the domain service alone. For the adapters also, we need to write separate end-to-end tests for both the Web and the API one. In this case, we can talk of *external ports* and *external adapters*.

Of course the same application can have several internal and external ports and adapters, and they can also change from internal to external (and vice-versa) the moment a different deployment strategy is chosen. Whatever the situation is, though, it's important to notice that we are always in control of internal ports and adapters, because since we are shipping them together in the same bundle, we always know which of them we are adding to the system, and so we can make end-to-end tests taking all of them together.

On the other hand, if we have external ports, we cannot control what adapters may be connected to them, because they're designed and used by different teams, or even third parties. For this reason, we cannot write end-to-end tests that include external adapters: the best thing we can do is to write tests with "transparent" adapters, that use the external port without doing any protocol translation, and verify that the port responds the way it's supposed to do. In this case, we can still consider this an integration test, in the sense that we're still testing the integration of the internal components of the system (which may of course include other internal ports and adapters).


## Example

Let's finally see an example of ports and adapters for an application design according to Hexagonal Architecture. Let's consider a Weather System Application, taking weather data from different sources, and sending notifications about the weather to registered parties.

A first actor will be a Weather Source, representing the source of weather data: thus, Weather Data will be a primary port because each time new data is available, the source sends a message to the application with the new data; several different devices (and thus adapters) might act as Weather Data, for example a wire feed, an HTTP feed, and of course system test suites.

Next, the application is also used by an Administrator to tweak configuration, etc., making it necessary for the application to provide an Administration primary port: adapters for it could be a GUI, an HTTP API, other applications, and tests.

The application will then need to talk to Recipients of notifications about weather data: to do this it will provide a Notifications secondary port, which will interface with adapters for phone and email devices, in addition to mock objects for tests.

Finally, the application will need to persist some data into a Storage, through the Storage secondary port, talking to a database adapter and mocks.


## Resources

- http://blog.ploeh.dk/2013/12/03/layers-onions-ports-adapters-its-all-the-same/
- http://alistair.cockburn.us/Hexagonal+architecture
- http://geekswithblogs.net/cyoung/archive/2014/12/20/hexagonal-architecturendashthe-great-reconciler.aspx
