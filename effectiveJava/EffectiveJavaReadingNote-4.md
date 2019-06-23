# Effective Java Reading Note-4

[TOC]

## Item 15: Minimize the accessibility of classes and members 

### 1. Purpose of minimizing accessibility 

The single most import factor that distinguishes a well-designed component from a poorly designed one is the degree to which the component hides its internal data and other implementation details from other components. This concept, known as *information hiding* or *encapsulation*, is a fundamental tenet of software design. 

### 2. Why hiding information ?

- It decouples the components that comprise a system, allowing them to be developed, tested, optimized, used, understood, and modified in isolation. **This speeds up system development because components can be developed in parallel. **
- It eases the burden of maintenance because components can be understood more quickly and debugged or replaced with litle fear of harming other components. 
- It increases software resue because components that aren't tightly coupled often prove useful in other contexts besides the ones for which they were developed. 
- It decreases the risk in building large systems because individual components may prove successfully even if the system does not. 

### 3. Rule: Make each class or member as inaccessible as possible. 

### 4. Access levels for members (fields, methods, nested classes and nested interfaces)

- **`private`** - The member is accessible only from the top-level class where it is declared. 
- **`package private`** - The member is accessible from any class in the package where it is declared. Technically known as *default* access.
- **`protected`**—The member is accessible from subclasses of the class where it is declared (subject to a few restrictions [JLS, 6.6.2]) and from any class in the package where it is declared.
-  **`public`**—The member is accessible from anywhere.



### 5. Suggestions 

-  If a package-private top-level class or interface is **used by only one class**, consider making the top-level class a private static nested class of the sole class that uses it (Item 24). 

- It is super important to reduce the accessibility of a gratuitously (*unjustifiably / without good reason*) public class. 

- For members of public classes, a huge increase in accessibility occurs when the access level goes from package-private to protected. **A protected member is part of the class's exported API and must be supported forever.** The need for protected members should be relatively rare. 

- Instance fields of public classes should rarely be public. 

  - If an instance field is nonfinal or is a reference to a mutable object, then by making it public, you give up the ability to limit the values that can be stored in the field. This means you give up the ability to enforce invariants involving the field. Also, you give up the ability to take any action when the field is modified, so **classes with public mutable fields are not generally thread safe.**

- A field containing a reference to a mutalbe object has all the disadvantages of a nonfinal field. While the reference cannot be modified, the referenced object can be modified — with disastrous results. 

- Note that a nonzero-length array is always mutable, **so it is wrong for a class to have a public static final array field, or an accessor that returns such a field.**

  - If a class has such a field or accessor, clients will be able to modify the contents of the array. This is a frequent source of security holes. 

  - You can solve the above proble with the following two ways: 

    - Make the public array private and use a public immutable list. 

    - ```java
      // Potential security hole!
      public static final Thing[] VALUES = { ..... };
      
      // Fix 
      private static final Thing [] PRIVATE_VALUES = { ..... };
      public static final List<Thing> VALUES = Collections.unmodifiableLIst(Arryas.asList(PRIVATE_VALUES));
      ```

    - Make the array private and add a public method that returns a copy of a private array: 

    - ```java
      // Potential security hole!
      private static final Thing[] PRIVATE_VALUES = { ..... };
      public static final Thing[] values() {
        return PRIVATE_VALUES.clone();
      }
      ```

- With the exception of public static final fields, which serve as constants, public classes should have no public fields. Ensure that objects referenced by public static final fields are immutable.



## Item 16: In public classes, use accessor methods, not public fields

Public classes should never expose mutable fields. It is less harmful, though still questionable, for public classes to expose immutable fields. It is, however, sometimes desirable for package-private or private nested classes to expose fields, whether mutable or immutable.



## Item 17: Minimize mutability

An immutable class is simple a class whose instances cannot be modified. Examples are `String`, `BigInteger` and `BigDecimal`. There are many good reasons for this: Immutable classes are easier to design , implement, and use than mutable classes. They are less prone to error and are more secure. 



### 1. Rules to follow in making a class immutable

- **Don't provide methods that modify the objects's state** (known as mutators, like set method). 
- **Ensure that the class can't be extended**. 
  - This prevents careless or malicious subclasses from compromising the immutable behavior of the class by behaving as if the object's state has changed. Preventing subclassing is generally accomplished by making the class final, but there is an alternative that we'll discuss later. 
