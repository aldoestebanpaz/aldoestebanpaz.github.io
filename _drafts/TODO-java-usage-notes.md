- [Java usage notes](#java-usage-notes)
  - [Language features](#language-features)
    - [Interfaces](#interfaces)
      - [Default methods](#default-methods)
    - [Functional Interfaces](#functional-interfaces)
      - [The @FunctionalInterface annotation](#the-functionalinterface-annotation)

# Java usage notes

## Language features

### Interfaces

In an interface, we're allowed to use:
- constants variables
- abstract methods
- static methods
- default methods

We also should remember that:
- all interface declarations should have the public or default access modifier; the abstract modifier will be added automatically by the compiler.
- an interface method can't be protected or final.
- up until Java 9, interface methods could not be private; however, Java 9 introduced the possibility to define private methods in interfaces.
- interface variables are public, static, and final by definition; we're not allowed to change their visibility.

TODO

#### Default methods

A Default method is a feature introduced by Java 8 that is like an abstract method, but optional.

Traditional interfaces in Java 7 and below don't offer backward compatibility.

What this means is that if you have legacy code written in Java 7 or earlier, and you decide to add an abstract method to an existing interface, then all the classes that implement that interface must override the new abstract method. Otherwise, the code will break.

Java 8 solved this problem by introducing the default method that is optional and can be implemented at the interface level.

### Functional Interfaces

Functional Interfaces are additionally recognized as Single Abstract Method Interfaces (In short, SAM interfaces).

A functional interface is an interface that contains only one abstract method (that is, they can have only one functionality to exhibit). However, they can include any quantity of default and static methods.

Runnable, ActionListener, Comparable are some of the examples of functional interfaces.

Some examples of functional interfaces are the following:
- Runnable - This interface only contains the run() method.
- Comparable - This interface only contains the compareTo() method.
- ActionListener – This interface only contains the actionPerformed() method.
- Callable – This interface only contains the call() method.

Note that in Functional interfaces, there is no need to use the `abstract` keyword as it is optional because, by default, the method defined inside the interface is abstract only.

**Implementing Functional Interfaces**

Before Java 8, we had to create anonymous inner class objects or implement these interfaces.

Example:

```java
class Test {
    public static void main(String args[])
    {
        new Thread(
            // create anonymous inner class object
            new Runnable() {
                @Override
                public void run() {
                    System.out.println("New thread created");
                }
            }
        ).start();
    }
}
```

**Implementing Functional Interfaces using Lambda Expressions**

From Java 8 onwards, lambda expressions can be used to represent the instance of a functional interface.

```java
class Test {
    public static void main(String args[])
    {
        new Thread(
            // lambda expression to create the object
            () -> {
                System.out.println("New thread created");
            }
        ).start();
    }
}
```

#### The @FunctionalInterface annotation

@FunctionalInterface annotation is used to ensure that the functional interface can't have more than one abstract method. In case more than one abstract methods are present, the compiler flags an 'Unexpected @FunctionalInterface annotation' message. However, it is not mandatory to use this annotation.
