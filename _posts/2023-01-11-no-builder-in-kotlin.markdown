---
layout: post
title: "How to do the builder pattern in Kotlin"
date:   2023-01-11 12:00:00 +0200
author: Stephan Schröder
categories: tech
tags: Kotlin Lombok Builder mapstruct
excerpt: "<br/>Java has Lombok with its @Builder annotation. How to replicate this functionality when moving the codebase to Kotlin<br/><br/>"
---

Kotlin was developed  as a "better Java", with the interoperability with it being a primary concern. Not totally coincidentally that makes moving from Java to Kotlin very straight forward.
Most features of Java and its immediate ecosystem including the likes of Lombok have direct or not so direct counterparts.
So while rewriting a Java project to Kotlin, should you e.g. encounter a Lombok @Data annotation, it's easy to choose a `data class` on Kotlin's side.

**But what about Lombok's @Builder annotation? There seems to be no in-build builder facility in Kotlin. Do we have to implement a Builder for each class manually?**

**TLDR:** No, we don't. Use optional parameters (=parameters with default arguments) in your Kotlin class and use named parameters when invoking it.  

## What is the problem Lombok @Builder is solving?

Let's assume we have a dto containing many properties including `username` and `password`. The class has one constructor to initialize all its properties.
Invoking this constructor in Java>=10 will look something like this:

```java
var a = new A(null, null, "a_schmidt", "1234", null, null);
```

The issues here include:
- a lot of visual noise (having to write `null` for all the parameters you don't have a value for)
- are you sure that username and password are on the right position without jumping to the constructor definition?
The compiler only guarantees that the types are matching, but how do you know that you didn't mix up the order of - in this case - username and password?

## How Lombok @Builder helps

adding a `@Builder` annotation to `A` will allow you to write code like this

```java
var a = A.builder()
            .setUsername("a_schmidt")
            .setPassword("12324")
            .build();
```
As you can see, we no longer have to provide the parameters we don't have values for, and it's immediate apparent that you didn't confuse the order of parameters.
It's definitely progress.

**Side note: If you're very concerned about securing that you don't mix up usernames and passwords, you can use wrapper types.**
After all your organisation might give out usernames like "1234" and allow passwords like "a_schmidt". With Lombok's help, this would look like

```java
@Value
class Username {
    private String value;
}

@Value
class Password {
    private String value;
}

@Data
class A {
    private Role role;
    private String x1;
    private Username username;
    private Password password;
    private String x2;
    private String x3;
}
```

Now even invoking the constructor without a Builder mixed up would fail (at least for username and password), since

```java
var a = new A(null, null, new Password("1234"), new Username("a_schmidt"), null, null);
```
is a compiletime error.

## Why this isn't a problem in Kotlin to begin with

You don't need to employ the Builder in Kotlin because Kotlin provides optional parameters, named parameters and nullability also plays a certain part. 
So given a class declaration like this:

```kotlin
data class A(
    val role: Role = Role.User, // provide a sensible default if such a default should exist 
    val x1: String? = null,     // or use a nullable type and initialise it with null by default
    val username: String,
    val password: String,
    val x2: String? = null,
    val x3: String? = null,
)
```

you can invoke the constructor like this

```kotlin
val a = A(
    username = "a_schmidt",
    password = "1234",
)
```

As you can see, this looks very close to what a Builder in Java gives you, but comes out of the box in Kotlin.
Yes, you do have to think a bit more when writing the constructor in order to determine which parameters are optional and which ones have to be provided every time.
This gives you additional security though (not even thinking about nullability here). In Java you can write code like

```java
var a = A.builder()
            .setPassword("12324")
            .build();
```

and create an instance that is bound to trigger a NullPointer exception (or fail a check) at runtime.
In Kotlin the equivalent code

```kotlin
val a = A(
    password = "1234",
)
```
won't even compile. Most likely you will notice it immediately because your IDE will point it out to you.

**Side note: wrapper types don't cause a runtime overhead in Kotlin.**
So obviously you can also write wrapper types in Kotlin. Not so obvious is that you can do it in a way that is more efficient at runtime.
The problem here is that normal classes cause a pointer indirection overhead, since you have to follow one more pointer to reach the final value. 
For this reason some people avoid wrapper types even though the gained compiletime safety would probably be worth it.
Kotlin provides [inline classes](https://kotlinlang.org/docs/inline-classes.html), that are (most often) compiled away. 
So you are left with all the safety and (probably) none of the runtime overhead.

An inline class can only wrap a single property and looks like this:

```kotlin
@JvmInline
value class Username(private val value: String)

@JvmInline
value class Password(private val value: String)
```

## Extra: What about mapstruct?

With what to replace or how to use [mapstruct](https://mapstruct.org/) is the second most common question I heard when refactoring a Java project to Kotlin.
Mapstruct itself will tell you that you can use it in Kotlin. The problem is that it knows nothing about Kotlin nullability or optional parameter, so all the properties of
your Kotlin class would have to be nullable and without default value. This is obviously not idiomatic for Kotlin, so **don't use mapstruct** in Kotlin.

To the best of my knowledge there's no Kotlin equivalent compiler plugin either to take over mapper generation in Kotlin land.
But I suspect that the reason for this is that writing mappers manually in Kotlin is less cumbersome than in Java.

Not only do you have optional and named parameters, I normally don't even see a point in writing a mapper class.
I write an **extension function** instead:

```kotlin
fun A.toB(): B = B(
    username = this.username,
    password = this.password,
    isImportant = this.role == User.Admin,
)
```

The reason why I do this as an extension function - and not as a normal function in A - is that mapping doesn't belong to
the core responsibilities of A. At its declaration site A doesn't even need to know that B exists.
But it's still nice to convert an instance of A on the fly writing `a.toB()`.
Of course, this way it's no longer possible to mock the mapping-part.

# About the author

[Stephan Schröder](https://github.com/simon-void/) is a senior software developer and works for [gematik GmbH](https://www.gematik.de/).
He has a long history with Java and is intrigued by languages that offer even more safety guarantees at compiletime like [Kotlin](https://kotlinlang.org/) and [Rust](https://www.rust-lang.org/).
He also works at starting an [Aikido dojo](https://flux-aikido.com/).