# Effective Reading Note Chapter 3

[TOC]



## Chapter 3. Methods Common to All Objects 

This chapter tells you when and how to override the nonfinal `Object` methods. The `finalize` method is omitted from this chapter because it was discussed in [Item 8](https://learning.oreilly.com/library/view/effective-java-3rd/9780134686097/ch2.xhtml#lev8). While not an `Object` method, `Comparable.compareTo` is discussed in this chapter because it has a similar character.



### Item 10: Obey the general contract when overriding equals 

#### 1. When you don't need to override the `equals`

Overriding the `equals` method seems simple, but there are many ways to get it wrong, and consequences can be dire. **The easiest way to avoid problems is not to override the `equals`method, in which case each instance of the class is equal only to itself.** This is the right thing to do if any of the following conditions apply:

• **Each instance of the class is inherently unique.** This is true for classes such as `Thread` that represent active entities rather than values. The `equals` implementation provided by `Object` has exactly the right behavior for these classes.

• **There is no need for the class to provide a “logical equality” test.** For example, `java.util.regex.Pattern` could have overridden `equals` to check whether two `Pattern` instances represented exactly the same regular expression, but the designers didn’t think that clients would need or want this functionality. Under these circumstances, the `equals` implementation inherited from `Object` is ideal.

• **A superclass has already overridden** `**equals**`, **and the superclass behavior is appropriate for this class.** For example, most `Set` implementations inherit their `equals`implementation from `AbstractSet`, `List` implementations from `AbstractList`, and `Map`implementations from `AbstractMap`.

• **The class is private or package-private, and you are certain that its** `**equals**`**method will never be invoked.** If you are extremely risk-averse, you can override the `equals` method to ensure that it isn’t invoked accidentally:

#### 2. When is it appropriate to override `equals` ? 

It is when a class has a notion of logical equality that differs from mere object identity and a superclass has not already overridden `euqals`. This is generally the case for value classes. A value class is simply a class that represents a value, such as `Integer` or `String`. **A programmer who compares references to value objects using the `equals` method expects to find out whether they are logically equivalent, not whether they refer to the same object. ** Not only is overriding the `equals` method necessary to satisfy programmer expectations, it enables instances to serve as map keys or set elements with predictable, desirable behavior. 

#### 3. Why we need to override the `equals` method? 

- Programmer expectations: check the highlighted sentence above. 
- Enable instances to serve as map keys or set elements with predictable, desirable behavior. 



One kind of value class that does not require the `equals` method to ve overridden is a class that uses instance control to ensure that at most one object exists with each value. Enum type fall into this category. For these classes, logical equality is the same as object identity, so `Object`'s `euqals` method functions as a logical `equals` method. 

#### 4. Adhere to its general contract when override the `equals` method

- *Reflexive*: For any non-null reference value `x`, `x.equals(x)` must return `true`. (自反性)
- *Symmetric*: For any non-null reference value `x` and `y`, `x.equals(y)` must return `true` if and only if `y.equals(x)` returns `true`. 
- *Transitive*: For any non-null reference values `x, y, z`, if `x.equals(y)` return `true` and `y.equals(z)` returns `true`, then `x.equals(z)` must return `true`. 
- *Consistent*: For any non-null reference values `x` and `y`, multiple invocations of `x.equals(y)` must consistently return true or consistently return `false`, provided no information used in `equals` comparisons is modified. 
- Fo any non-null reference value `x`, `x.equals(null)` must return `false`. 

#### 5. Suggestions in comparison 

- **Use the == operator to check if the argument is a reference to this object.**
  - If so, return `true`. This is just a performance optimization but one that is worth doing ifthe comparison is potentially expensive. 
- **Use the** `**instanceof**` **operator to check if the argument has the correct type.**
  -  If not, return `false`. Typically, the correct type is the class in which the method occurs. Occasionally, it is some interface implemented by this class. Use an interface if the class implements an interface that refines the `equals` contract to permit comparisons across classes that implement the interface. Collection interfaces such as `Set`, `List`, `Map`, and `Map.Entry`have this property.
- **Cast the argument to the correct type.** 
  - Because this cast was preceded by an `instanceof` test, it is guaranteed to succeed.
- **For each “significant” field in the class, check if that field of the argument matches the corresponding field of this object.** 
  - If all these tests succeed, return `true`; otherwise, return `false`. If the type in Step 2 is an interface, you must access the argument’s fields via interface methods; if the type is a class, you may be able to access the fields directly, depending on their accessibility.

> **CAUTION**:
>
> For primitive fields whose type is not `float` or `double`, use the `==` operator for comparisons; for object reference fields, call the `equals` method recursively; for `float` fields, use the static `Float.compare(float, float)` method; and for `double` fields, use `Double.compare(double, double)`. The special treatment of `float` and `double` fields is made necessary by the existence of `Float.NaN`, `-0.0f` and the analogous `double` values;
>
> While you could compare `float` and `double` fields with the static methods `Float.equals` and `Double.equals`, this would entail autoboxing on every comparison, which would have poor performance. For array fields, apply these guidelines to each element. If every element in an array field is significant, use one of the `Arrays.equals` methods.

#### 6. Warning 

- **Always override** `**hashCode**` **when you override** `**equals**` ([Item 11](https://learning.oreilly.com/library/view/effective-java-3rd/9780134686097/ch3.xhtml#lev11)).

- **Don’t try to be too clever.** If you simply test fields for equality, it’s not hard to adhere to the `equals` contract. If you are overly aggressive in searching for equivalence, it’s easy to get into trouble. It is generally a bad idea to take any form of aliasing into account. For example, the `File` class shouldn’t attempt to equate symbolic links referring to the same file. Thankfully, it doesn’t.

- **Don’t substitute another type for** `**Object**` **in the** `**equals**` **declaration.** It is not uncommon for a programmer to write an `equals` method that looks like this and then spend hours puzzling over why it doesn’t work properly:
- **Consistent use of the `Override` annotation,** as illustrated throughout this item, will prevent you from making this mistake ([Item 40](https://learning.oreilly.com/library/view/effective-java-3rd/9780134686097/ch6.xhtml#lev40)). This `equals` method won’t compile, and the error message will tell you exactly what is wrong:

#### 7. Suggesstion 

Writing and testing `equals` (and `hashCode`) methods is tedious, and the resulting code is mundane. An excellent alternative to writing and testing these methods manually is to use Google’s open source AutoValue framework, which automatically generates these methods for you, triggered by a single annotation on the class . In most cases, the methods generated by AutoValue are essentially identical to those you’d write yourself.

#### 8. Summary 

In summary, don’t override the `equals` method unless you have to: in many cases, the implementation inherited from `Object` does exactly what you want. If you do override `equals`, make sure to compare all of the class’s significant fields and to compare them in a manner that preserves all five provisions of the `equals` contract.



### Item 11. Always override `HASHCODE` when you override `EQUALS`

#### 1. You must override `hashCode` in every class that overrides `equals`.

If you fail to do so, your class will violate the general contract for `hashCode`, which will prevent it from functioning properly in collections such as `HashMap` and `HashSet`. Here is the contract, adapted from the `Object` specification: 

- When the `hashCode` method is invoked on an object repeatedly during an execution of an application, it must consistently return the same value, provided no information used in `equals` comparisons is modified. This value need not retain consistent from one execution of an application to another. 
- If two objects are equal according to the `equals (Object)` method, then calling `hashCode` on the two objects must produce the same integer result. 
- If two objects are unequal according to the `equals (Object)` method, it is not required that calling `hashCode` on each of the objects must produce distinct results. However, the programmer should be aware that producing distinct results for unequal objects may improve the performance of hash tables. 

#### 2. What will happen if you fail to override `hashCode` when you override `equals`

**The key provision that is violated when you fail to override `hashCode` is the second one: equal objects must have equal hash codes.** Two distinct instances may be logically equal according to a class's `equals` method, but to `Object`'s `hashCode` method, they're just two objects with nothing much in common. Therefore, `Object`'s `hashCode` method returns two seemingly random numbers instead of two equal numbers as required by the contract. 

For example, suppose you attempt to use instance of the `PhoneNumber` class from Item 10 as keys in a `HashMap`: 

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(nuew PhoneNumber(707, 867, 5309), "Jenny");
```

At this point, you might expect `m.get(new PhoneNumber(707, 867, 5309))` to return `"Jenny"`, but instead, it returns null. Notice that two `PhoneNumber` instances are involved: one is used for insertion into the `HashMap`, and a second, equal instance is used for (attempted) retrieval. The `PhoneNumber` class's failure to override `hashCode` causes the two equal instances to have unequal hash codes, in violation of the `hashCode` contract. Therefore, the `get` method is likely to look for the phone number in a different hash bucket from the one in which it was stored by the `put` method. Even if the two instances happen to hash to the same bucket, the `get` method will almost certainly return `null`, because `HashMap` has an optimization that caches the hash code associated with each entry and doesn't bother checking for object equality if the hash codes don't match. 

> Question: 
>
> How can java hash two logically equal objects to different hash code? 

#### 3. How to write a hash code function? 

A good hash function tends to produce unequal hash codes for unequal instances. This is exactly waht is meat by the third part of the `hashCode` contract. Ideally, a hash function should distribute any reasonable collection of unequal instances uniformly across all `int` values. Achieving this ideal can be difficult. Luckily it's not too hard to achieve a fair approximation. Here is a simple recipe: 

- Step 1: Declare an `int` variable named `result`, and initialize it to the hash code `c` for the first significant field in your object, as computed in step 2.a (Recall from Item 10 that a significant field is a field that affects equals comparisons.) 

- Step 2: For every remaining significant field `f` in your object, do the following: 

  - Step a: Compute an `int` hash code `c` for the field: 

    - step I: If the field is of a primitive type, compute `Type.hashCode(f)`, where `Type` is the boxed primitive class corresponding to f's type. 
    - step II: If the field is an object reference and this class's `equals` method compares the field by recursively invoking `equals`, recursively invoke `hashCode` on the field. If a more complex comparison is required, compute a "canonical representation" for this field and invoke `hashCode` on the canonical representation. If the value of the field is null, use 0 (or some other constant, but 0 is traditional). 
    - If the field is an array, treat it as if each significant element were a separate field. That is, compute a hash code for each significant element by applying these rules recursively, and combine the values per step 2.b. If the array has no significant elements, use a constant, preferably not 0. If all elements are significant, use `Arrays.hashCode`.

  - Step b: Combine the hash code `c` computed in step 2.a into `result` as follows: 

    `result = 31 * result + c;`

- Step 3: Return `result`. 



#### 4. Suggestions 

- Suggestion 1: Use  `com.google.common.hash.Hashing`
  - While the recipe in this item yields reasonably good hash functions, they are not state-of-the-art. They are comparable in quality to the hash functions found in the Java platform libraries' value types and are adequate for most uses. If you have a bona fide need for hash functions less likely to produce collisions, see Guava's `com.google.common.hash.Hashing`. 

- Suggestion 2: **Save hash code when appropriate**
  - **If a class is immutable and the cost of computing the hash code is significant**, you might consider caching the hash code in the object rather than recalculating it each time it is requested. 

- Suggestion 3: **If you believe that most objects of this type will be used as hash keys, then you should calculate the hash code when the instance is created.** 
- Suggestion 4: **Do not be tempted to exclude significant fields from the hash code computation to improve performance.** 
  - While the resulting hash function may run faster, its poor quality may degrade hash tables’ performance to the point where they become unusable. In particular, the hash function may be confronted with a large collection of instances that differ mainly in regions you’ve chosen to ignore. If this happens, the hash function will map all these instances to a few hash codes, and programs that should run in linear time will instead run in quadratic time.
  - This is not just a theoretical problem. Prior to Java 2, the `String` hash function used at most sixteen characters evenly spaced throughout the string, starting with the first character. For large collections of hierarchical names, such as URLs, this function displayed exactly the pathological behavior described earlier.
- Suggestion 5: **Don’t provide a detailed specification for the value returned by** `**hashCode**`**, so clients can’t reasonably depend on it; this gives you the flexibility to change it.**
  - Many classes in the Java libraries, such as `String` and `Integer`, specify the exact value returned by their `hashCode` method as a function of the instance value. This is *not* a good idea but a mistake that we’re forced to live with: It impedes the ability to improve the hash function in future releases. If you leave the details unspecified and a flaw is found in the hash function or a better hash function is discovered, you can change it in a subsequent release.
  - In summary, you *must* override `hashCode` every time you override `equals`, or your program will not run correctly. Your `hashCode` method must obey the general contract specified in `Object` and must do a reasonable job assigning unequal hash codes to unequal instances. This is easy to achieve, if slightly tedious, using the recipe on page 51. As mentioned in [Item 10](https://learning.oreilly.com/library/view/Effective+Java,+3rd+Edition/9780134686097/ch3.xhtml#lev10), the AutoValue framework provides a fine alternative to writing `equals` and `hashCode`methods manually, and IDEs also provide some of this functionality.



### Item 12: Always override `TOSTRING`

While `Object` provides an implementation of the `toString` method, the string that it returns is generally not what the user of your class wants to see. It consists of the class name followed by an "at" sign (@) and the unsigned hexadecimal representation of the hash code, for example, `PhoneNumber@163b91`. **The general contract for `toString`** syas that the returned string should be **"a concise but informative representation that is easy for a person to read."** While it could be argued that `PhoneNumber@163b91` is concise and easy to read, it isn’t very informative when compared to `707-867-5309`. The `toString` contract goes on to say, “It is recommended that all subclasses override this method.” Good advice, indeed!



#### 1. Benefits of override `toString` method

- **Providing a good** `**toString**` **implementation makes your class much more pleasant to use and makes systems using the class easier to debug**. 
  - The `toString`method is automatically invoked when an object is passed to `println`, `printf`, the string concatenation operator, or `assert`, or is printed by a debugger. Even if you never call `toString` on an object, others may. For example, a component that has a reference to your object may include the string representation of the object in a logged error message. If you fail to override `toString`, the message may be all but useless.
- **When practical, the** `**toString**` **method should return** ***all*** **of the interesting information contained in the object**
- One important decision you’ll have to make when implementing a `toString` method is **whether to specify the format of the return value in the documentation.**
  - **Pros**: It is recommended that you do this for *value classes*, such as phone number or matrix. The advantage of specifying the format is that it serves as a standard, unambiguous, human-readable representation of the object. This representation can be used for input and output and in persistent human-readable data objects, such as CSV files. If you specify the format, it’s usually a good idea to provide a matching static factory or constructor so programmers can easily translate back and forth between the object and its string representation. This approach is taken by many value classes in the Java platform libraries, including `BigInteger`, `BigDecimal`, and most of the boxed primitive classes.
  - **Cons**: The disadvantage of specifying the format of the `toString` return value is that once you’ve specified it, you’re stuck with it for life, assuming your class is widely used. Programmers will write code to parse the representation, to generate it, and to embed it into persistent data. If you change the representation in a future release, you’ll break their code and data, and they will yowl. By choosing not to specify a format, you preserve the flexibility to add information or improve the format in a subsequent release.
- **Whether or not you decide to specify the format, you should clearly document your intentions.** If you specify the format, you should do so precisely. 
- Whether or not you specify the format, **provide programmatic access to the information contained in the value returned by** `**toString**`**.** 
  - For example, the `PhoneNumber`class should contain accessors for the area code, prefix, and line number. If you fail to do this, you *force* programmers who need this information to parse the string. Besides reducing performance and making unnecessary work for programmers, this process is error-prone and results in fragile systems that break if you change the format. By failing to provide accessors, you turn the string format into a de facto API, even if you’ve specified that it’s subject to change.
- It makes no sense to write a `toString` method in a static utility class ([Item 4](https://learning.oreilly.com/library/view/effective-java-3rd/9780134686097/ch2.xhtml#lev4)). Nor should you write a `toString` method in most enum types ([Item 34](https://learning.oreilly.com/library/view/effective-java-3rd/9780134686097/ch6.xhtml#lev34)) because Java provides a perfectly good one for you. 
  - You should, however, write a `toString` method in any abstract class whose subclasses share a common string representation. For example, the `toString` methods on most collection implementations are inherited from the abstract collection classes.
- Google’s open source AutoValue facility, discussed in [Item 10](https://learning.oreilly.com/library/view/effective-java-3rd/9780134686097/ch3.xhtml#lev10), will generate a `toString`method for you.



### ITEM 13: OVERRIDE `CLONE` Judiciously

#### 1. What *does* `Cloneable` do?

Given that it contains no methods? It determines the behavior of `Object`’s protected `clone` implementation: if a class implements `Cloneable`, `Object`’s `clone` method returns a field-by-field copy of the object; otherwise it throws `CloneNotSupportedException`. This is a highly atypical use of interfaces and not one to be emulated. Normally, implementing an interface says something about what a class can do for its clients. In this case, it modifies the behavior of a protected method on a superclass.

#### 2. Java supports *covariant return types*. In other words, an overriding method’s return type can be a subclass of the overridden method’s return type. 



### Item 14: Consider Implementing `COMPARABLE`

Unlike the other methods discussed in this chapter, the `compareTo` method is not declared in `Object`. Rather, it is the sole method in the `Comparable` interface. It is similar in character to `Object`'s `equals` method, except that it permits order comparisons in addition to simple equality comparisons, and it is generic.  By implementing `Comparable`, a class indicates that its instance have **natural ordering**. Sorting an array of objects that implement `Comparable` is as simple as this: 

```java
Arrays.sort(a);
```

It is similarly easy to search, compute extreme values, and maintain automatically sorted collections of `Comparable` objects. 



#### 1. Benefits 

By implementing `Comparable`, you allow your class to interoperate with all of the many generic algorithms and collection implementations that depend on this interface. You gain a tremendous amount of power for a small amount of effort. Virtually all of the value classes in the Java platform libraries, as well as enum types, implement `Comparable`. If you are writing a value class with an natural ordering, such as alphabetical order, numerical order, or chronological order, you should implement the `Comparable` interface. 



#### 2. General contract of hte `compareTo` method 

Compares this object with the specified object for order. Returns a negative integer, zero, or a positive integer as this object is less than, equal to, or greater than the specified object. Throws `ClassCastException` if the specified object's type prevents it from being compared to this object. 

- The implementor must ensure that `sgn(x.compareTo(y)) == -sgn(y.compareTo(x))` for all `x` and `y`.  (This implies that `x.compareTo(y)` must throw an exception if and only if `y.compareTo(x)` throws an exception.)
- The implementor must also ensure that the relation is transitive: `(x. compareTo(y) > 0 && y.compareTo(z) > 0)` implies `x.compareTo(z) > 0`.
- Finally, the implementor must ensure that `x.compareTo(y) == 0` implies that `sgn(x.compareTo(z)) == sgn(y.compareTo(z))`, for all `z`.
- It is strongly recommended, but not required, that `(x.compareTo(y) == 0) == (x.equals(y))`. Generally speaking, any class that implements the `Comparable` interface and violates this condition should clearly indicate this fact. The recommended language is “Note: This class has a natural ordering that is inconsistent with `equals`.”

#### 3. Fail to implement `compareTo` will affect corresponding collections 

Just as a class that violates the `hashCode` contract can break other classes that depend on hashing, a class that violates the `compareTo` contract can break other classes that depend on comparison. Classes that depend on comparison include the sorted collections `TreeSet` and `TreeMap` and the utility classes `Collections` and `Arrays`, which contain searching and sorting algorithms.

#### 4. Suggestions 

- Do not use `>` and `<`
  - Prior editions of this book recommended that `compareTo` methods compare integral primitive fields using the relational operators `<` and `>`, and floating point primitive fields using the static methods `Double.compare` and `Float.compare`. In Java 7, static `compare` methods were added to all of Java’s boxed primitive classes. **Use of the relational operators** `**<**`**and** `**>**` **in** `**compareTo**` **methods is verbose and error-prone and no longer recommended.**
- Comparing fields in the significant order 
  - If a class has multiple significant fields, the order in which you compare them is critical. Start with the most significant field and work your way down. If a comparison results in anything other than zero (which represents equality), you’re done; just return the result. If the most significant field is equal, compare the next-most-significant field, and so on, until you find an unequal field or compare the least significant field. Here is a `compareTo` method for the `PhoneNumber` class in [Item 11](https://learning.oreilly.com/library/view/effective-java-3rd/9780134686097/ch3.xhtml#lev11) demonstrating this technique:
- **Using `Comparator` interfaces provided after Java 8**
  - In Java 8, the `Comparator` interface was outfitted with a set of *comparator construction methods*, which enable fluent construction of comparators. These comparators can then be used to implement a `compareTo` method, as required by the `Comparable` interface.  





































