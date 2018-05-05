## Chapter 1. Getting Started with DDD

The *Ubiquitous Language* is a set of terms and expressions that are developed by the team, and that are used to talk about the domain, and to refer to its concepts, both during discussions, and inside code and documents.

The Ubiquitous Language is neither strictly business related, nor strictly technical, but it's agreed upon by both domain experts and developers. In fact, it wouldn't be possible to define a language strictly over business terms, either, because oftentimes business experts don't agree on the usage of some term.

Instead, the purpose of the Ubiquitous Language is not to define a general standard of communication that should be used throughout the industry; rather, it's very project specific, meaning that it's created for ease the communication within a specific project, and it might as well be discarded afterwards.

Thus, it's not important to decide what is the best term in general to use to indicate a concept, or what is the real meaning of some expression in a broad field of knowledge, but to be as consistent as possible with the usage of the terms within a specific project. Of course, then, the Ubiquitous Language can be improved and refined as the project progresses, yet keeping consistency at every moment.

For example, in a project related to flu vaccines administration, the standard approach (not using any Ubiquitous Language) could be something like:
```java
patient.setShotType(ShotTypes.TYPE_FLU);
patient.setDose(dose);
patient.setNurse(nurse);
```

Classic object-oriented modeling often goes down the path of trying to make every method as generic as possible, to take into account every possible variation and condition, or at least as many of them as possible. Worst, when the code is too complex and some unexpected behaviour is happening, the domain experts are of no help, because they don't understand the code. This is due to several mistakes, like:
- method names do not reveal any real intent, but just describe the implementation detail of what is going on
- method implementations contain hidden complexity, meaning that the complexity is not declared in the method interface
- objects are too often dumb data holders, rather than real objects with meaningful behaviour (anemic objects)

If we spend some time thinking about how we would describe the use case with a precise language, we, with the help of our domain experts, may end up with a use case that sounds like "Nurses administer flu vaccines to patients in standard doses". From here, we can try to design our model using terms taken from the Ubiquitous Language, like:
```java
Vaccine vaccine = vaccines.standardAdultFluDose();
nurse.administerFluVaccine(patient, vaccine);
```

First of all, the interfaces of this example are much more clear in delivering their intent, and can be easily understood even by non-programmers. In addition to this, then, our objects are hardly just data-holders, because no Ubiquitous Language term can be just a data holder, since data holders are low-level technical concepts that find no room in any Ubiquitous Language.

Another reason why it's so important that the Ubiquitous Language be used for all communication and in the actual code is that the language will naturally change and evolve with time, so that it will be very hard to maintain a documentation of it that is always updated. On the other hand, actual daily communication and the code itself will always be up to date.

Let's take a look at a sample code where not much thought is put into using the business concepts in the design:
```java
public void saveCustomer(
	String customerId,
	String customerFirstName, String customerLastName,
	String streetAddress1, String streetAddress2,
	String city, String stateOrProvince,
	String postalCode, String country,
	String homePhone, String mobilePhone,
	String primaryEmailAddress, String secondaryEmailAddress)
{
	Customer customer = customerDao.readCustomer(customerId);

	if (customer == null) {
		customer = new Customer();
		customer.setCustomerId(customerId);
	}
	if (customerFirstName != null) {
		customer.setCustomerLastName(customerLastName);
	}
	if (streetAddress1 != null) {
		customer.setStreetAddress1(streetAddress1);
	}
	if (streetAddress2 != null) {
		customer.setStreetAddress2(streetAddress2);
	}
	if (city != null) {
		customer.setCity(city);
	}
	if (stateOrProvince != null) {
		customer.setStateOrProvince(stateOrProvince);
	}
	if (postalCode != null) {
		customer.setPostalCode(postalCode);
	}
	if (country != null) {
		customer.setCountry(country);
	}
	if (homePhone != null) {
		customer.setHomePhone(homePhone);
	}
	if (mobilePhone != null) {
		customer.setMobilePhone(mobilePhone);
	}
	if (primaryEmailAddress != null) {
		customer.setPrimaryEmailAddress(primaryEmailAddress);
	}
	if (secondaryEmailAddress != null) {
		customer.setSecondaryEmailAddress(secondaryEmailAddress);
	}

	customerDao.saveCustomer(customer);
}
```

