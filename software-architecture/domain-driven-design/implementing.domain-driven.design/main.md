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

#### Whiteboard Time
- using the specific domain you currently work in, think of the common terms and actions of the model
- write the terms
- next, write phrases that should be used by your team when you talk about the project

First of all, we want to define a Bounded Context, and its related Ubiquitous Language. The easiest one would be the Resources Context. A Resource is a link to a Web page, augmented by a Title and a URL. A Resource is Added when we want to bookmark a Web page; a Resource is Deleted when we want to delete the link we saved for a Web page; Resources can be Listed to show all saved links. You can Open a Resource clicking to its link on a browser: this will
open the linked Web page on a new tab. Other concerns like managing tags and doing searches can be dealt with in different Bounded Contexts.


## Chapter 2. Domains, Subdomains, and Bounded Contexts

Software systems are always built to help a business achieving its goals in the real world. This means that, of course, the elements of the real world that pertain to those goals are of primary importance when it comes to designing and building those systems. The actions that are taken and the knowledge that is needed to achieve the end goals, and the context in the real world where they take place, are called the *Business Domain*.

Since a Domain can be quite large and complex, it's often divided in *Subdomains*, according to the distinct functional areas and knowledge that can be identified in the Domain.

Among Subdomains, the *Core Domain* identifies the functional area of primary importance for the business: this is the area where the business must excel to allow it to have success, and for these reasons the Core Domain always has the highest priority among all other Subdomains.

*Supporting Subdomains* model other aspects of the business that are essential, but not as much as the Core Domain, and they are also very specialized on a functional area. *Generic Subdomains*, instead, capture generic functionality that is needed, but don't carry any special meaning for the business. Unlike the Core Domain, the business doesn't need to excel
in any of the Supporting or Generic Subdomains, and thus they have lower priority.

Subdomains are to the Bounded Context what the problem space is to the solution space: this means that while the problem space contains all the concepts that are relevant to the business, which is the one that needs to use a software solution to achieve its goals, and thus it's described by Subdomains, the solution space contains the software models that are designed to provide that software solution, and thus they're represented by the Bounded Contexts.

To make an example, let's consider a company that sells products online. The functions that need to be performed range from presenting a catalog of products, to allowing orders to be placed, collecting payments and shipping products. We can thus identify Subdomains based on these functions:
- Product Catalog
- Orders
- Invoicing
- Shipping

In addition to this, imagine that there is also an Inventory, and an External Forecasting. In this situation, there are three separated systems: the e-Commerce System, the Inventory System and the External Forecasting System. Of these, two are internal (part of the same application), and another is external. These systems have been developed in a
very coupled way, meaning that the Subdomains that are part of them really depend on one another, and cannot be easily extracted. For this reason, these systems can also be regarded as separated Bounded Contexts.

However, the e-Commerce Context is really composed of several different Subdomains, as we saw, and this can possibly create problems as soon as those Subdomains need to grow to provide new features, because their different concerns will eventually conflict with one another, being so tightly coupled together.

Additionally, the Forecasting System is performing a very complex task, that probably justifies it being a new Core Domain altogether, meaning that instead of being a Subdomain of the original project, it's a whole different Domain, with its own Subdomains.

In good DDD designs, each Bounded Context tend to fall within a single Subdomain. In the previous example, this clearly isn't the case for the e-Commerce System, since many different Subdomains are located inside the same Bounded Context. The problem with this is that the same terms of an Ubiquitous Language, which is bound to a Bounded Context, are used to mean different business concepts, and this increases confusion.

For example, the term Customer in the Subdomain of the Catalog is related to previous purchases, loyalty, available products, discounts and shipping options; in the Order Subdomain, instead, Customer is a very different concept, since it has to do with shipping and billing addresses, total due, payment terms, etc. This means that there is no one clear
meaning for the term Customer, and thus that no Ubiquitous Language can be defined.

Additionally, if we closely examine the Inventory System Context we may find inconsistencies in the usage of terms there as well. For example, we might be using the term Item to refer to inventory items, but then Item have very different meanings according to the way it's used. An ordered Item that is not yet available is a Back-Ordered Item; an Item being received is called Goods Received; an Item in stock is a Stock Item, etc. According to how these different terms are being described, we might end up needing separated Bounded Contexts inside the *Inventory* itself.

Let's make another example. We have a collaborative system where Users can log in and use a Calendar, a Blog, a Forum, etc. Different Users will have different Permissions on each tool. A naive approach would be to couple every tool, like the Calendar and the Forum, with both the User and the Permission, because after all a Forum has Users, and according to which Permission each User has, it can be allowed to do certain things rather than others.

The problem with this approach, however, is still in the linguistics: the Collaboration Context contains the concepts of Forum, Blog, Discussion, etc., and User. However, Permission is not part of that Ubiquitous Language, because it belongs instead to the Security Context. Every term included in the Ubiquitous Language of the Collaboration Context must be related to the idea of collaboration, and nothing else. The problem here is solved once we realize that the Collaboration Context contains also the concepts of Author and Moderator, that model the idea of allowing different kinds of users to do different things in a way that is natural to the Ubiquitous Language.

Once we realize that we need a separated Security Subdomain, we also discover that those same functions are going to be needed by several other future company projects: decoupling this Subdomain from the other project-specific domains, thus, allows us to reuse that functionality much more widely than expected.

#### Whiteboard Time
- In one column make a list of all the Subdomains that you are aware of in your project. In another column list the Bounded Contexts. Do Subdomains intersect with multiple Bounded Contexts? If so, it's not necessarily a bad thing.

Let's start with the Core Domain. Resources contains the definition of Resource, and maybe the concept of a Resource List, that Resources can be added to and removed from; additionally, a Resource can be added, edited and deleted. Resources doesn't know anything about Tags or anything else, since it must be able to work without depending on other domains.

Tags is a Supporting Subdomain; a Tag has a Name providing context information that can be attached to a Resource; a Resource can have multiple Tags, and the same Tags can be assigned to different Resources; additionally, Tags are related to each other as Parents and Children, where each Tag can have multiple Children Tags, but each Tag can only have one Parent Tag at most. Some Tags can have no Parents at all: those are First-Level Tags. A Tag can be Selected, to show all Resources of that Tag. Resources don't know of the existence of Tags, thus the responsibility of maintaining the relation is on Tags.

Another Supporting Subdomain is Resource Search: a Resource Search operation can be performed to return Resources Results; a Resource Search can be made for Search Keys, so that Resources Results will include Resources containing those Search Keys in the Title or URL.

Tag Search is a Subdomain to provide a Tag Search function to return Tags Results; a Tag Search can be performed for Search Keys, so that Tags Results will include Tags containing those Search Keys in their Name.

In these examples, Bounded Contexts pretty much coincide with Subdomains. Tags is connected with the Resources Core Domain. Resource Search is only connected with Resources, while Tag Search is only connected with Tags. No Bounded Context appears to overlap with two different Subdomains. No Generic Subdomain appears to have been identified.
