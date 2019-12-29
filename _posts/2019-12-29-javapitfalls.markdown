---
layout: post
title:  "Java Pitfalls"
date:   2019-12-29 17:52:52 +0530
categories: Java-Pitfalls Java  Java-Mistakes
---
Java is an [extensively](https://www.tiobe.com/tiobe-index/java/) used programming language over last two decades and it is likely to stay relevant at least for some more, because a huge number of commercial and open source projects has been using it.
This article will try to point out some of the things that beginners/intermediate level programmers need to be aware of while working with Java. Some of them are straight out obvious while few of them would be subject to personal preferences. 

## Common coding mistakes
Let’s go over some common coding mistakes that Java programmers make.

### 1. Equals/hashCode - Missing or buggy implementation
A hotspot for many bugs seen in several Java applications which is due to missing or incorrect implementations of these methods in the implemented classes. This would result in incorrect equality checks for collections and map lookups.
Mistakes made:
- These methods are not overridden - Java will use the default Object's implementation of equals/hashCode method and that would result in bugs when the objects of these classes are used in collections.
- Using variables whose values can be mutated even after object creation - If we are using any variables then it could result in different return values for the same object which would result in incorrect equality checks.
- Does not use @Override tag and instead use same class parameter. This doesn’t override the intended equals method. If they are annotated with Override tag then Java compiler will not compile due to incorrect method parameters.
{% highlight java %}
public class EqualsExample {
  //Bug as it incorrectly overrides
  public boolean equals(EqualsExample equalsObject) {
 boolean isEquals = true;
    //some logic
    return isEquals;
}

  //Correctly Overridden - and always use the Override annotation when overriding methods
  @Override
  public boolean equals(Object equalsObject) {
    boolean isEquals = true;
    //some logic
    return isEquals;
  }
}
{% endhighlight %}

Please read the documentation for [equals](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#equals-java.lang.Object-) and [hashCode](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#hashCode--) for a detailed specifications of both of these methods accordingly.

### 2. Thread-safety - Using APIs which are not thread-safe in multi-threaded program
While using an existing APIs some developers often overlook the warnings in the documentation with respect to thread-safety. Some common examples are as below

- [SimpleDateFormat](https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html) - You need either create new object every time you have parse/format or use threadlocal
{% highlight java %}
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class SimpleDateFormatExample {
       private final SimpleDateFormat dateFormatUnsafe = new SimpleDateFormat("dd-MMM-yyyy");
    private final ThreadLocal<SimpleDateFormat> dateFormatSafe = new ThreadLocal<SimpleDateFormat>() {
        @Override
        protected SimpleDateFormat initialValue() {
            return new SimpleDateFormat("dd-MMM-yyyy");
        }
    };

    //No thread-safe
    public Date parseBuggy(String dateStr) throws ParseException {
        return dateFormatUnsafe.parse(dateStr);
    }

    //thread-safe and efficient
    public Date parseSafe(String dateStr) throws ParseException {
        return dateFormatSafe.get().parse(dateStr);
    }

    //thread-safe but inefficient as new formatter object is created for every parsing call
    public Date parseAlsoSafe(String dateStr) throws ParseException {
        return new SimpleDateFormat("dd-MMM-yyyy").parse(dateStr);
    }
}
{% endhighlight %}
- [HashMap](https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html) - Using HashMap can cause infinite loops in case multiple threads try to insert concurrently. This happens when the Hashmap is resizing its internal data structures to increase the HashMap capacity. Most beginners/intermediate programmers are unaware of this and generally face this first time during a production environment by when it will be too late.
{% highlight java %}
import java.util.HashMap;
import java.util.Map;

public class MapExample {

    public final Map<String, String>  hashMap = new HashMap<String, String>();
    public final Map<String, String> safeMap = new HashMap<String, String>();

    //Not thread-safe
    public void addToMap(String k, String v) {
        hashMap.put(k, v);
    }

    //thread-safe
    public void addToSafeMap(String k, String v) {
        safeMap.put(k, v);
    }
}
{% endhighlight %}
- Iterating over a fail-fast iterators can result in ConcurrentModificationException

### 3. Unrestricted access specifiers
Java has multiple access specifiers for its classes and people often use incorrect ones in their programs

- Using public access specifiers for field variables.
- Starting with default. If you do not provide any access specifiers then it uses default which less restrictive then private
- Not using final for variables whose value is assigned only once.
- Marking a variable as static when it should have been a member variable
{% highlight java %}
public class AccessSpecifiersExample {
   public String badAccessSpecifier = "Everyone Can Access Me";
   private String good = "Only my owner can access me";
   protected String stillGood = "Only my owner and their children can access me";

    //If the value is initialised only once, ensure they are final as a best practice
   private final String best = "Only my owner can access me";
}
{% endhighlight %}

### 4. Exception Handling
Java exception handling has been divided into two parts - checked and unchecked exceptions.
- This is a very common sight that for a programmer to catch an exception and not to do anything with it. Depending on the usec ase you should either log the exception or throw it further.
There are few scenarios where you would not want to do anything with it.
- Catching Throwable - [Throwable](https://docs.oracle.com/javase/8/docs/api/java/lang/Throwable.html) is root class which has both Errors and Exceptions extending from it.
While catching exceptions and dealing with it is required, you almost never catch an Error has it generally implies something bad happened and program should be stopped (e.g. OutOfMemory).


### 5. Floating point comparison
While this is not restricted to Java as the computer hardware representation for floating points has limited precision and results in minor precision errors when storing and calculating.
Care has to be taken to ensure that the epslion error is handled while comparing double variables. There are already available DoubleUtils classes in many open source project and they should be used if you already their dependencies included.
e.g. Google's [Guava libraries](https://github.com/google/guava/blob/master/guava/src/com/google/common/math/DoubleUtils.java)

### 6. Concurrency
- Not handling exceptions when running tasks in a threadpool.
- Using data structures which are not thread-safe (e.g. HashMap, LinkedList, ArrayList)
- Using primitive or volatile variable for concurrent calculations. Volatile variables do not provide atomic guarantees for your calculations. You need to either make your code synchronized or use Atomic variables for same.

{% highlight java%}
import java.util.concurrent.atomic.AtomicInteger;

public class ConcurrentIncrementExample {

    private int primitiveCounter = 0;
    private volatile int volatileCounter = 0;
    private AtomicInteger atomicInteger = new AtomicInteger(0);

    //Thread unsafe
    public void addToPrimitive(int add) {
        primitiveCounter += add;
    }

    //Thread unsafe
    public void addToVolatile(int add) {
        volatileCounter += add;
    }

    //Thread-safe
    public void addToAtomic(int add) {
        atomicInteger.addAndGet(add);
    }
}
{%endhighlight%}

### 7. I/O
- **Not closing Network/File/DB resources** - Whenever you are connecting to a network, connect to DB(which is also over the network), access a file you are required to free resources once the processing is complete. These actions consume a OS level resource which are limited in numbers. When they are not closed it can lead to these resources not being available to your program. An example is “Too many open files” because everything in linux is a file. Additionally close the resources in the finally block or use try with finally which is available since Java 7


## Design Mistakes
### Too many Singletons
Singleton restricts the creation of objects to only one instance. They are meant to be used when dealing with resources (e.g. DB Connections, Sockets, etc.). There are obvious use cases where it should be used singleton classes are useful, it is far too common to see programmers overuse this design patterns. Singletons create a global state. In a multi-threaded program this can lead to race conditions. Singletons is also a problem when writing unit test codes, as test cases require clean initial state to being with

### Mutable data structures
Often our intent is not change the state of a collection/object during the runtime. Java provides some ways in which you can do easily.

### Using exceptions for business logic
While in some rare cases it may be required, using too much exceptions for logic flow is generally a bad design pattern. Please note exceptions are expensive to handled by JVM and using them to define your program logic might slow your execution, while also making your code very difficult to read.

## Other problems
### Not using an IDE
As a first time programmer learning java, some tutorials would guide you through compiler and run command tools like javac, java. While it may be the right approach for your first couple of programs very soon you will start to feel the pinch of typing the code in your editor and then going back to javac to see if runs or not. The bigger the project, the more cumbersome it becomes and hence you need start using IDEs. There are several benefits writing code in IDEs in Java

- Java is a statically typed language and the IDEs point out errors and suspicious code while you are typing them. This alone would save you huge amount of time in the long run.
- Java IDEs can refactor the code far smartly and almost completely error free in most cases.
- There is another huge benefit of IDEs that it would suggest you improvements over your code and in a sense that becomes your guide. Below is an example with Intellij idea. My intent is write method which inserts into map if there is no element found in the map. Intellij Idea is suggesting me instead use [computeIfAbsent](https://docs.oracle.com/javase/8/docs/api/java/util/Map.html#computeIfAbsent-K-java.util.function.Function-) which is more readable and can even be thread-safe when used with [ConcurrentHashMap](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentHashMap.html) as an implementation.

Another e.g. for below code intellij suggests to use merge method
{% highlight java%}
public class IntellijAssistExample {

    public final Map<String, Integer> hashMap = new ConcurrentHashMap<>();

    public void incrementCounter(String k) {
        Integer h = hashMap.get(k);
        if(h == null) {
            hashMap.put(k, 1);
        } else {
            hashMap.put(k, h + 1);
        }
    }
}
{% endhighlight %}

Here is the suggestion from intellij for the above code.
![Intellij Sugestion](/images/intellij_assist.png){:class="img-responsive"}
{% highlight java%}
public class IntellijAssistExample {

    public final Map<String, Integer> hashMap = new ConcurrentHashMap<>();
    // After accepting the suggestion
    public void incrementCounter(String k) {
        hashMap.merge(k, 1, Integer::sum);
    }
}
{% endhighlight %}

### Not using a build tool
Surprisingly, this is a common occurrence I have seen with beginners and understandably so. Java has few widely used build tools like Maven, Gradle and Ant and they have bit of learning curve before you can start to use them effectively.

Build tools help you with
- **Standard project structure** - It becomes easier for others to understand and start working in your projects and easily import in their own development environment without a major configuration effort.
- **Dependency management** - Since Java’s ecosystem and the set of available third party libraries is a major reason for it’s success. It becomes evident that you will be using lot of libraries and the dependencies need to be managed well and which can be done easily with a build tool. Build tools help you to manage their versions and resolve conflicts when same dependencies are used by multiple sources in your

### Coming over from other languages
This generally happens when a proficient programmer of non-Java languages is learning/working on Java, they try to use the styles, methods of their known language in Java ecosystem.
This might make your project difficult to read for other programmers.