The main problem with this code is that it doesn't tackle a very clear and single use case: in fact this method can be called in a dozen different situations, according to how many parameters are defined. Furthermore, checking that it works fine means worrying about all possible situations that may happen. In general, the three errors described above can be found:
- the method name `saveCustomer()` is not descriptive enough (covers too many different use cases)
- its implementation hides a lot of additional complexity
- the domain object, `Customer` is just an anemic data holder.

If instead we think about the possible business language that we might be speaking in this situation, we might try to model the code starting from the use cases, instead of immediately looking for a generic solution that should work in all cases (like the `saveCustomer` method was):
```java
public interface Customer {
	public void changePersonalName(String firstName, String lastName);
	public void postalAddress(PostalAddress postalAddress);
	public void relocateTo(PostalAddress changedPostalAddress);
	public void changeHomeTelephone(Telephone telephone);
	public void disconnectHomeTelephone();
	public void changeMobileTelephone(Telephone telephone);
	public void disconnectMobileTelephone();
	public void primaryEmailAddress(EmailAddress emailAddress);
	public void secondaryEmailAddress(EmailAddress emailAddress);
}
```

Here the `Customer` is not a dump data holder any more, but it reflects the several use cases pertaining to the business concepts. Furthermore, the application service can now be refactored so that it tackle only one use case, like:
```java
public void changeCustomerPersonalName(String customerId, String customerFirstName, String customerLastName)
{
	Customer customer = customerRepository.customerOfId(customerId);
	if (customer == null) {
		throw new IllegalStateException("Customer does not exist.");
	}
	customer.changePersonalName(customerFirstName, customerLastName);
}
```

Of course this code can be understood and verified for correctness much better.

Let's see another example:
```java
public class BacklogItem extends Entity
{
	private SprintId sprintId;
	private BacklogItemStatusType status;
	...

	public void setSprintId(SprintId sprintId)
	{
		this.sprintId = sprintId;
	}

	public void setStatus(BacklogItemStatusType status)
	{
		this.status = status;
	}
	...
}
```

```java
// client commits the backlog item to a sprint
// by setting its sprintId and status
backlogItem.setSprintId(sprintId);
backlogItem.setStatus(BacklogItemStatusType.COMMITTED);
```

Here the domain model just exposes its technical implementation, instead of capturing the business use cases and concepts. A better version of this code, expressing the Ubiquitous Language, could be:
```java
public class BacklogItem extends Entity
{
	private SprintId sprintId;
	private BacklogItemStatusType status;
	...

	public void commitTo(Sprint aSprint)
	{
		if (!this.isScheduledForRelease()) {
			throw new IllegalStateException("Must be scheduled for release to commit to a sprint.");
		}

		if (this.isCommittedToSprint()) {
			if (!aSprint.sprintId().equals(this.sprintId())) {
				this.uncommitFromSprint();
			}
		}

		this.elevateStatusWith(BacklogItemStatusType.COMMITTED);
		this.setSprintId(aSprint.sprintId());

		DomainEventPublisher
			.instance()
			.publish(new BacklogItemCommitted(this.tenant(), this.backlogItemId(), this.sprintId()));
	}
}
```

```java
// client commits the backlog item to a sprint
// by using a domain-specific behavior
backlogItem.commitTo(sprint);
```

In the second example we abide by the "tell, don't ask" principle, letting the domain model take the responsibility to know how the proper wiring needs to be done, instead of forcing the client to know the details.

A *Bounded Context* is the smallest software unit that can capture the complete Ubiquitous Language for a given business domain, meaning that once the Ubiquitous Language has been defined, also the Bounded Context scope is set. A Bounded Context integrates with other Bounded Contexts through *Context Maps*, and the other Bounded Context have their own Ubiquitous Language, that is generally different from the one of the first (although some overlap might happen).

---

#### Whiteboard Time
- using the specific domain you currently work in, think of the common terms and actions of the model
- write the terms
- next, write phrases that should be used by your team when you talk about the project

