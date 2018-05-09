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
