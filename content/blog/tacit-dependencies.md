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

Suppose we want to test a Java object,
an instance of some Java class.
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
[_anything that can vary_](https://vimeo.com/34356209).)

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

Examples abound in Geode code.
The `PartitionedRegion` class's constructor
alone
includes multiple examples
of each.

On lines [lines 791-2](https://github.com/apache/geode/blob/0ea005d5d7d1deb5ebe9639b34b0294af577b51d/geode-core/src/main/java/org/apache/geode/internal/cache/PartitionedRegion.java#L791-L792)
the constructor creates a collaborator by calling `new`:

```java
this.prStats = new PartitionedRegionStats(cache.getDistributedSystem(), getFullPath(),
        statisticsClock);
```

And on [line 813](https://github.com/apache/geode/blob/0ea005d5d7d1deb5ebe9639b34b0294af577b51d/geode-core/src/main/java/org/apache/geode/internal/cache/PartitionedRegion.java#L813)
the constructor obtains a collaborator
by calling a static method
of another class:

```java
this.distAdvisor = RegionAdvisor.createRegionAdvisor(this);
```





## How Tacit Dependencies Inhibit Testing

Tacit dependencies make an object harder to test
by inhibiting tests' ability to:

-   Control the conditions
    that affect the responsibility being tested.
-   Observe the results
    that the object produced.

### Tacit Dependencies Inhibit Control


If the responsibiilty we're testing
depends on how a collaborator responds,
the test must control the collaborator
to make it respond in a particular way.
Further, if the collaborator's response
depends on its state,
the test must control the collaborator's state.

Classes that create tacit dependencies
often keep them private.
Tests cannot control these dependencies directly,
and may not be able to control them even indirectly.

Tacit dependencies are often not directly accessible.
An object might create a collaborator
and assign it to a private field.
Or it might create a collaborator,
interact with it,
and discard it.

### Tacit Dependencies Inhibit Observation




## Examples from Geode Code

Here are some examples of tacit dependencies from `PartitionedRegion`'s constructor.

### Creating Tacit Dependencies

Note that calling this constructor
introduces an _unnecessary_ dependency.
`PartitionedRegion` has no need for the `statisticsClock` parameter
other than to forward it to the `PartitionedRegionStats` constructor.
If instead the stats instance were passed as a parameter,
the `PartitionedRegion` constructor
would have no need for a `statisticsClock` parameter at all.

### Obtaining Tacit Dependencies from Static Sources

```java
this.distAdvisor = RegionAdvisor.createRegionAdvisor(this);
```

This makes _every instance_ of `PartitionedRegion` dependent on
whatever implementation `RegionAdvisor` chooses to create.




## To Make a Class More Testable, Make its Dependencies Explicit




[^other-ways]: In addition to these two common ways of creating tacit dependencies,
there are other ways
that are more complicated or more subtle.
I'll leave those for another day.