The example domain to be used as a study case is "studying documents", which would be the activity of a software developer improving his or her skills with personal study. Studying documents implies reading books and articles, taking notes about the documents read (which could themselves become other documents too), work on exercises and projects, and in general mastering effective study techniques. These activities employ several different tools, many of which are already available and effective, like browsers and PDF readers to read documents, text editors and development tools to take notes and work on exercises and projects, and cloud repositories like GitHub to store notes and projects.

What we'd like to improve with respect to the current available tools, is a way to manage bookmarks of documents. A new tool to manage bookmarks will be our project. In this context, a Bookmark represents the bookmark of a Web page, or a book, that can be accessed through a browser (thus including local PDF files opened through a local URL). Bookmarks have a Link (which is the URL used to access them from within a browser), and a Title. A Bookmark is Added when a bookmark for the Web page or book is created and stored in the Bookmarks List; a Bookmark is Deleted when it is removed form the Bookmarks List; Bookmarks can be Listed when the Bookmarks List is displayed, showing all saved bookmarks: Bookmarks Listing can be enhanced in various ways, like with Pagination, or Filters; a Bookmark can be Opened navigating to its Link in a browser: this will open the linked Web page or book in a new browser tab.

In addition to this, other features will be helpful to bookmark documents, like managing tags and performing searches, but these will be further explored later on.

---


## Chapter 2. Domains, Subdomains, and Bounded Contexts

Software systems are built to help a business achieving its goals in the real world. Of course, then, the elements of the real world that pertain to those goals are of primary importance when it comes to designing and building those systems. The actions that are taken and the knowledge that is needed to achieve the end goals, and the context in the real world where they take place, are called the *Business Domain*.

To build a sofware system that is able to support a Domain, it's necessary to capture the characteristics of the Domain that are meaningful for the software in a *Domain Model*. Usually the whole Domain of an organization is extremely complex, and made of several diverse parts, so that it's impractical, and ultimately pointless, to try to create an all-encompassing Model of it. The actual point of Domain-Driven Design is, instead, to understand what are the different areas composing the Domain, and focus on each one separately. These areas are called *Subdomains*, and are identified by the distinct functional areas and knowledge that can be identified in the Domain. Models will then be created for specific Subdomains, rather than for the whole Domain.

Among Subdomains, the *Core Domain* identifies the functional area of primary importance for the business: this is the area where the business must excel to allow it to have success, and for these reasons the Core Domain always has the highest priority among all other Subdomains.

*Supporting Subdomains* model other aspects of the business that are essential, but not as much as the Core Domain, and they are also very specialized on a functional area. *Generic Subdomains*, instead, capture generic functionality that is needed, but don't carry any special meaning for the business. Unlike the Core Domain, the business doesn't need to excel in any of the Supporting or Generic Subdomains, and thus they have lower priority.

Once the Domain Model of a Subdomain has been defined, it can be implemented in software elements: these constitute a *Bounded Context*. Subdomains are to Bounded Contexts what the problem space is to the solution space: this means that while the problem space contains all the concepts that are relevant to the business, which is the one that needs to use a software solution to achieve its goals, and thus it's described by Subdomains, the solution space contains the software models that are designed to provide that software solution, and thus they're represented by Bounded Contexts. Bounded Contexts are "bounded" because they work with specific Ubiquitous Languages, and the terms defined in the Ubiquitous Language used by a Bounded Context are never used, at least with that meaning, outside of that Bounded Context. This makes Bounded Contexts actually "linguistic boundaries".

A common mistake made while working with complex software systems, is trying to create an all-inclusive model of the whole system, and find an agreement on what is the unique meaning of each term in that model, regardless of the differences between each specific Subdomain. The problem with this approach is that, even if a formal agreement is reached, experts of each Subdomain will actually continue to use those terms with the real different meanings they have in each different Subdomain. Additionally, the meaning of each term is likely to change with time, and in different ways in each different context.

