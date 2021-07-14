---
title: Tacit Dependencies Inhibit Testing
draft: true
---

Code becomes difficult to test
when it is coupled to dependencies
that are hard to control and observe.
Tacit dependencies are hard to control and observe.
To make a class easier to test,
make its dependencies explicit.
<!--more-->




## Testability

Suppose we want to test an instance of some Java class.
To do that, we:
1. Establish any relevant preconditions
    in the object,
    in the object's collaborators,
    or elsewhere in the object's environment.
1. Send the object a message
    that triggers it to carry out
    the responsibility we're testing.
1. Observe the results
    that the object produced.

Step 1 requires us to
_control_ the variables
that affect how the object carries out its responsibilities.
Step 3 requires us to
_observe_ the variables
affected by the object as it carries out its responsibilities.
(Note that by _variable_ I mean
_anything that can vary[^variables]._

> **Testability = controllability + observability.**

The easier it is to control and observe
the variables involved in an object's responsibilities,
the easier the object is to test.
The harder it is to control and observe those variables,
the harder the object is to test.

Below, we will
use `PartitionedRegion`
[as an example](#testing-partitionedregion-virtualput).




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

TODO

### Transitive Tacit Dependencies

If an object tacitly depends on collaborators,
and those collaborators tacitly depend on yet more collaborators,
our object becomes increasingly difficult to test.

## To Make a Class More Testable, Make its Dependencies Explicit

TODO


## Examples

### Tacit Dependencies in PartitionedRegion

`PartitionedRegion` creates dozens of tacit dependencies.
The constructor alone
includes multiple examples
of tacit dependencies.
Here are several examples.

The constructor creates a `RegionAdvisor`
[by calling a static method](https://github.com/apache/geode/blob/0ea005d5d7d1deb5ebe9639b34b0294af577b51d/geode-core/src/main/java/org/apache/geode/internal/cache/PartitionedRegion.java#L813)
on the implementation class:

```java
this.distAdvisor = RegionAdvisor.createRegionAdvisor(this);
```

This makes the `PartitionedRegion` class
tacitly depend on the `createRegionAdvisor()` method's choices
of which implementation to create
and how to configure it.

A few lines later it creates a `PRHARedundancyProvider`
[by calling `new`](https://github.com/apache/geode/blob/0ea005d5d7d1deb5ebe9639b34b0294af577b51d/geode-core/src/main/java/org/apache/geode/internal/cache/PartitionedRegion.java#L816):

```java
this.redundancyProvider = new PRHARedundancyProvider(this, cache.getInternalResourceManager());
```

This makes `PartitionedRegion`
tacitly depend on a _specific implementation_ of redundancy provider
and, transitively,
on every tacit dependency of that specific implementation.

The constructor is not the only place where `PartitionedRegion`
tacitly creates the collaborators it depends on.
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


### Testing PartitionedRegion virtualPut()

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
Otherwise the destination is
a remote member.


The choice of message to send
depends on the parameters passed to `virtualPut()`:
If `isNew` is `true`,
the region must send a _create_ message to the destination.
Otherwise it must send a _put_ message.


**A test.**
Consider a test
for a single combination of conditions and inputs:
- Conditions: The partitioned region has a data store.
- Inputs: The caller calls `virtualPut()`,
    passing `true` as the argument to `ifNew`.
- Results: The partitioned region
    must call the data store's `createLocally()` method,
    passing (mostly) the same arguments that were passed to `virtualPut()`.

To test this responsibility,
the test must:
- **control** whether the partitioned region has a data store.
- **observe** what message the partitioned region sent, and to what destination.

### How Tacit Dependencies Inhibit Testing PartitionedRegion

Two of `PartitionedRegion`'s tacit dependencies in particular
make it difficult to write this test:
The redundancy provider
and the data store.

The region has a getter that makes its data store accessible.









[^other-ways]: In addition to these two common ways of creating tacit dependencies,
there are other ways
that are more complicated or more subtle.
I'll leave those for another day.

[^variables]: [Testing With Variables](https://vimeo.com/34356209)
demonstrates how even the simplest software
involves a multitude of variables.

