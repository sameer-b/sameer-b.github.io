---
layout: single
title:  "Meaningful Abstractions"
date:   2019-01-29 21:21:10 -0800
tags: software architecture
---

Interfaces are incredibly powerful software design constructs, I cannot imagine writing building any non trivial piece of software without them. Interfaces enable us to define abstractions and build contracts before actually implementing the software. If defined succinctly, a well defined interface could make your implementation trivial. They enable us to leverage powerful tools such as polymorphism that we all love. Yet I've seen folks (including myself) abuse them frequently. In this post I'm going to talk about the incorrect usages of interfaces, where they don't add any value. More specifically I'm going to talk about the common pattern: `Interface -> Class` pattern we adopt blindly.

There have been numerous times I've seen the following pattern:

```java
public interface PermissionService {
    Permission getPermissionForUser(User user);
}
```

The implementation of the interface looks something like:

```java
public class PermissionServiceImpl implements PermissionService {
    Permission getPermissionForUser(User user) {
        //implementation for fetching the user mission
    }
}
```
In the above example the interface is very specific to the implementation. Maybe we could ditch our interface and simplify our implementation as follows:

```java

public class PermissionService {
    Permission getPermission(User user) {
        //fetch user's mission
    }
}
```

Let's compare the example above with the first example. You may ask what's the fuss with creating the interface? After all, it only takes a few seconds to actually create it. Hold on, what if we need to rename the `getPermission` API, now we need to make changes to the API and the implementation. In the real world, this could involve building two different packages. How many times have you changed the implementation and started your build then noticed a build failure? This is productivity lost! What if we have to relocate the class to a another package or folder structure? You get the idea...

 What would an alternate implementation of `PermissionService` look like? Maybe we could model the `user` more generically and our service could represent generic permission checks. Let's refine our example.

```java
public interface PermissionService {
    Permission getPermission(Persona persona);
}
```

Now that we have a more generic input, a `Persona` could represent either a `User` or `Admin` or `SalesAgent`. Let's see what our implementations would look like.

```java
public class UserPermissionService implements PermissionService {
    Permission getPermission(Persona persona) {
        //fetch user's permissions
    }
}
```
An alternate implementation could be for a `Admin` or `SalesAgent`

```java
public class AdminPermissionService implements PermissionService {
    Permission getPermission(Persona persona) {
        //fetch admin's permissions
    }
}
```

```java
public class SalesAgentPermissionService implements PermissionService {
    Permission getPermission(Persona persona) {
        //fetch sales agent's permissions
    }
}
```

This example may make more sense to you, but this is only useful if we really do have use cases to support the different types of users. If it's not a realistic scenario, we have over engineered our design and introduced an interface that adds no real benefit, but has a maintenance cost. Maybe we only have a `User` and our service is only used in the context of the user. Our code could be a lot cleaner if we kept things simple.


Let's look at a more real word example of interfaces.

```java
  public interface AutoClosable {
      void close() throws Exception;
  }
```

 You're probably familiar with this interface, you could use any class that implements this interface with a [try-with-resource](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html) statement, and `close` method will be called automagically. If you create new network stream or database a connection, they could implement the interface and you're golden. This is because the interface is well through of, it seldom changes (never?) and its purpose is very clear. It does one thing, and it's well defined.

Let's look at a few examples from [java.util.Collections](https://docs.oracle.com/javase/8/docs/api/?java/util/Collections.html). It defines generic data structures such as `List` or `Map`. Could you have several implementation of each, yet a uniformed API to operate them. You could `add` things to a `List` without worry about the underlying implementation, it could be a `Vector`, `ArrayList` or a `LinkedList`. The interfaces are modeled after abstract data structures. It wouldn't make much sense to have one interface for `LinkedHashMap` and one for `HashMap`, would it?

### Conclusion

Hopefully this post has convinced to that creating interfaces for the sake of having abstract is bad and it's always good to spend more time thinking about your abstractions. It's also crucial to define your contracts, identify your clients in the process of defining them. Creating interfaces is cheap, but maintaining them can be expensive. Keep things simple and avoid bloat where possible.