Oftentimes Bounded Contexts don't overlap precisely with Subdomains: instead, some Bounded Context may implement several Domain Models, and some Domain Model implementation may span multiple different Bounded Contexts. This is not an ideal situation from a DDD perspective, because fewer Bounded Contexts are used to provide many business functions, and as such multiple concerns will be conflicting within the same implementation: starting from this situation, when any Model will need to grow to support new features of its corresponding Subdomain, also the implementations of the other unrelated Models that sit in the same Bounded Context will likely need to be changed, and this may be impossible or overly expensive, with respect with the correct situation where concerns belonging to different Models are properly separated in different Bounded Contexts.

To make an example, let's consider a company that sells products online. The functions that need to be performed range from presenting a catalog of products, to allowing orders to be placed, collecting payments and shipping products. We can thus identify Subdomains based on these functions:
- Product Catalog
- Orders
- Invoicing
- Shipping

In addition to this, imagine that there is also an Inventory, and an External Forecasting system. In this situation, there are three separated systems (Bounded Contexts): the e-Commerce System, the Inventory System and the External Forecasting System. Of these, two are internal (part of the same application), and another is external. These systems have been developed in a very coupled way, meaning that the Subdomains that are part of them really depend on one another, and cannot be easily extracted.

However, the e-Commerce Context is really composed of several different Subdomains, as we saw, and this can possibly create problems as soon as those Subdomains need to grow to provide new features, because their different concerns will eventually conflict with one another, being so tightly coupled together.

Additionally, the Forecasting System is performing a very complex task, that probably justifies it being a new Core Domain altogether, meaning that instead of being a Subdomain of the original project, it's a whole different Domain, with its own Subdomains.

In good DDD designs, each Bounded Context tend to fall within a single Subdomain. In the previous example, this clearly isn't the case for the e-Commerce System, since many different Subdomains are located inside the same Bounded Context. The problem with this is that the same terms of an Ubiquitous Language, which should be bound to a specific Bounded Context, are used to mean different business concepts, and this increases confusion.

For example, the term Customer in the Catalog Subdomain is related to previous purchases, loyalty, available products, discountsl, shipping options, etc.; in the Order Subdomain, instead, Customer is a very different concept, since it has to do with shipping and billing addresses, total due, payment terms, etc. This means that there is no one clear meaning for the term Customer, and thus that an Ubiquitous Language hasn't really been defined.

Additionally, if we closely examine the Inventory System Context we may find inconsistencies in the usage of terms there as well. For example, we might be using the term Item to refer to inventory items, but then Item have very different meanings according to the way it's used. An ordered Item that is not yet available is a Back-Ordered Item, an Item being received is called Goods Received, an Item in stock is a Stock Item, etc. According to how these different terms are being described, we might end up needing separated Bounded Contexts inside the *Inventory* itself.

Let's make another example. We have a collaborative system where Users can log in and use a Calendar, a Blog, a Forum, etc. Different Users will have different Permissions on each tool. A naive approach would be to couple every tool, like the Calendar and the Forum, with both the User and the Permission, because after all a Forum has Users, and according to which Permission each User has, it can be allowed to do certain things rather than others.

The problem with this approach, however, is still in the linguistics: the Collaboration Context contains the concepts of Forum, Blog, Discussion, etc., and User. However, Permission is not part of that Ubiquitous Language, because it belongs instead to the Security Context. Every term included in the Ubiquitous Language of the Collaboration Context must be related to the idea of collaboration, and nothing else. The problem here is solved once we realize that the Collaboration Context contains also the concepts of Author and Moderator, that model the idea of allowing different kinds of users to do different things in a way that is natural to the Ubiquitous Language.

Once we realize that we need a separated Security Subdomain, we also discover that those same functions are going to be needed by several other future company projects: decoupling this Subdomain from the other project-specific domains, thus, allows us to reuse that functionality much more widely than expected.

Bounded Contexts are useful not only to implement Domain Models, but also to isolate external systems, like other applications or services. Additionally, a Bounded Context can contain much more than just the Model: for example it may contain a database schema, if the names of tables and columns are those of the Ubiquitous Language. Also the user interface may be part of the Bounded Context, if the model needs to be rendered and to interact with the user (although the Domain still resides in the Model, and not in the interface, which is just a tool); in addition to this, the Bounded Context might be exposing services, like REST or SOAP; both user interface components and service endpoints will delegate to application services, which are also inside the Bounded Context.

