---
title: Testability
date: 2021-09-26
---

**Testability** is the ease of determining how well a component satisfies its responsibilities.
A component's testability depends on how easy it is
to **control** the variables that affect the component and
to **observe** the variables affected by it.

<!--more-->

Suppose we want to test some responsibility of a technical component.
The component might be as small as a single, standalone function
or as large as a multi-cluster distributed cache running on multiple machines
in different parts of the world.
Regardless of the component's size,
we test the responsibility by:

1. Establishing relevant preconditions in the component and in its environment.
1. Sending the component a message that triggers it to carry out the responsibility.
1. Comparing the results that the system produced to the results we want.

Step 1 requires us
to _control_ the variables[^variables] that affect the component
as it carries out the responsibility.
Step 3 requires us
to _observe_ the variables affected by the component
as it carries out the responsibilty.

The easier it is to control and observe these variables,
the easier the component is to test.
The harder it is to control and observe these variables,
the harder the component is to test.

> **Testability = Controllability + Observability.**

## Improving Testability
To make a component more testable:
- Identify the variables that affect how the component carries out its responsibilities, and make those variables easier to control.
- Identify the variables affected by the component as it carries out its responsibilities, and make those variables easier to observe.

These steps may seem obvious or simple.
The next section hints at some of the factors
that make these seemingly simple, obvious tasks
less obvious and simple.
In future posts I'll have more to say about these challenges and how to address them.

## Factors that Affect Testability
Any factor that makes key varaibles
harder to control and observe
makes a component less testable.

**Component Complexity.**
The larger and more complex a component,
the more variables are likely to be involved in its responsibilities.
Refactoring a large component into smaller, more focused subcomponents
can greatly reduce the number of variables
that each test must control or observe.

**Coupling.**
The more coupled a component is to other components,
the harder it is to control and observe the variables involved in its responsibilities.
Coupling makes a component depend *indirectly*
on the variables that affect its collaborators.
Testing a highly-coupled component
may require controlling and observing key variables *indirectly*
through the component's collaborators,
the collaborators' collaborators,
the collaborators' collaborators' collaborators,
and so on through many layers of indirection.

**Visibility of Dependencies.**
Coupling is especially challenging when the component's dependencies are *hidden.*
A *hidden dependency* is any dependency
that a component acquires in some way other than through its own interface.
A component has a hidden dependency
on any component it creates or that it retrieves from a hard-coded source,
such as a static method of another class.
Hidden dependencies can make key variables extremely difficult to control or observe.

[^variables]: Note that by _variable_ I mean _anything that can vary._ For an example of how even the simplest software involves a multitude of variables, see [Testing With Variables](https://vimeo.com/34356209), a 20-minute video by Elisabeth Hendrickson and me.
