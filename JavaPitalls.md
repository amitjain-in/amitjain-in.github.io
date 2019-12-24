# Pitfalls in Java Programming
Date: 24 Dec 2019

Java has been an [extensively](https://www.tiobe.com/tiobe-index/java/) used programming language over last two decades (and some more) and I suspect it will continue to stay relevant for next two decades at least, since a huge number of commercial projects has been using it over others. This article will try to point out some of the things that beginners or intermediate level programmers need to be aware of while working with Java.
Some of them are straight out obvious while few of them would be debatable and subject to personal preferences.

So let's start with some mistakes that beginners might make.

## Not using an IDE
As a first time programmer learning java, some tutorials would guide you through compiler and run command tools like javac, java. While it may be the right approach for your first couple of programs very soon you will start to feel the pinch of typing the code in your editor and then going back to javac to see if runs or not. The bigger the project, then more cumbersome it becomes and hence you need start using IDEs. 
There are several benefits writing code in IDEs in Java
- Java is a statically typed language and the IDEs point errors and suspicious code while you are typing them. This alone would save you huge amount of time. Being statically typed language has another benefit that IDEs can refactor the code far smartly and almost completely error free.
- There is another huge benefit of IDEs that it would suggest you improvements over your code and in a sense that becomes your teacher. Below is an example with Intellij idea. My intent is write method which inserts into map if there is no element found in the map. Intellij Idea is suggesting me instead use computeIfAbsent which is more readable and can even be thread-safe when used with ConcurrentHashMap as an implementation.

## Not using a build tool
Surprisingly, this is a common occurrence I have seen with beginners and understandably so. Java has few widely used build tools like Maven, Gradle and Ant and they have bit of learning curve before you can start to use them effectively.
Build tools help you with 
- Standard project structure - It becomes easier for others to understand and start working in your projects and easily import in their development environment.
- Dependency management - Since Java's ecosystem and the set of available third party libraries is a major reason for it's success. It becomes evident that you will be using lot of libraries and the dependencies need to be managed well and which can be done easily with a build tool. Build tools help you to manage their versions and resolve conflicts when same dependencies are used by multiple sources in your 

## Coding mistakes
Now let's get back to some of the coding mistakes that Java programmers make.

1. Equals/hashCode - An hotspot for many bugs seen in several Java applications which is due to missing or incorrect implementations of these methods in their own classes. This would result in incorrect equality checks for collections and map lookups.
    - Either these methods are not overridden
    - Or they are using variables whose values can be mutated even after object creation.
    - Do not use @Override tag and instead use same class parameter. This doesn't override the intended equals method.

Please read the documentation for [equals](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#equals-java.lang.Object-) and [hashcode](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#hashCode--)

2. Using APIs which are not thread-safe in a concurrent environment.
Some e.g.s below
 - [SimpleDateFormat](https://docs.oracle.com/javase/7/docs/api/java/text/SimpleDateFormat.html) - You need either create new object every time you have parse/format or use [threadlocal](https://docs.oracle.com/javase/7/docs/api/java/lang/ThreadLocal.html)
 - [HashMap](https://docs.oracle.com/javase/7/docs/api/java/util/HashMap.html) - Using HashMap can cause infinite loops in case multiple threads try to insert concurrently. This happens when the Hashmap is resizing.
 - Iterating over a fail-fast iterators can result in ConcurrentModificationException


3. Unrestricted access specifiers - Java has multiple access specifiers for its classes, 
  - Using public access specifiers for field variables.
  - Starting with default
  - Not using final for variables whose value is assigned only once. 

4. Generics - 
  - Incorrect use of wildcards.

5. Exception Handling - Java exception handling has been divided into two parts - checked and unchecked exceptions. This is a very common sight that 
    - For programmers to catch an exception and not to do anything with it. [Hiding the problem under the carpet]

6. Concurrency - 
  - Not handling exceptions when running tasks in a threadpool.
  - Using data structures which are not thread-safe
  - Using primitive or volatile variable for concurrent calculations. Volatile variables do not provide atomic guarantees for your calculations. You need to either make your code synchronized or use Atomic variables for same.

7. I/O
 - 

Using final for collections which are meant to be immutable

## Design problems
1. Too many Singletons - Singleton restricts the creation of objects to only one instance. They are meant to be used when dealing with resources  (e.g. DB Connections, Sockets, etc.). There are obvious usecases where it should be used singleton classes are useful, it is far too common to see programmers overuse this design patterns.
Singletons create a global state. In a multi-threaded program this can lead to race conditions. Singletons is also a problem when writing unit test codes, as test cases require clean initial state to being with.

2. Mutable data structures - Often our intent is not change the state of a collection/object during the runtime. Java provides some ways in which you can 