---

#### Whiteboard Time
- In one column make a list of all the Subdomains that you are aware of in your project. In another column list the Bounded Contexts. Do Subdomains intersect with multiple Bounded Contexts? If so, it's not necessarily a bad thing.

In our "studying documents" domain we can highlight the following Subdomains:
- Bookmarks: catalog of Bookmarks to Documents and functionality to manage them, as well as to easily search and read them.
- Notes: repository of Notes and ways to read and write them.
- Projects: repository of Projects and exercises, and tools to work on them.
- Planner: tool to create study plans, featuring tracking Documents read, notes and Projects related to each Document, future work, etc.

The Core Domain is indeed Bookmarks, since it wraps the main activity related to studying documents, without which the job simply cannot be done, and such that the job could still be done, albeit less effectively, if only that Domain was available. All others are Supporting Subdomains.

Not all these Subdomains need a software system dedicated to them to be created brand new, since they can be effectively supported by already existing systems:
- Browsers: existing, to open Bookmarks and Notes
- Bookmarks: to be created, to manage Bookmarks
- Text editors and development tools: existing, to write Notes and work on Projects
- Git and GitHub: existing, to manage Notes and Projects
- Planner: to be created

The main project to be working on, the Bookmarks Context, represents the implementation of a Model of the Bookmarks Subdomain. As far as how much we know by now, no other Subdomain will be included in the software project. Another software project that can be considered is the Planner: this would be a separate Bounded Context that will need to collaborate with all other Subdomains.

Since we already know that Bookmarks will need additional features in the future, we could, as an exercise, define additional Bounded Contexts to relate to Bookmarks, containing the additional features:
- Bookmarks:
    - Bookmark
    - Bookmark List
    - Add Bookmark
    - Remove Bookmark
    - Edit Bookmark
    - Show Bookmark
    - List Bookmarks
- Tags
    - Tag
    - Parent Tag
    - Children Tags
    - Children Bookmarks
    - Create Tag
    - Delete Tag
    - Edit Tag
    - Show Tag
    - Change Parent Tag
    - List Children Tags
    - List Children Bookmarks
- Search
    - Result Bookmarks
    - Result Tags
    - Search Bookmarks by Keyword
    - Search Tags by Keyword

The main Bounded Context, Bookmarks, is independent from everything else. Tags depends on Bookmarks, and it adds to it features to manage a Tag tree, where each Tag is associated to multiple Bookmarks. Search Context depends on both Bookmarks Context and Tags Context, since it provides search functionality to both.

---

#### Whiteboard Time
- See if you can identify some subtly different concepts that exist in multiple Bounded Contexts in your Domain.

In the previous example, we can see how the Bookmarks concept appears in all three Bounded Contexts. However, in Bookmarks it represents a container of all the information of a bookmark, in Tags it represents the children of Tags, and in Search it represents the result of a certain type of search. Something similar can be said for the concept of Tag, which appears in both Tags and Search.

---

It's helpful to think of Bounded Contexts in terms of the actual technical components that will implement them. For example, a Bounded Context will usually fit a single source code project: an entire software project composed of many different Bounded Contexts will thus contain several distinct code projects, that might well be deployed to different systems.


## Chapter 4. Architecture


### Layers Architecture

The Layers Architecture is a classic architectural pattern introducing the distinction between Domain Model, Infrastructure, User Interface and Application.

The User Interface only deals with user input and output, meaning how to present data to the user, and how to gather user requests; it takes no business nor application decisions, and for this reason it usually shouldn't directly use objects from the Domain Model, which may expose the ability to take decisions, but Data Transfer Objects instead, containing only data of the Domain Model that needs to be presented.

