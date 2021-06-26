---
title: Testability and Tacit Dependencies
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

Testability = controllability + observability.

To test a class, we:

1. Establish some preconditions
    in an object that is an instance of the class,
    in the object's collaborators,
    or elsewhere in the environment.
1. Send the object a message
    that triggers it to carry out
    the responsibility we're testing.
1. Observe the results
    that the object produced.

The first step requires us to
control the variables
that influence the object's behavior.
The third step requires us to
observe the variables
affected by the object.

The easier it is to control and observe
the variables involved in the class's responsibilities,
the easier the class is to test.
The harder it is to control and observe those variables,
the harder the class is to test.

## Tacit Dependencies

The _tacit dependencies_ of a class
are the objects and other resources
that it interacts with
that are not mentioned explicitly
in the class's public interface.

Tacit dependencies include any collaborator
that the object obtains by:

-   Creating it.
-   Calling a static method of another class.
-   Reading a static field of another class.
-   Interacting with collaborator obtained
    in one of these ways.

## How Tacit Dependencies Make Testing Harder

Tacit dependencies make a class harder to test.
Here are some examples from `PartitionedRegion`'s constructor.

### Creating a tacit dependency

```java
this.prStats = new PartitionedRegionStats(cache.getDistributedSystem(), getFullPath(),
        statisticsClock);
```

Note that `PartitionedRegion` has no need for the `statisticsClock` parameter
other than to forward it to the `PartitionedRegionStats` constructor.
If instead the stats instance were passed as a parameter,
the `PartitionedRegion` constructor
would have no need for a `statisticsClock` parameter at all.

### Obtaining a dependency by calling a static method on another class
```java
    // Warning: potential early escape of instance
    this.distAdvisor = RegionAdvisor.createRegionAdvisor(this);

```

This makes _every instance_ of `PartitionedRegion` dependent on
whatever implementation `RegionAdvisor` chooses to create.

The comment raises another concern.
I'll leave that for another day.

### Obtaining a dependency by interacting with any non-parameter object
```java
```


## To Make a Class More Testable, Make its Dependencies Explicit

