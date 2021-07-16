---
title: Tacit Dependencies Inhibit Testing
draft: true
---

Code becomes difficult to test
when it depends on collaborators
that are hard to control and observe.
Tacit dependencies are hard to control and observe.
To make a class easier to test,
make its dependencies explicit.
<!--more-->




## Testability

Suppose we want to test an instance of some Java class
(Geode's `PartitionedRegion` class
[for example](#testing-partitionedregion-virtualput)).
To do that, we:
1. Establish any relevant preconditions
    in the object,
    in the object's collaborators,
    or elsewhere in the object's environment.
1. Send the object a message
    that triggers it to carry out
    the responsibility we're testing.
1. Observe the results
    that the object produced
    and compare them to the planned results.

Step 1 requires us to
_control_ the variables
that affect how the object carries out its responsibilities.
Step 3 requires us to
_observe_ the variables
affected by the object as it carries out its responsibilities.
(Note that by _variable_ I mean
_anything that can vary[^variables]._)

> **Testability = controllability + observability.**

The easier it is to control and observe
the variables involved in an object's responsibilities,
the easier the object is to test.
The harder it is to control and observe those variables,
the harder the object is to test.




## Tacit Dependencies

The _tacit dependencies_ of an object
are the objects and other resources
that it interacts with
that are _not mentioned explicitly_
in the object's public interface.

Tacit dependencies include[^other-ways]
any collaborators that the object:

-   Creates via `new`.
-   Obtains from a static source.




## How Tacit Dependencies Inhibit Testing

Tacit dependencies make an object harder to test
by inhibiting tests' ability to:

-   Control the conditions
    that affect the responsibility being tested.
-   Observe the results
    that the object produced.

### How Tacit Dependencies Inhibit Control

If the responsibiilty we're testing
depends on how a collaborator responds,
the test must control the collaborator
to make it respond in a particular way.
Further, if the collaborator's response
depends on its state,
the test must control the collaborator's state.

Classes that create tacit dependencies
often assign them to private fields.
Or a method might create a collaborator,
interact with it,
and discard it.
Tests cannot control these private dependencies directly,
and may not be able to control them even indirectly.
Even if a class exposes a tacit dependency
through a getter,
the complexity of a real, production-ready collaborator
can be difficult for a test to control.

Each of an object's tacit collaborators
essentially becomes _part of the object's state._
If a responsibility
depends on the state of the object,
the inaccessibility and complexity of these collaborators
make it very difficult
to write a test
that puts the object in an appropriate state.

### How Tacit Dependencies Inhibit Observation

When testing an object,
a test might want to make two kinds of observations
about the object's collaborators:

- What messages the object sent to the collaborator.
- What state the collaborator is in.

Tacit dependencies inhibit these observations.

If the object keeps the collaborator private,
the test has no way to observe its state directly.
The only way to observe
whether the collaborator ended up in the correct state
is by _inferring it_
from how it influences the object under test's
subsequent behavior.
This sort of inference
always depends on the specific implementation of the collaborator.

Even if the object under test
exposes a collaborator via a getter,
the collaborator itself may be complex enough
that its state is difficult to observe directly.

The essential responsibility of the object under test
is _not_ to put its collaborators into the proper state.
It is to _send the right messages_ to its collaborators.
It is the collaborators' responsibility
to get themselves into the correct state.

### Transitive Tacit Dependencies

If an object tacitly depends on collaborators,
and those collaborators tacitly depend on yet more collaborators,
our object becomes increasingly difficult to test.

## To Make a Class More Testable, Make its Dependencies Directly Controllable and Observable

TODO


## Examples

### Testing PartitionedRegion's virtualPut() method

**A responsibility.**
The `PartitionedRegion` class's `virtualPut()` method
has the responsibility
to send a predefined message
to a predefined destination
so that the system
remembers the new value for the key.

The choice of destination for each message
depends on the state of the partitioned region:
If the region has a data store,
the destination is the data store.
Otherwise the destination is the cluster member
that hosts the primary bucket for the entry.

The choice of message to send
depends on the parameters passed to `virtualPut()`:
If `isNew` is `true`,
the region must send a _create_ message to the destination.
Otherwise it must send a _put_ message.

Here is a summary of the responsibility:

Has data store? | `isNew` | Response
-|-|-|
yes | true | send a "create" message to the data store
yes | false | send a "put" message to the data store
no | true | send a "create" message to the member that hosts the primary bucket
no | false | send a "put" message to the member that hosts the primary bucket

**A test.**
Consider a test for for the first row of that table:
- Conditions:
    The partitioned region has a data store.
- Inputs:
    `ifNew` is `true`.
- Response:
    The partitioned region
    must call the data store's `createLocally()` method.

To test this responsibility,
the test must:
- **control** whether the partitioned region has a data store.
- **observe** what message the partitioned region sent, and to what destination.

### Tacit Dependencies in PartitionedRegion

`PartitionedRegion` creates dozens of tacit dependencies
some in its constructor,
some during subsequent initialization,
and yet others in methods
called later.

Here are two tacit dependencies
that inhibit the test we want to write.

The constructor creates a `PRHARedundancyProvider`
[by calling `new`](https://github.com/apache/geode/blob/0ea005d5d7d1deb5ebe9639b34b0294af577b51d/geode-core/src/main/java/org/apache/geode/internal/cache/PartitionedRegion.java#L816):

```java
this.redundancyProvider = new PRHARedundancyProvider(this, cache.getInternalResourceManager());
```

This makes `PartitionedRegion`
tacitly depend on a _specific implementation_ of redundancy provider
and, transitively,
on every tacit dependency of that specific implementation.

TODO: List the tree of dependencies dragged in by `PRHARedundancyProvider`,
and describe how that affects control and observation.

During initialization,
the `initializeDataStore()` method
creates a `PartitionedRegionDataStore`
[by calling `new`](https://github.com/apache/geode/blob/0ea005d5d7d1deb5ebe9639b34b0294af577b51d/geode-core/src/main/java/org/apache/geode/internal/cache/PartitionedRegion.java#L1375-L1377):

```java
this.dataStore =
    PartitionedRegionDataStore.createDataStore(cache, this, ra.getPartitionAttributes(),
        getStatisticsClock());
```

Numerous other `PartitionedRegion` methods
create additional tacit dependencies.
In all,
a `PartitionedRegion` creates dozens of tacit dependencies.

TODO: Is it the PR's responsibility to get the data store into the right state?
The data store already handles a great deal of that responsibility...
as long as it's called with the appropriate arguments.
Maybe we can let PR focus on _calling the data store_ correctly,
and leave it to the data store to actually do the storage.

### How Tacit Dependencies Inhibit Testing PartitionedRegion

Two of `PartitionedRegion`'s tacit dependencies in particular
make it difficult to write this test:
The redundancy provider
and the data store.

The region has a getter that makes its data store accessible.

# Improve Testability by Making Dependencies Explicit

**Mock complex collaborators.** If a collaborator's state is complex enough to inhibit control and observation, use mocks instead of real collaborators to make control and observation easier.

**Inject dependencies.** Dependency injection allows tests to configure real or fake collaborators to make them easier to control and observe.

**Use factories to create collaborators.** Sometimes creating a collaborator requires information computed by the object being tested. In these cases, inject a factory for the object to use to create the collaborator. The test can configure the factory to return a collaborator that the test can easily control and observe.

**Extract cumbersome responsibilities.** Sometimes an implementation method creates tacit dependencies. Consider extracting that method's responsibilities into a separate class. That class then becomes a *collaborator* of the class under test, which allows a test to supply a more controllable and observable implementation.








[^other-ways]: In addition to these two common ways of creating tacit dependencies,
there are other ways
that are more complicated or more subtle.
I'll leave those for another day.

[^variables]: [Testing With Variables](https://vimeo.com/34356209)
demonstrates how even the simplest software
involves a multitude of variables.

