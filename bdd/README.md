From the article [what's in a story](https://dannorth.net/whats-in-a-story/):

*Behaviour-driven development* approaches the design of software systems from the outside-in: first identifying the desired business outcomes, and only afterwards taking into account the technical aspects that are needed to create those solutions.

The business outcomes are a set of features, with acceptance criteria. These are defined in terms of "stories", characterized by one or more subjects performing actions in certain conditions, and producing results.

The purpose of this is to define a systematic way to express business requirements, which are often too broad or imprecise to be directly translated to technical specifications. In addition to help the technical design, stories also help business stakeholders understanding each other perspectives about the features.

This is a common template used for writing stories:

```
Title (one line describing the story)

Narrative:
As a [role]
I want [feature]
So that [benefit]

Acceptance Criteria: (presented as Scenarios)

Scenario 1: Title
Given [context]
And [some more context...]
When [event]
Then [outcome]
And [another outcome...]

Scenario 2: ...
```

Stories are not to be intended as definitive system requirements, rather they are a formal description of the stakeholders' wishes. After writing stories, it's possible to make estimates of the costs of implementing them (or do some research work to understand it beforehand), and possibly decide that certain stories are not worth implementing, or should be changed.

Consider the following example:

```
Story: Account Holder withdraws cash

As an Account Holder
I want to withdraw cash from an ATM
So that I can get money when the bank is closed

Scenario 1: Account has sufficient funds
Given the account balance is $100
And the card is valid
And the machine contains enough money
When the Account Holder requests $20
Then the ATM should dispense $20
And the account balance should be $80
And the card should be returned

Scenario 2: Account has insufficient funds
Given the account balance is $10
And the card is valid
And the machine contains enough money
When the Account Holder requests $20
Then the ATM should not dispense any money
And the ATM should say there are insufficient funds
And the account balance should be $10
And the card should be returned

Scenario 3: Card has been disabled
Given the card is disabled
When the Account Holder requests $20
Then the ATM should retain the card
And the ATM should say the card has been retained

Scenario 4: The ATM has insufficient funds
...
```

Each story defines a specific feature of our system: as such, the story's title should clearly an concisely describe the feature. Furthermore, since everything revolves around roles and actions, the title should describe an action performed by a role, like in the previous example "Account Holder withdraws cash", rather than a generic topic like "Account management" or "ATM behavior", because these might encompass multiple features, it will be harder to understand when the story is complete, and the edge cases will be harder to treat.

Stories relate features to outcomes. In the previous example, the feature is withdrawing cash, and the outcome is being able to get money when the bank is closed. It may happen that, will developing the scenarios of a story, the feature being described doesn't actually produce the expected outcome. This means two things: first, that the current feature does not provide the expected benefit, and thus might just be unnecessary to implement; second, there's an hidden story we didn't write yet: one describing a feature we don't know yet, that will actually provide the outcome that the current story wasn't able to provide.

In most cases scenarios revolve around a single event, and thus they should map to a single call to the production code. However, it may be needed sometimes to use more complicated scenarios, like sequences of events. A part from these cases, however, usually a specific story talks about a single event, and all scenarios of that story differ only in the context (givens) and outcomes.

You shouldn't use more givens than those required, because it might be distracting, nor less than necessary, otherwise more than one outcome will be possible with the given givens.

Stories should be small enough to fit a single iteration: if too many scenarios starts to crop up, that's a sign that the story should be split into smaller stories, grouping related scenarios together. To achieve this, it's better to give each story only one degree of freedom. In the ATM example, we really have three moving parts: the account balance, the card status and the ATM state. It will probably turn out to be better to have three separated stories instead:
- Account Holder withdraws cash (ATM working and card valid)
- Account Holder withdraws cash with invalid card (ATM working)
- Account Holder withdraws cash from flaky ATM (card valid)


From https://github.com/cucumber/cucumber/wiki/Cucumber-Backgrounder

One upside of writing stories and scenarios is the fact that they can be easily converted to automated acceptance tests. If scenarios' steps constitute the high-level definition of a feature, steps implementations constitute the acceptance tests.

Scenarios should be written in a declarative way, rather than imperative. These usually means to refrain from listing details, which will be described in the steps implementation anyway, and just stick to the high level meaning of the steps, those that are valuable to the stakeholders.

Another way to state this, is that you should describe the result you want to achieve, rather than the details of how you currently expect to get it. For example:

```
Given I visit "/login"
When I enter "Bob" in the "user name" field
And I enter "tester" in the "password" field
And I press the "login" button
Then I should see the "welcome" page
```

This is an imperative description of the sequence of steps needed to perform the login action, and as such is filled with implementation details. Other than pushing the feature description far from what is of value, and understandable, for stakeholders, it also strongly couples a feature to its implementation: if for some reason in the future the implementation needs to be changed, than also all stories depending on it must be changed as well. A much better definition would be:

```
When "Bob" logs in
```

Thus, when writing features it's worth to ask ourselves what happens to the definition if the implementation of a feature changes.

It's important to write steps implementations as soon as their definitions: this way, writing the bare minimum implementation required to satisfy the requirement, we understand what other feature scenarios are necessary. On the other hand, writing more complicated implementations than needed will result in having code not covered by any scenario, and code that will never be used.


From http://www.thinkcode.se/blog/2016/06/22/cucumber-antipatterns

As in TDD, also BDD requires you to write features definitions before the actual code, to let features drive the application's design. This way what needs to be done is clear to every party involved, before any actual work (that could be then thrown away) is done.

Scenarios need to be written by all involved parties together, because each one has a different perspective on the problem, and would end up writing a biased scenario, that doesn't meet the expectations of the others. In particular, if scenarios are only written by business people, they'll end up being un-testable, thus requiring developers to change them: however, in the process the original meaning captured by the scenarios could be lost. On the other hand, if developers only are writing scenario, they won't be thinking about valuable use cases with real people and users.