- **Make all fields final**. 
- **Make all fields private.** 
  - This prevents clients from obtaining access to mutable objects referred to by fields and modifying these objects directly. While it is technically permissible for immutable classes to have public final fields containing primitive values or references to immutable objects, it is not recommended because it precludes changing the internal representation in a later release
- **Ensure exclusive access to any mutable components.** 
  -  If your class has any fields that refer to mutable objects, ensure that clients of the class cannot obtain references to these objects. Never initialize such a field to a client-provided object reference or return the field from an accessor. Make *defensive copies* ([Item 50](https://learning.oreilly.com/library/view/effective-java-3rd/9780134686097/ch8.xhtml#lev50)) in constructors, accessors, and `readObject` methods ([Item 88](https://learning.oreilly.com/library/view/effective-java-3rd/9780134686097/ch12.xhtml#lev88)).



### 2. Functional approach 

Functional approach are isntance methods that will not modify the states of an instance, instead it  will return the new result. 

#### 2.1 Benefits of functional approach (Immutable objects)

-  **Immutable objects are simple.** 

  - An immutable object can be in exactly one state, the state in which it was created. If you make sure that all constructors establish class invariants, then it is guaranteed that these invariants will remain true for all time, with no further effort on your part or on the part of the programmer who uses the class. Mutable objects, on the other hand, can have arbitrarily complex state spaces. If the documentation does not provide a precise description of the state transitions performed by mutator methods, it can be difficult or impossible to use a mutable class reliably.

- **Immutable objects are inherently thread-safe; they require no synchronization.**

  - They cannot be corrupted by multiple threads accessing them concurrently. This is far and away the easiest approach to achieve thread safety. Since no thread can ever observe any effect of another thread on an immutable object

-  **Immutable objects can be shared freely.**

  - Immutable classes should therefore encourage clients to reuse existing instances wherever possible. One easy way to do this is to provide public static final constants for commonly used values. For example, the `Complex` class might provide these constants:

  - ```java
    public static final Complex ZERO = new Complex(0, 0);
    public static final Complex ONE  = new Complex(1, 0);
    public static final Complex I    = new Complex(0, 1);
    ```

- **Not only can you share immutable objects, but they can share their internals.** 

  - For example, the `BigInteger` class uses a sign-magnitude representation internally. The sign is represented by an `int`, and the magnitude is represented by an `int` array. The `negate`method produces a new `BigInteger` of like magnitude and opposite sign. It does not need to copy the array even though it is mutable; the newly created `BigInteger` points to the same internal array as the original. (Since no one can modify the magnitude inside the original and the new negative big integer.)

- **Immutable objects provide failure atomicity for free** ([Item 76](https://learning.oreilly.com/library/view/effective-java-3rd/9780134686097/ch10.xhtml#lev76)). 

  - Their state never changes, so there is no possibility of a temporary inconsistency.



### 2.2 Problems of immutable objects and how to solve it

- **The major disadvantage of immutable classes is that they require a separate object for each distinct value.**
  - The first is to guess which multistep operations will be commonly required and to provide them as primitives.
- Recall that to guarantee immutability, a class must not permit itself to be subclassed. This can be done by making the class final, but there is another, more flexible alternative. Instead of making an immutable class final, you can make all of its constructors private or package-private and add public static factories in place of the public constructors ([Item 1](https://learning.oreilly.com/library/view/effective-java-3rd/9780134686097/ch2.xhtml#lev1)).



### 3. Suggestions 

- Classes should be immutable unless there is a very good reason to make them mutable. 
- If a class cannot be made immutable, limit its mutability as much as possible. 
- Declare every field `private final` unless there's a good reason to do otherwise. 
- Constructors should create fully initialized objects with all of their invariants established. 



## Item 18: Favor composition over inheritance 

It is safe to use inheritance within a package, where the subclass and the superclass implementations are under the control of the same programmers. It is also safe to use inheritance  when extending classes specifically designed and documented for extension. **Inheriting from ordinary concrete classes across package boundaries, however, is dangerous.**



### 1. Why not inheritance? 

-  Unlike method invocation, inheritance violates encapsulation. 
  - In other words, a subclass depends on the implementation details of its superclass for its proper function. The superclass's implementation may change from release to release, and if it does, the subclass may break, even though its code has not been touched. 



### 2. How to avoid 





