The Application Layer contains services that are able to take decisions about how the application flow should go, but they know nothing about business logic. For example, they might deal with persistence transactions and security, or with sending notifications to other systems or composing emails to send to users. The Application Layer services are used to express use cases or user stories, for example taking requests from the User Interface and calling the Domain Model to perform the actual business logic requested, and then sending the results back to the User Interface perhaps: as such, Application services are usually simple and thin.

The Domain Model is where the definition of all the business concepts resides, as well as where all the business logic is implemented, and as such can be regarded as the core of the logic of the system that is valuable for the domain.

The Infrastructure contains actual implementations of persistence or messaging mechanisms, and other concrete components and frameworks to provide low-level services.

Each of these parts of the system is cohesive, meaning that all its components are conceptually linked to each other, and as decoupled as possible from each other, to prevent responsibilities that don't belong to certain part to leak into it. Parts that are cohesive and decoupled are thus called Layers, to enforce the concept of them being well distinct from one another, and at the same time containing components that belong together. The second important characteristic of Layers is that they are ordered by means of a dependency relation. Layers are usually laid out in a top-bottom fashion, and this is another reason why they're called Layers, where each layer only depends on the layers below. The topmost layer (thus with the highest number of dependencies) is the User Interface Layer, followed by Application, Domain Model and Infrastructure. The Infrastructure Layer is thus completely independent from all others, and as such contains elements that can be freely used in other different contexts and systems.

There's some variations with the dependency rule in Layers Architecture: Strict Layers Architecture allows a layer to couple only to itself and the layer immediately below; Relaxed Layers Architecture allows a layer to couple to any other layer standing below. This second interpretation is the most employed since it's often the case that many upper layers need to access the Infrastructure.

Sometimes it's required that a lower layer uses an element of a higher layer. This of course cannot be done directly, because it would mean that the lower layer knows of the existence of the higher one. The two most typical ways of getting around this are using the Observer or Mediator patterns. Both are based on the idea that the lower layer defines an interface for the element it needs, and the higher layer wraps its element into that interface, and pass the wrapped element to the lower layer, which thus is maintained oblivious of the existence of the other layer.

The main problem of the traditional definition of Layers Architecture is that some high level layers depend on low level ones, namely the Domain Layer, which should be pretty abstract in concept, is bound to know low level details contained in the Infrastructure Layer. This breaks the Dependency Inversion Principle, stating that concrete modules should depend on abstract ones, and never the other way around. The solution to this is to let the Infrastructure Layer depend on abstractions defined in higher layers. This in fact causes a complete reordering of the layers, where the Infrastructure Layer, which previously was at the bottom, not depending on anything else, now is at the top, depending on interfaces defined in possibly all other layers, and the Domain Layer is at the bottom, not knowing about any other layer. Additionally, now the other layers don't know about the Infrastructure Layer anymore either, because the Infrastructure Layer is implementing interfaces defined in the other layers, so the direction of those dependencies have been inverted. Consequently, the Application Layer only depends on the Domain Layer, and the User Interface Layer only depends on the Application and Domain Layers.


### Hexagonal Architecture

The natural consequence of this approach is Hexagonal Architecture, which leverages the Dependency Inversion Principle applied to Layers Architecture to allow any number of different clients to interact with the application, without it needing to be modified at all to support them. In the same way, output components like graphics and persistence can as well be easily swapped. Hexagonal Architecture changes the traditional approach of looking at an application's communication layout.

Classic applications have a "front-end", where interactive clients are located, and a "back-end", where persistent data is stored. Hexagonal Architecture, on the other way, sees an application as having an "outside" and an "inside": the outside contains both input mechanisms provided by clients, and output mechanisms to access persisted data, whereas the inside contains the business and application models. Inner modules interfaces are called "ports" while outer implementations of those interfaces provided by user interface and infrastructural components are called "adapters". What previously was front-end and back-end now are called primary actors (those using the application) and secondary actors (those the appication uses).

While employing Hexagonal Architecture, the Application Layer is focused on use cases, which will be the same for each different client, since the application is oblivious of which and how many clients will connect. Additionally, one port is defined for each use case, and many adapters can attach to the same port, thus using the same use case. This means that the number of ports of an application is usually fairly small, and that each port has to support many functionality. To avoid breaking the SRP, a common solution is using the command pattern, so that the application defines the commands published by a port, and their handlers, while the adapters properly issue those commands. A similar result would be obtained using message queues, for example. This also makes clear that an adapter can run as a different process than the application.


