---
layout: post
title: Kotlin Best Practices
date: 2018-04-13
comments: true
description: For many Kotlin adopters coming from Java, it's natural to start writing Kotlin in the same style that you're used to. After all, many of the constructs and libraries are the same, so it's easy to continue writing code the same way you always have. Kotlin is all about saving time and improving code **maintainability** through **readability**, and these Java conventions frequently detract from that.
---

For many Kotlin adopters coming from Java, it's natural to start writing Kotlin in the same style that you're used to. After all, many of the constructs and libraries are the same, so it's easy to continue writing code the same way you always have. Kotlin is all about saving time and improving code **maintainability** through **readability**, and these Java conventions frequently detract from that.


Our teams tend to start writing Kotlin the same way, though we've come a long way in the past year and a half. Here's a collection of best practices that we've come up with through our code reviews.

### Contents

[1. Embrace Immutability](#1-embrace-immutability)  
[2. Ditch ArrayList && Hash Map](#2-ditch-arraylist--hashmap)  
[3. Use Functional Constructs](#3-use-functional-constructs)  
[4. javaClass](#4-javaclass)  
[5. String Interpolation](#5-string-interpolation)  
[6. Infer Types](#6-infer-types)  
[7. Semantic Test Naming](#7-semantic-test-naming)  
[8. Safe Operator?](#8-safe-operator)  
[9. Elvis Throws](#9-elvis-throws)  
[10. List Literals in Annotations](#10-list-literals-in-annotations)  
[11. Collection Helpers](#11-collection-helpers)  
[12. No more .equals()](#12-no-more-equals)  
[13. Method Readability - Named Parameters](#13-method-readability---named-parameters)  
[Learn More](#want-to-learn-more)

## 1. Embrace Immutability

Java generally isn't explicit about immutability. While there's the `final` operator that can help, it isn't always used consistently. Given that immutable programming is often simpler and less error prone, it makes sense to get the habit of making everything immutable, and only reverting to mutable objects when absolutely necessary.

```java
public void main() {
    String someData = "Data";
    int someNumber = 42

    someData = null; // NPE!
    someNumber = 2;
}
```

In this example, there's nothing preventing the modification of the values behind `someData` and `someNumber`, they're mutable variables. If some code later on were to use those values, there could be unintended consequences. While this may be fine for one developer working on a project, teams or even an individual who has been away from a piece of code for a while can lose track of the usages of a value, and could unintentionally introduce a bug here.

Instead, Kotlin makes it incredibly easy to declare values immutable from the beginning. In fact, IntelliJ Idea will typically recommend a quick-fix to change `var` variables to `val` if they value isn't being mutated anywhere.

```kotlin
fun main() {
    val someData = "Data"
    var someMutableNumber = 42

    someData = "some other data" // Compiler error! val cannot be reassigned
    
    // This is okay, someMutableNumber was declared as a mutable var
    someMutableNumber = 2
}
```


In general, seeing `var` in code should raise a red-flag. There are times where it's necessary, but the majority of usecases can be refactored to be immutable.

## 2. Ditch ArrayList && HashMap

For Java developers, `ArrayList` and `HashMap` are probably some of the most frequently used collections. I personally don't find either name to be particularly useful, though. I _know_ that these collections are mutable, but it's not natural and easy to read.

```kotlin
fun oldCollections() {
    val myList = ArrayList<Int>()
    myList.add(1)
    myList.add(2)
    myList.add(3)

    val myMap = HashMap<Int, Int>()
    myMap[0] = 0
}

```

Kotlin instead provides some useful helpers over collection interfaces to abstract away the details underneath in favor of readability. 

```kotlin
fun newCollections() {
    val myList = listOf(1, 2, 3)
    my mutableList = mutableListOf(4, 5)
    
    myList.add(4) // Error: List<Int> does not have method add()
    mutableList.add(5)

    val myMap = mapOf(0 to 0, 1 to 1)
    val mutableMap = mutableMapOf(2 to 2)

    mutableMap[3] = 3
}

```

I'll concede that this does hide some details that could be important for performance on large collections. In specialized cases, you'll likely want to use more specific implementations. However, _most of the time_, it's probably not important enough to sacrifice code readability.

## 3. Use Functional Constructs

Functional constructs help bring together immutability and collections, while also cleaning up code. When operating on a collecton, we often write code that looks soemthing like this:

```kotlin
fun doubleList(values: Array<Int>) {
    for (i in 0..values.count()) {
        values[i] = values[i] * 2
    }
}

```

Not only is this method mutating the array that was passed in, but the signal to noise ratio isn't particularly good. The `for` syntax is understandable, but it's extra noise detracting from the "signal" in the method where the value multiplication is happening. This is an excellent place for an operation like `.map()`:

```kotlin
fun doubleList(values: Array<Int>): Array<Int> {
    return values.map { it * 2 }
}

```

This example is clearly much shorter, with far less "noise" code around the important multiplication bit. Another important point here is that the original array isn't being modified at all, but rather a copy of the array with the modifications is being returned. Using the various functional constructs like `.map()` can often remove the need to abstract collection operations out to another method, since calling them in-line is typically clean enough.

**Bonus:**

```kotlin
fun getNames(people: List<Person>): List<String> {
    return people.map { it.name }
}

```

Another area where `.map()` shines is extracting values from complex objects. This is another instance where you may typically loop over the complex objects and build up a new list with the desired values, which `.map()` can do in a single call.

Java does have these features with the Streams API starting in Java 8, and this is definitely a nice feature in Java projects. Unfortunately, the Streams API isn't available for people who are forced to use older versions of Java, and the API itself isn't quite as intuitive as Kotlin's.

## 4. javaClass

This one in particular I would call less a "best practice" and put it more in the bucket of "here's a cool tip". Some Java libraries, particularlly loggers, need a reference to the current class. In Java, this is done pretty simply:

```java
LoggerFactory.getLogger(MyClass.getclass());

// Or

LoggerFactory.getLogger(this.getclass());

```

In Kotlin, :

```kotlin
LoggerFactory.getLogger(MyClass:class.java)

// Or 

LoggerFactory.getLogger(this::class.java)

```

These both work fine. The first is prone to copy/paste issues, and both otherwise have a lot going on. Kotlin does provide a nice little shortcut for this, `javaClass`:

```kotlin
LoggerFactory.getLogger(javaClass)

```

## 5. String Interpolation

Concatenating strings is not only one of the fundamental things that people learn early on in the programming world, but it's also used in some form in nearly every programming project. Many languages provide a few alternatives to standard concatenation to help with formatting or performance, but there's typically a price to pay in terms of readability. 

```java
System.out.println("Processing error at " + DateTime.now() + 
    " with message : " + ex.message + ".");

```


Concatenation is straightforward: take these pieces of string and add them together, but that comes with a lot of additional symbols that can be frustrating to keep track of. Ultimately, the most readable facet of string concatenation is the generally final string, so ideally, the code would come as close as possible. To get to this, Kotlin has string interpolation (frequently called string templating) that makes the construction of a string much easier to read.

```kotlin
println("Processing error at ${DateTime.now()} with message: ${ex.message}.")

```

## 6. Infer Types

This is a great feature in Kotlin that can be a little awkward if you're not used to a language where types don't need to be explicitly declared. Kotlin's compiler is smart enough to infer the type of most things, without losing the power of those types in practice (unlike JavaScript where the type system is somewhat rudimentary). I won't get into a debate about whether static or dynamic typing is better, but I think it goes without saying that typechecking can help avoid bugs, typically at the cost of readability. For example:

```java
User currentUser = new User(username, email, profileUrl);

```

Declaring `currentUser` as type `User` here is redundant. The variable is immediately initialized to a `User` object, and a maintainer can easily see that as they're reading through. Kotlin is flexible to allow the omission of the type in this case, as well as smart enough to infer the type in most other situations.

```kotlin
val currentUser = User(username, email, profileUrl)

```

Of course, this is a contrived example that's relatively simple. "User" isn't that many extra characters at the end of the day, and [finite keystrokes aside](https://blog.jonudell.net/2007/04/10/too-busy-to-blog-count-your-keystrokes/), it really doesn't take very long. Remember, readability is key, and mental time spent filtering out redundant types is time not spent interpreting the meaning of a line of code. Plus, Java class names can get quite long, take this example of getting the configuration of an AWS Lamdbda function:

```java
public String getLambdaArn(String functionName) {
    GetFunctionConfigurationRequest req = new GetFunctionConfigurationRequest()
        .withFunctionName(functionName);
    GetFunctionConfigurationResult result = client.getFunctionConfiguration(req);
    return result.getFunctionArn();
}

```

This code isn't doing anything overly complex and the types as they are make sense, but writing out `GetFunctionConfigurationRequest` twice just isn't a good use of time or mental capacity. Further, while it's nice to know that `result` is going to be of type `GetFunctionConfigurationResult`, someone reading this code could probably infer pretty easily that it will contain the result of getting the function configuration without the type declaration. Remember, write code for humans, not machines.

```kotlin
fun getLambdaArn(functionName: String): String {
    val req = GetFunctionConfigurationRequest()
        .withFunctionName("myFunction")

    val result = client.getFunctionConfiguration(req)
    return result.functionArn
}

```

For those who still like seeing types (because it can still be helpful when writing code), [Intellij Idea can show type hints in the editor](https://i2.wp.com/blog.jetbrains.com/kotlin/files/2017/06/kotlin113-type-hints.png?ssl=1).

## 7. Semantic Test Naming

While standard method naming convention calls for short and descriptive names, test methods generally get long names that describe what particlar scenario the test covers. For example, we've all probably written and sifted through tests that look like this:

```kotlin
@Test
fun handlerShouldSaveRecordToDbOnUpdatedEventProcessed() { ... }

```

This is descriptive, and not very easy to read. [Kotlin's conventions](https://kotlinlang.org/docs/reference/coding-conventions.html#naming-rules) allow for naming test methods in natural language format with spaces (or underscores).

```kotlin
fun `handler should save record to db on updated event processed`() { ... }

```

## 8. Safe Operator?

One of Kotlin's headline features is its handling of compile-time null safety. Through the implementation of its null system, Kotlin provides a handful of helpers to make handling nullable types easy and clean. Traditional Java convention translated to Kotlin looks something like this:

```kotlin
fun getNameFromDb(): String? {
    val dbRow: DbRow? = selectFromDbById(2)

    if (dbRow != null) {
        if (dbRow.person != null) {
            if (dbRow.person.name != null) {
                return dbRow.person.name
            }
        }
    }

    return null
}

```

Instead, Kotlin provides helpers similar to [other languages](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-conditional-operators) that support nullable types:

```kotlin
fun getNameFromDb(id: Int): String? {
    val dbRow: DbRow? = selectFromDbById(id)

    return dbRow?.person?.name
}

```

The `?` access operator will only continue with accessing the field if the parent container is not null. If any value in the chain is null, the entire line will return null. You can read more about Kotlin safe calls [here](https://kotlinlang.org/docs/reference/null-safety.html#safe-calls).

## 9. Elvis Throws

Following along with safe calls, the [Elvis Operator](https://kotlinlang.org/docs/reference/null-safety.html#elvis-operator) can coalesce a nullable value into a non-null value. In the example above, `getNameFromdb()` returns a nullable string, and relies on the caller to handle the case when a null value is returned. Using the elvis operator, we can assert that `getNameFromDb()` will always return a non-null string or throw a more detailed exception if the value can't be found.

```kotlin
fun getNameFromDb(id: Int): String {
    val dbRow: DbRow? = selectFromDbById(id)

    return dbRow?.person?.name 
        ?: throw NotFoundException("Unable to find user with id $id")
}

```

## 10. List Literals in Annotations

In Spring, some controller annotations can take multiple values for a field. For example, `@RequestMapping` can take multiple values for the method of the call, and Kotlin code for that would look something like this:

```kotlin
@RequestMapping(value = "/endpoint", method = arrayOf(RequestMethod.POST))
fun updateEndpoint() {}

```

Using `arrayOf()` here is necessary, but wordy. Instead, Kotlin supports list literals in annotations:

```kotlin
@RequestMapping(value = "/endpoint", method = [RequestMethod.POST])
fun updateEndpoint() {}

```

## 11. Collection Helpers

Kotlin's collections have powerful helper methods that can make operating on collections cleaner and more "Kotlin-y". Typical operations on collections involve looping, often with a mutable accumulator collection that stores the result.

```kotlin
fun searchForElem(searchElem: Any, list: List<Any>): Any? {
    for (elem in list) {
        if (searcnElem == elem) {
            return elem
        }
    }

    return null
}

fun profilesWithPicture(profiles: List<Profile>): List<Profile> {
    var foundProfiles: List<Profile> = mutableListOf()

    for (profile in profiles) {
        if (profile.picture != null) {
            foundProfiles.add(profile)
        }
    }

    return foundProfiles
}

```

These types of operations, as well as many others, are common, and generally end up in the creation of some sort of utility class or library so that they can be reused. The latter of the examples, `profilesWithPicture()` also requires the use of a mutable list, which deviates from immutability as a best practice. To save time and encourage immutable conventions, Kotlin's standard library comes with a handful of helper methods on built-in collections. These take in either a function reference or a lambda that is performed against each element of the collection. I typically use the lambda syntax:

```kotlin
fun searchForElem(searchElem: Any, list: List<Any>): Any? {
    return list.firstOrNull { it == searchElem }
}

fun profilesWithPicture(profiles: List<Profile>): List<Profile> {
    return profiles.find { it.picture != null }
}

fun checkAllProfilesHavePicture(profiles: List<Profile>): List<Profile> {
    return profiles.all { it.picture != null }
}

```

There's quite a few more operations like `.contains()`, `.isEmpty()`, `.filter()` and more available on Kotlin collections. The Kotlin [documentation for List](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list/index.html) outlines these, and most of them are transferrable to collections of other types.

## 12. No more .equals()

As someone originally coming from a Python and C# background, the way that Java handles checking equality caused me a great deal of pain and debugging. In those languages, as well as many others, the `==` operator determines value equality or calls an equals method on a complex object. This means that everything from numbers to strings to objects can have their equality checked via `==`.

In Java, operator overloads don't exist, so `==` determines reference equality (something I've never been inclined to actually check for). This means that only primitives can be checked using `==` and complex objects (like Strings) call for the use of `.equals()`. Imagine sifting through code that looks like this:

```java
if (input.equals("some other string")) {}

if (numberInput == 5) {}

```

Kotlin allows for [operator overloading](https://kotlinlang.org/docs/reference/operator-overloading.html), and `==` essentially becomes an alias to `.equals()`. This allows the operator to be used on built-in Java types as well as custom types that have already implemented `.equals()`. For new Kotlin objects, [data classes](https://kotlinlang.org/docs/reference/data-classes.html) will implement a default `.equals()` method for you, which will check the equality of each field.

At the end of the day, this makes Kotlin code easier to follow and easier to pick up and transition for pepole who are used to non-Java equality

```kotlin
if (input == "some other string") {}

if (myClass == otherClass) {}

```

## 13. Method Readability - Named Parameters

Object-oriented practices, particularly in Java, call for the use of what's known as the [builder pattern](https://www.javaworld.com/article/2074938/core-java/too-many-parameters-in-java-methods-part-3-builder-pattern.html) when creating objects that have a number of different fields. While this improves call-site readability of a constructor, it causes a sprawl of code that has to be created in some other class whose sole purpose is to create other classes in a cleaner way. Kotlin removes the need for this by permitting named parameters on both constructors and standard methods.

```kotlin
val profile = Profile(
    firstName = "Bob",
    lastName = "Kotlin",
    profileUrl = "https://kotlinlang.org/",
    email = "bob.kotlin@test.com"
)

```

On a constructor, this can make it very clear what value is going where, rather than relying on positional indexing. While methods with a large number of arguments is generally considered an anti-pattern, named parameters give you the ability to add more parameters if needed without a major loss in comprehendability.

```kotlin
val apiResult = getFromApi(
    client = apiClient,
    baseUrl = API_URL,
    endpoint = API_ENDPOINT,
    headers = customHeaders,
    authToken = token,
    queryString = null
)

```

This is a contrived example that probably lends itself to better design, but it helps illustrate how a method with that many parameters could get hard to understand and debug if it weren't for the parameter names to help.

## Want to Learn More?

If you're interested in learning more about Kotlin, I highly recommend taking a walk through the [language reference documentation](https://kotlinlang.org/docs/reference/). I believe that it's very well written and is quite helpful in getting started. <a target="_blank" href="https://www.amazon.com/gp/product/1617293296/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=1617293296&linkCode=as2&tag=batchofcode-20&linkId=ae56c27ff64c13311877e267ee2924ae">Kotlin in Action</a><img src="//ir-na.amazon-adsystem.com/e/ir?t=batchofcode-20&l=am2&o=1&a=1617293296" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" /> is a book that I've found helpful. The authors both work on Kotlin development at JetBrains and explain concepts with examples and detail that help the reader understand why a particular feature is useful.