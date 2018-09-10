## Effective Java - 3rd Edition Notes

### Chapter Index
- [ ] 02 - Creating and Destroying Objects
- [ ] 03 - Methods Common to All Objects
- [ ] 04 - Classes and Interfaces
- [ ] 05 - Generics
- [ ] 06 - Enums and Annotations
- [ ] 07 - Lambdas and Streams
- [ ] 08 - Methods
- [ ] 09 - General Programming
- [ ] 10 - Exceptions
- [ ] 11 - Concurrency
- [ ] 12 - Serialization

### Chapter 02 - Creating and Destroying Objects

#### Item 1 - Consider static factory methods instead of constructors
- traditional vs. flexible way of object instantiation
- example of static factory method:
```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```
##### Advantages and disadvantages:
- (PRO) static factories have names, unlike constructors
- (PRO) static factories are not required to create a new object on each invocation
- such classes are called _instance controlled_
  - enable singleton (Item 2) and non-instantiability (Item 3) guarantees
  - allows immutable value class to guarantee no two instances exist
  - forms the basis of [Flyweight](https://en.wikipedia.org/wiki/Flyweight_pattern#Example_in_Java) Pattern
  - Enum types provide this guarantee
- (PRO) static methods can return an object of any subtype of their return type, unlike constructors
- this can lead to compact APIs
- lends itself to _interface-based frameworks_ (Item 20)
- companion classes mostly obviated in Java 8
  - default methods
  - still some limitations which are dealt with in Java 9+
- (PRO) static factories allow the class of the returned object to vary from call to call as function of input params
- example: ```EnumSet```
  - backed by a ```long``` - ```RegularEnumSet```
  - backed by a ```long[]``` -  ```JumboEnumSet```
- (PRO) static factories do not require the class of returned object to exist when the class containing the method is written
- form the basis of _service provider frameworks_
- example: JDBC
- variants:
  - dependency-injection frameworks (i.e. Spring, Guice)
  - [Bridge](https://en.wikipedia.org/wiki/Bridge_pattern#Java) Pattern
- (CON) classes without public/protected constructors cannot be subclassed
- a blessing in disguise
  - encourages composition over inheritance
- (CON) hard to find in documentation
- current JavaDoc limitations

##### Some common names for static factory methods
- ```from()```
- ```of()```
- ```valueOf()```
- ```instance()``` or ```getInstance()```
- ```getType()```
- ```newType()```
- ```type()```

##### Conclusion
Avoid the reflex to provide a public constructor and consider static methods/factories. 

#### Item 2 - Consider a builder when faced with many constructor params
- static factories and constructors share a limitation: they scale poorly with the increase of (optional) params
- traditional ways of dealing with this:
  - _telescoping constructor_
    - also scales poorly: hard to write with many params and even harder to read
  - _JavaBeans_ pattern
    - allows inconsistency
    - precludes immutability (Item 17)
- a better way: [Builder](https://en.wikipedia.org/wiki/Builder_pattern#Java) pattern
  - typically a static member class (Item 24)
  - the client code is easy to write and, more importantly, easy to read
  - easy to incorporate validity checks
    - check on object fields after copying params from the builder (Item 50)
    - failing check will throw an ```IllegalArgumentException``` (Item 72) with exception details (Item 75)
  - well suited to class hierarchies
    - generic builder with recursive type parameters (Item 30) can construct any subclass
    - abstract ```self()``` simulates self-type which, combined with covariant return typing, obviates need for casting
  - flexible
- disadvantages:
  - cost of creating a builder
  - more verbose than a telescoping constructor
- conclusion:
  - almost always start with a builder in the first place
    - especially so if we have more than a handful params
    - client code much easier to read and write than telescoping constructors
    - builder are much safer than JavaBeans   
     
#### Item 3 - Enforce the singleton property when a private constructor or enum type
- _singleton_ is a class that is instantiated exactly once
- making a class a singleton can make it difficult to test
- three common ways of implementing it:
  - with ```public final``` field
  - with static factory
  - with a single-element enum (preferred)
- conclusion:
  - if a singleton is indeed warranted, create it as a single-element enum
    - well-defended against reflection
    - solves serialization problems

#### Item 4 - Enforce non-instantiability with a private constructor
- occasionally, we want to write a class that is simply a grouping of static methods and static fields
- such _utility classes_ have acquired a bad reputation due to their abuse in avoiding thinking in terms of objects but they have valid uses:
  - group related methods on primitive values or arrays
    - examples: ```java.util.Math``` or ```java.util.Arrays```
  - group static methods, including factories (Item 1)
    - default methods are also available (providing we own the interface)
  - group methods on a final class, since we can't put them in a subclass
- make such classes non-instantiable
  - provide an explanatory comment
  - throw an ```AssertionError``` from constructor instead of empty one, to guard against accidental constructions from within the class
  - as a side-effect, this class is now effectively final
    - the class cannot be subclassed as there are no available constructors
    - still, it is good to document this a make the class itself ```final```

#### Item 5 - Prefer dependency injection to hard-wiring resources
- do not use static utility methods or singletons to handle creations of class' resources
  - these yield inflexible and untestable classes
- favour dependency injection by supplying the required resource parametrically
  - we inject (pass) the dependency (resource) into the class that requires it
  - testable
  - flexible (esp. with [Factory Method](https://en.wikipedia.org/wiki/Factory_method_pattern#Java) pattern)
    - ```Supplier<T>``` is perfectly suited for representing factories
    - methods that take this interface should typically constrain the factory's type parameter with a bounded wildcard type (Item 31)
      - the client should be able to pass in a factory that requires any subtype of a specified type
- the manual dependency injection can be automatised with frameworks

#### Item 6 - Avoid creating unnecessary objects
- creating unnecessary objects can be avoided by using static factory methods (Item 1)
- some object creations are much more expensive than others
  - it may be advisable to cache such objects
  - example: regex matching
- immutable objects can trivially be reused
- other examples are [Adapter](https://en.wikipedia.org/wiki/Adapter_pattern#Java)s (a.k.a. _views_)
  - adapter is an object delegating to a backing object, providing an alternative interface
  - has not state beyond the backing object thus provides a view of it
  - example: ```keySet()``` in ```Map``` interface
- autoboxing can often subtly create unnecessary objects
- almost never create own _object pools_
  - JVM gc will almost always outperform such pools
  - exception to this: very expensive objects
    - example: database connection objects
- counterpoint to this is _defencive copying_ (Item 50)

#### Item 7 - Eliminate obsolete object references
- when relying on automatic gc, be wary of leaks
  - example: a resizing-array stack that doesn't null out its references on ```pop()```
- common sources of leaks
  - classes that manage their own memory
  - caches
    - can be mitigated with ```WeakHashMap```
    - sophisticated caches might need to use ```java.lang.ref``` directly
  - listeners and other callbacks
    - APIs that register callbacks but don't deregister them explicitly
    - can be mitigated with ```WeakHashMap```
- conclusion: it very desirable to learn to anticipate such problems before they occur as they can be very costly to fix

#### Item 8 - Avoid finalizers and cleaners
- _finalizers_ and _cleaners_ are used to reclaim non-memory resources
  - example: input/output streams, files, etc.
  - they are not analogues of C++ destructors
- finalizers are unpredictable, dangerous and generally unnecessary
- cleaners are less dangerous than finalizers but still slow, unpredicatble and generally unnecessary
- disadvantages:
  - spec makes no guarantee when they'll be executed
    - their execution is a function of GC algorithm, thus JVM implementation
    - never do anything time-critical in a finalizer or cleaner
    - providing a finalizer may arbitrarily delay reclamation of class' instances
    - cleaners are a bit better but still run under the control of GC so still the same applies
    - any existing methods that _claim_ to trigger finalization are decade-old traps
      - examples: ```System.runFinalizerOnExit``` or ```Runtim.runFinalizerOnExit```
  - uncaught exception thrown during finalization is ignored and finalization of that object terminates
    - object is potentially left in corrupted state
    - normally, uncaught exception terminates the executing thread and dumps stacktrace but not if it occurs in finalizer
    - cleaners do not suffer from this problem
  - there is a _severe_ performance GC penalty for using finalizers and cleaners
    - finalizers: ~50x slower reclamation
    - cleaners: ~5x slower reclamation, equal to finalizers if they're used to clean all instance of the class
  - finalizers open up classes to finalizer attacks
    - if an exception is thrown during finalization, the finalizer leaves the class unreclaimed
    - attackers can exploit this and run code that shouldnt've existed
    - to protect non-final classes against it - write a ```final finalize()``` that does nothing
- instead of using finalizers or classes simply use ```AutoCloseable``` 
  - require clients to invoke ```close()``` whenever instance is not required
- legitimate uses:
  - act as a safety net for closeables
    - think long and hard before doing so
  - reclaim native peer objects
- conclusion: don't use cleaners, or in releases prior to Java 9, finalizers - except as a safety net or to terminate non-critical native resources. Even then, beware the indeterminacy and performance consequences

#### Item 9 - Prefer ```try-with-resources``` to ```try-finally```
- Java libraries include many resources that must be closed manually by invoking ```close()```
  - examples: ```InputStream```, ```OutputStream```, ```java.sql.Connection```, etc.
- closing objects is often overlooked by clients
- many use finalizers as safety net, with dire performance consequences (Item 8)
- historically, ```try-finally``` was the best way to guarantee a resource would be closed properly, even when facing exception or return
  - while it doesn't look bad for a single resource, it doesn't scale well with the increase of resources required to be closed
    - nested ```try-finally``` blocks stick out like a sore thumb
  - it's tricky to get it right, even in JDK
  - nested ```finally``` block complicate debugging
- ```try-with-resources``` suffers none of this issues
  - example: 
  ```java
      // try-with-resources on multiple resources - short and sweet
      static void copy(String src, String dst) throws IOException {
      try (InputStream in = new FileInputStream(src);
           OutputStream out = new FileOutputStream(dst)) {
           byte[] buf = new byte[BUFFER_SIZE];
           int n;
           while ((n = in.read(buf)) >= 0)
           out.write(buf, 0, n);
           }
      } catch (IOException e) {
           // do something here
      }
  ```
  - shorter, more readable than ```try-finally```
  - provides far better diagnostics
    - no exceptions are suppressed
- conclusion: always use ```try-with-resources``` when working with resources that must be closed

### Chapter 03 - Methods Common to All Objects

#### Item 10 - Obey the general contract when overriding ```equals()```
- overriding the ```equals()``` seems simple but there are many pitfalls and consequences can be dire
- easiest way to avoid problems is not override the method at all
  - then, we fall back to object identity only - each object is equal only to itself
- do *not* override the method if:
  - each instance of the class is inherently unique
    - example: ```java.util.thread.Thread```
  - there is no need for the class to provide 'logical equality' test
    - example: ```java.util.regex.Pattern```
  - superclass is already overriding ```equals()``` and superclass' behaviour is appropriate for the subclass
    - example: most root classes from ```java.util.collection``` and subclasses of ```java.util.Map``` inherit ```equals()``` behaviour from their abstract implementations
  - the class is (package-)private and we're certain ```equals()``` will never be invoked
    - the risk-averse may throw an exception in the subclass if ```equals()``` is called
  - the class is *instance controlled* (Item 1)
    - example: enum types
- *do* override the method if a class has a notion of **logical equality** that differs from mere object identity
  - this is generally the case for *value classes*
  - examples: ```java.util.Integer``` or ```java.util.String```
- overriding the ```equals()``` implies adhering to its general contract of *equivalence relation*:
  - (*reflexivity*) ```x.equals(x) == true```, for x != null
    - hard to violate unintentionally
  - (*symmetry*) ```x.equals(y) == y.equals(x)```, for x, y != null
    - example: compare ordinary and case insensitive strings
  - (*transitivity*) ```x.equals(y) == y.equals(z) == x.equals(z)```, for x, y, z != null
    - examples: subclass adds a new value component; ```java.util.Date``` and ```java.util.Timestamp```
    - easier to violate when using inheritance
    - once this property is violated, subsequent fixes are likely to violate other properties
    - fundamental problem of equivalence relations in OO languages => there is no way to extend an instantiable class and add a value component whilst preserving the relation!
    - workaround: favour composition over inheritance
  - (*consistency*) ```x.equals(y) == x.equals(y)```, for x, y != null
    - example: 'java.util.URL'
    - do not write ```equals()``` that depend on unreliable resources
  - (*non-nullity*) ```x.equals(null) == false```, for x != null
    - hard to violate unintenionally, unless an exception is thrown instead of returning ```false```
- violation of this contract means it is **uncertain how other objects will behave when confronted with our object**
- conclusions: 
  - rely on IDEs to generate the ```equals()```
  - if manual tuning is really necessary, pay attention to equivalence relation violation (write tests!) and performance

#### Item 11 - Always override ```hashCode()``` when overriding ```equals()```
- violating this rule will violate the general contract for ```hashCode()```
  - it prevents proper functioning of hash-based collections
- general ```hashCode()``` contract:
  - (*consistency*) ```x.hashCode() == x.hashCode()```, for x != null
  - (*equality*) ```x.equals(y) => (x.hashCode() == y.hashCode())```, for x, y != null
- worst possible hash-code implementation is returning the same number
  - degrades performance horrifically
- conclusions:
  - it is mandatory to override ```hashCode()``` each time ```equals()``` is overridden
    - failure to do so precludes correct functioning of the program (fallback on Object)
  - obey the general ```hashCode()``` contract
  - rely on the ```Object.hashCode(...)``` to compute the hash-code unless performance is very important

#### Item 12 - Always override ```toString()```
- makes systems using the class easier to debug
  - disadvantage: once specified, it's for-life
- clearly document intentions
- don't override ```toString()``` on
  - static utility class (Item 4)
  - enum types (Item 34)
- rely on IDE to create a good ```toString()``` implementation

#### Item 13 - Override ```clone()``` judiciously
- conclusions: 
  - do not use ```clone()```
  - use static factory to copy objects whenever possible

#### Item 14 - Consider implementing ```Comparable```
- isn't inherited from ```Object``` type
- located in functional interface ```Comparable```
- similar to ```equals()``` except it permits order comparisons in addition to simple equality comparisons
- implementing ```Comparable```, class indicates its instances follow *natural ordering*
- it also makes it easy to apply various searching, sorting and extreme values computations on such a class
  - example: ```java.util.String```
  - class automatically interoperates with a variety of generic algorithms and collection implementations
  - small effort and code footprint to leverage existing (vast) capabilites
- general contract of ```compareTo()``` is very similar to ```equals```:
  - (*reflexivity*) ```(x.compareTo(y) == 0) => (sgn(x.compareTo(z) == sgn(y.compareTo(z))```, for x, y, z != null
  - (*symmetry*) ```sgn(x.compareTo(y)) == -sgn(y.compareTo(x))```, for x, y != null
    - helps imposing total order
  - (*transitivity*) ```(x.compareTo(y) && y.compareTo(z)) > 0 => (x.compareTo(z)) > 0```, for x, y, z != null
    - helps imposing total order
  - (*optional consistency with equals*) ```(x.compareTo(y) == 0) <=> (x.equals(y))```, for x, y != null
    - if violated, still will yield valid comparisons but will generally not guarantee obeying the general contract for collections and maps
    - examples: ```java.util.BigDecimal``` in ```HashSet``` and ```TreeSet```
- to compare object ref fields, invoke ```compareTo()``` recursively
- do not use relational operators in ```compareTo()```
  - use ```compare()``` on boxed primitive types
  - alternately, use comparator fluent construction methods in ```Comparator``` interface
- do not use hashCode arithmetics-based comparisons
  - though somewhat more performant, they are fraught with danger from integer overflow and FP arithmetic artifacts
  - use the same techniques as in the bullet point above
- generally, same limitations and workarounds as for the ```equals()``` apply here
  - exceptions may be thrown, however, without violating the contract
- conclusions:
  - whenever a value class has sensible ordering, ```Comparable``` should be implemented
    - easy sorting, searching and usage in comparison-based collections
  - do not use relational comparison operators to determine the result of comparing two elements
    - rely on static ```compare()``` in the boxed primitive types
    - alternately, use comparator construction methods in ```Comparator``` interface

### Chapter 04 - Classes and Interfaces

#### Item 15 - Minimise the accesibility of classes and members
- well-designed components:
  - hide their internal data and other implementation details from other components
  - thus, they cleanly separate their API from its implementation
  - components are then oblivios to each other's workings
  - this is *encapsulation* (information hiding) -> a fundamental tenet of software design
- importance of encapsulation:
  - *decouples* the components comprising a system
  - this allows for development/optimisation/usage/reasoning/modification *in isolation*
    - speeds up development (parallelisation)
    - reduces maintenance costs (easier reasoning/debugging/replacement)
    - makes for effective performance tuning (isolation -> does not influence correctness of others)
    - increases reuse (decoupled components usually provide use in contexts beyond original design)
    - de-risks development (individual components may prove successful even if the system does not) 
- Java encapsulation facilities:
  - access control mechanism (```private```, ```protected```, ```public``` keywords + default access)
- encapsulation rule of thumb: *make each class/member as inaccessible as possible*
  - after carefully designing a class' public API, the reflex should be to *make all other members private*
  - change design if you find yourself opening up the API too frequently
    - there may be a better decomposition with higher decoupling
  - some fields may leak into the exported API if the class implements ```Serializable``` interface
- huge increase in visibility when going from default to ```protected```
  - becomes a part of exported API (has to be supported forever)
  - should be relatively rare
- overridden methods cannot have restrictive access level in the subclass than it is in the superclass
  - fields a compile-time error
  - a consequence of *Liskov substition principle*
  - special rule -> if a class implements an interface, all implemented methods *must* be public
- opening up a class to facilitate testing is acceptable (to a degree)
  - the less restrictive access must not be any higher than *default* access
  - it is fine to make the code "worse" in order to cover it with tests (M. Feathers, "Working Effectively With Legacy Code")
- instance fields of public classes should rarely be public
  - gives up :
    - the ability to enforce invariants on the field
    - the flexibility to change data structure
  - not thread-safe
  - same applies for ```static``` fields, except if they are also ```final```
    - exception: it's always wrong to expose a mutable data structure as ```public static final``` (e.g. Arrays, Lists...)
    - for such instances make an accessor which defencively copies the data structure and make the field ```private```
- Java 9 modules:
  - module is a *group of packages*, much like a pakcage is group of classes
  - it may explicitly *export* some of its packages (via export declarations in its module declaration ```module-info.java```)
  - public/protected members of unexported packages in a module are inaccessible outside the module
    - within, they work as before
  - using the module system allows us to share classes among packages within a module without making them visible to the entire world
  - the need for this kind of sharing is relatively rar and can often be eliminated by rearranging classes within a package
  - it is unclear if it will achieve widespread use outside of the JDK itself (where it's strictly enforced)
- summary:
  - reduce accebility of program elements as much as possible
  - do not expose mutable types as ```public static final``` fields
  
#### Item 16 - Favour accessors over public fields in public classes

- occasionaly, degenerate classes are rolled out such as this one:
```java
class Point {
    public double x;
    public double y
}
```
- if confined to package-private or private access modifier, they can be very useful in reducing visual clutter and overhead
  - making them more accessible would be dangerous and is not advised
    - there are examples in JDK that violate this rule (```java.awt.Point```, ```java.awt.Dimension```)

#### Item 17 - Minimise mutability

- immutable classes are classes whose instances cannot be modified
  - all of the data in the object is fixed for the lifetime of the object
  - e.g. ```java.lang.String```, the boxed primitive classes, ```BigInteger``` and ```BigDecimal```
- many reasons to use immutable classes -> easier to design, implement and use than mutable classes
- to make a class mutable, follow these 5 rules:
  - don't provide mutators
  - ensure that the class cannot be extended
  - make all fields final
  - make all fields private
  - ensure exclusive access to any mutable components
- immutable classes are easier to realise using *functional*, rather than *imperative* approach
- immutable objects are *simple*
  - has always exactly *one* state - the state in which it was created
  - they are easier to use reliably
- immutable object are inherently *thread-safe* (they require no synchronisation)
  - thus can be shared freely, promoting reuse
  - no defencive copies necessary
  - their internals can be shared freely
- immutable objects make great building blocks for other objects
  - they also make great map keys or set elements
- immutable object provide atomicity for free
- the one disadvantage -> they require a separate object for each distinct value
  - the problem is exacerbated if the object is a part of multistep transformation
  - one way to solve this issue is by providing mutable 'companion classes'
- no method may produce an *externally visible* change in the object's state
  - consider using *lazy initialisation* technique
- summary:
  - classes should be immutable unless there is a very good reason to make them mutable
  - if a class cannot be made immutable, limit its mutability as much as possible
    - declare every field ```private final``` unless there is a good reason to do so otherwise
- constructors should create fully initialised objects with all their invariants established
  - e.g. ```CountDownLatch```

#### Item 18 Favour composition over inheritance

- this chapter applies to *implementation* inheritance, not *interface* inheritance
- inheritance is a powerful way to achieve code reuse 
- can lead to fragile software if employed inappropriately
- safe to use:
  - within a package (where sub- and super-class are under control of the same programmers)
  - if a class specifically is designed and documented for inheritance
- inheritance violates encapsulation
  - subclass depends on implementation details of the superclass
  - both must evolve in tandem
  - e.g. ```HashSet``` extension
    - depending on superclass' method to implement your own can lead to fragility - as the superclass evolves, your code may break without any changes
  - related cause of fragility is that their superclass can acquire new methods in subsequent releases
    - e.g. ```Hashtable``` and ```Vector``` classes had to had their security holes fixed before being retrofitted to participate in Collections framework
- solution to these issues is the application of *composition*
  - existing class becomes a component of the new one
  - new class is free to *forward* calls to the old class where appropriate (*forwarding methods*)
  - resulting classes are then rock solid:
    - they don't depend on the implementation details of the existing class
    