### REST

The REST architectural style can be used to design Web adapters. A common strategy in this case is to separate a REST user interface into its own Bounded Context: this makes sense because the way a user interacts with a REST service has its own rules and concepts, that are completely different from those of the Core Domain. In other words, a REST interface has its own Ubiquitous Language and use cases, unrelated to the ones of the main Bounded Context we are developing. This is also why Domain Models shouldn't be directly mapped to REST resources, because the use cases for the use of these resources and models could possibly be very different.


### CQRS

Applying CQRS, in the command model aggregates won't have any query method, but only command methods, while repositories will have only a simple `add()` or `save()` method, and only a single query method to fetch an aggregate based on its ID. To get data to display to the user we would use a query model.

The client will use a set of query processors to get data from the application, which in turn use the query store. The read side is very simple, it's not even necessary to pack the read data into DTOs in many cases, but often the database could be queried directly.

The query model is a denormalized data model. For example, if the data model is an SQL database, each table would contain data for a single view. In case of complex data sets, joins could be needed.

On the write side, the user interface client will send commands to the write model, which will in turn use the domain model through command handlers, with its aggregates and repositories, to perform the assigned task. What command handlers will do can be creating a new aggregate instance and adding it to a repository, or getting an aggregate instance from the repository and calling some method on it.

When the command handler completes, a single aggregate will have been used, and a domain event will have been published. If other aggregates need to be updated as consequence of this command, this would not be done directly from the command handler, which knows nothing about other handlers, but as a reaction to the published event. Also the query model will be updated as a reaction to the event being published.

---

#### Whiteboard Time

We learned that we should separate complex user interfaces into their own bounded contexts. Considering the Bookmarks Context, it will have a service context, Bookmarks Service, which will provide the actual functionality independent of user interfaces, storage mechanisms or other external entities. Additionally, it will have an API interface, which will turn into a Bookmarks API Context; the purpose of it would be to expose the data of Bookmarks to be consumed by automated systems. For example, a Web Interface may want to display a subset of Bookmarks, a "page": to do this, it would reach for the `/bookmarks` API endpoint using the `GET` method, and passing as query arguments something like `firstBookmark=12&lastBookmark=22`, and it would get a JSON response containing all required Bookmarks' data, which it will then parse and display. Other than this, it will also have a graphic Web interface, to present data in a way directly consumable by human users.

The central point here is that the use cases for Bookmarks API are likely different than those of Bookmarks Service, or Bookmarks Web contexts. For example, the concept of a "page" of results would make sense only in the Web context, because the GUI will need to paginate the results to be more user-friendly; the API context will probably have a more generic concept of "filters" to be applied to the query, that used in the proper way will return "pages" of results; finally, the Bookmarks Service context won't probably be concerned with filters at all, since the read side of the system could be exposing just an SQL database, from which data is retrieved.

Thus, the API Context will have the "get filtered bookmarks" use case, that other contexts won't have. As a more simple example, just retrieving a single bookmark will be a different use case in the two user interfaces: from a Web interface perspective, retrieving a bookmark means printing some kind of graphical widget containing the bookmark link the user can click on to open the connected document: this assuming that no "single bookmark" page is designed to be provided by the Web interface; on the other hand, retrieving a bookmark in the API Context means to get a JSON response containing only the data of just that single bookmark.

Finally, the Bookmarks Service, containing the business logic, has no use case for either getting a list of bookmarks, or for getting a single one, because it is directly exposing the data from the read side, for clients to consume.

---

Separating the write side from the read side, the problem of data consistency arises. If the update of the read side is done synchronously with write operations, the query and command model would update their databases (which could also be the same one) at the same time, or even in the same transaction: this way both models are always completely consistent. This, however, would be less performant because many tables would need to be updated, and this would make queries slower if the system is under heavy load. In this sitations, it'd better to update the query model asynchronously, leading to a situation of eventual consistency, where the user interface won't show the latest updates which have just been submitted to the command model.


