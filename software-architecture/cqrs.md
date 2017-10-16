# CQRS

CQRS puts commands and events at the first place in the application design, rather than entities and values. The rationale behind this is that an application's features are usually best described in terms of verbs (actions) rather than nouns (objects).

Given the requirements for an application, the first thing we do is making a list of things that can happen within the domain, like `OrderPlaced`, or `ApplicationSent`: this are *domain events*, and since they describe things that have happened, they are named in the past tense.

Then, we can identify things that are requested to the domain (to the application), like `AddToCart` or `SavePost`: these are *domain commands*, and since they describe actions that are requested to be executed, they are named in the imperative mood. Since these actions are not happened yet, they may be accepted or rejected: an accepted command leads to possibly some events being sent, while a rejected command usually leads to a *domain exception* being thrown. Thus, also domain exceptions should be modeled, like `InvalidEmailInserted`, or `PaymentRefusedByGateway`: these should try to explain why the command failed.

The nouns that commands and events are related to are *domain aggregates*, which capture the application state. Each aggregate has its own stream of related events: reproducing all events related to an aggregate, that happened since the current moment, we can get to the current state of the aggregate. Once we know the current state of the aggregate, we can use this information to understand if a related command must be accepted or rejected.

Representing the whole domain logic in terms of events and commands is particularly useful when it comes to acceptance testing. In fact, a typical test would go like: "given a stream of events, when a command is executed, then a certain event should be sent". This, however, is not enough to be sure that the correct domain logic is implemented, because we could have commands that just send the right events, to have all tests pass. Since we cannot expose any state to test, we rely on indirect exception testing: we can write tests that ensure that in certain situations exceptions are thrown, like: "given a stream of events, when a command is executed, then an exception is thrown". In our earlier implementation, where commands just threw the right events, these new tests would now fail, and we would be forced to write the proper domain logic to make them pass.

Although events are normally fired by commands, it's not unusual that some events are fired by other events as well, if the business rules require it. For example, the event handler of `OrderPlaced` could check if the amount is greater than a certain threshold, and send a `BigOrderPlaced` event, reacting to which another handler could apply a discount.

Commands are used to change the state of an application: thus they form the *write model*. However, we also need to query the application for information, and for this we need a *read model*. The read model is triggered by the events produced by the domain, and, reacting to them, it stores related information in many possible ways: in fact, many read models can react to the same domain event. Domain events are the actual authoritative source of information describing the normalized state of the system, and read models can be used as a source of denormalization, typically to present data in various forms, according to the needs of the application. This can have performance advantages, since the denormalization (think of a SQL `JOIN`) will happen once per change (event) rather than once per query.

Read models are designed according to the different ways information need to be presented to the users: for example, in an online shop the list of goods in the back-office could be a different read model than the list of goods in the front-office. Also, query methods exposed by read models (repositories) should not be generic ones like `getAll`, or `findByParam` but should be precisely crafted according to the specific types of information we want to get, like `getItemsRelatedByCategory`.

Read models are designed for maximum performance, and thus they directly talk to repositories specially designed to handle denormalized data, or even caches. This means that read models don't use anything in the domain layer: they actually never cross that boundary, but are completely implemented inside their specific adapters (Web, console, etc.), since also the repositories (gateways) are implemented inside some other adapter.

Commands are meant to be pure, meaning that they should not cause side effects external to the system, like sending emails (they can of course cause internal side effects, because they're the only way changes can be applied to the system). The only result of a command being accepted is some processing related to the domain logic being executed, and potentially some events being sent. This way, commands turn out to be idempotent, meaning that they can be executed any number of times, without putting the system in an inconsistent state. This way, we can setup a system to automatically re-try to run commands when they are rejected for certain reasons (like failed network connection). External side effects, on the contrary, should only be caused in reaction to events.

Each command should be handled by only one handler, to ensure transaction consistency in case the application is distributed. The handler can be a distinct object, or a method of the aggregate. Each command handler must also reference only a single aggregate.


## Resources
- http://www.cqrs.nu
- http://codeofrob.com/entries/cqrs-is-too-complicated.html
- https://msdn.microsoft.com/en-us/library/jj591573.aspx
- https://www.martinfowler.com/bliki/CQRS.html
- https://web.archive.org/web/20150118024058/http://cre8ivethought.com/blog/2009/11/12/cqrs--la-greg-young