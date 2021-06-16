---
title: "Code Smells: Cast and instanceof"
draft: true
---

In Java, casts and uses of `instanceof`
may indicate that the code that follows
might better be placed in some other class.
<!--more-->

# Casts as Code Smells

Every cast is a sign that this code is about to:

- Interact with an object in a way that its class's interface does not mention.

# `instanceof` as a Code Smell

```java
if (o instanceof SpecialClass) {
    ...
}
```

Notes:
- This code receives `o` from elsewhere.
- This code does not know `o`'s type.
- Whatever `o`'s type,
    this code has a responsibility
    to interact with it in some way.
- In order to interact correctly with `o`,
    this code needs to know whether `o` is a `SpecialClass`.
- The method through which this code received `o`
    did not declare `o` to be `SpecialClass`.
- So we know this code
    _wants to interact with `o`
    in some way
    that is not mentioned in the method's signature._