### Event-Driven Architecture

A system composed of different Bounded Contexts talking to each other, could decide to use Events as a communication mechanism. There are several kinds of events that can be published, of which the most important are Domain Events. There are several mechanisms to publish events, such as HTTP or AMPQ, but the important thing is that events are used, independently of the dispatching mechanism. Then, each Bounded Context producing and consuming events might implement the Hexagonal Architecture.

Each Bounded Context would receive all Events, of which only certain have meanings, and would trigger a reaction. It's often the case that a specific task that must be performed by a component requires several steps, each of which is triggered by a different Event or Message. This means that certain techniques to design systems which communicate through messages have to be employed.

There are several techniques that can be employed to build message-based systems. *Pipes and filters* is used when a system has a chain of components where the output of one is used as input of another: these components are called *filters*, because they take an input, and change it before outputting the result again; on the other hand, *pipes* are the channels where messages flow, connecting the various filters. In a broader sense, a filter might not change the input at all, but just perform some processing on the basis of it. Filters are designed to be independent from one another, and their interface is such that they can be interchanged, and recombined in diverse fashions. Filters might read from, and write to, multiple pipes, allowing parallel or concurrent processing. The same filters might be deployed in multiple copies, to increase the throughput in case of slow processing, or high traffic. In an event-driven architecture, filters read input from the events they listen to, and write output sending events containing output data: this way, events constitute a common interface that is shared by all filters, and allow to add new filters, replace or rearrange them at any time.

From a DDD perspective, the events used to communicate would be Domain Events, which are named accordingly to specific business concepts, and they model specific business activities, so they are indeed not just technical tools. Domain Events have a unique identity and contain as much information as possible.

Another event-driven processing pattern is called *Long-Running Processes*, or *Sagas*. Here, starting from a single sequential pipeline, as usual with Pipes and Filters, we add some new pipelines that listen to the same event published by the first data-reader component, and then go on processing the data in a parallel fashion. With this setup it's also necessary that a specific component acts as a manager, understanding when all pipelines are come to completion, gathering all the output and doing some final processing to it.

Event Sourcing, instead, is based on the idea of apply something similar to version control to the entities and aggregates of the model, meaning recording everything that happened to any given aggregate throughout time. The way this is done, is having the aggregate send events every time anything happens: these events are then stored in a proper event store, that actually acts as a version control system. To be more precise, every operational command executed on any aggregate, will lead to a specific Domain Event to be published, describing the execution outcome. These events are saved in the event store in the order they occurred. Having this event store in place, it will be possible to recover any previous version of the state of the aggregate just playing back the recorder events up to the desired point in time.


## Chapter 5. Entities

An *Entity* is a software element that models business concepts having an identity. Since an Entity has an identity, it also has an history, which is the set of changes that were applied to the software element having the same identity. The alternative to Entities are *Value Objects* which have no identity, and thus are immutable and have no history, exactly as the classic definition of what a *value* is.

The definition of an Entity is all about its identity: this means that the only goal of the definition of a class that represents an Entity, is to define its identity and life cycle continuity. When defining a new Entity, we focus only on the primary attributes and behaviors characterizing its identity. Secondary attributes, not related to the identity, may be added only after the identity has been clearly defined.

Entities that are contained in an Aggregate may have also a *local identity*, distinguishing them from the other Entities which are located in the same Aggregate. Only the Aggregate Root requires global uniqueness.

The identity of an Entity may not be limited to a single value, like a UUID: if deemed practical, developers might decide to craft an unique ID containing also the date when that ID has been generated, to make it easier to reason about the values. This way, the identity would be stored in a Value Object instead of a raw string, so that also the additional information about the identity can be exposed, like in `productId.creationDate()`. Another advantage of modeling an ID with a Value Object is static analysis: if multiple different kinds of IDs are employed in the Bounded Context, it's useful that each has its own type, to avoid passing the wrong IDs to methods. The component where the identity value is generated can be for example the Aggregate Root Repository, like in `productRepository.nextIdentity()`.
