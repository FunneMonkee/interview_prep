# Java Questions

## Memory

### Heap/Stack allocation
- All Java objects are dynamically stored in the heap memory
- References to these objects are stored in the stack memory
- Declaring a variable with no "new" keyword only creates the reference
- Heap stores actual objects
- Stack stores method call data, local variables and references

### Garbage collection
- The garbage collector is responsible for reclaiming memory by destroying objects that are no longer in use
- When an object is unreachable it becomes eligible for garbage collection
- Serial Garbage Collector: the simplest type of garbage collector. It uses a single thread to handle all garbage collection tasks
- Parallel Garbage Collector: the default garbage collector in Java 8. It uses multiple threads to perform garbage collection tasks
- Concurrent Mark-Sweep Collector: designed to minimize pause times by doing most of its work while the application is still running. It uses multiple threads to mark and clean up unreachable objects
- G1 Garbage Collector: introduced in JDK 7 and became the default garbage collector in Java 9. Designed for applications with large heap sizes (greater than 4GB)
- Z Garbage Collector: designed to handle large heaps with minimal pause times, typically in the range of milliseconds
- Shenandoah Garbage Collector: another low-pause-time garbage collector introduced in Java 12

### Young generation 
- Young generation is for small, short-lived objects. Optimized for fast garbage collection, 1/3 of the total heap by default
- Eden is where new objects are initially allocated 
- Survivor spaces S0 and S1 temporarily hold objects that survive garbage collection in Eden, default ratio to Eden of 8:1:1
- Minor Garbage Collection marks objects in Eden and the current Survivor Space, cleans up, copies survivors to the inactive survivor space, swaps inactive for active (Surivor Spaces)

### Old generation
- Usually 2/3 of the heap
- When an object in Young Generation survives with a certain age (default 15), its promoted to Old Generation
- Big objects or Survivor Space overflow may mean a new object is instantly allocated to Old Generation
- Major Garbage Collection, typically a "Full Garbage Collection" meaning both Major and Minor
- Major Garbage Collection algorithms trigger stop-the-world (STW) pauses, pausing application threads

### Permanent Generation
- Pre java 8
- Fixed heap size
- Stores metadata

### Metaspace 
- After Java 8
- Dynamic sizing
- Stores the sames things as Permanent Generation

### Object Memory Layout
- Object header: contains metadata like the object's class and lock information for synchronization
- Instance data: refers to the actual fields or variables that belongs to an object
- Padding: refers to the extra memory to align the object size to the word size for better performance

The object header includes:

- Class Pointer: a reference to the object's class
- Lock Information: used for synchronization in multithreaded programs
- GC-related Information: used by the garbage collector for managing the object lifecycle

### Memory leaks 
- Objects unintentionally retained
- Static collections growing forever
- Listeners not being removed
- Caches without eviction

### How would you investigate a Java application consuming too much memory?
- Check JVM metrics
- Monitor heap usage
- Take a heap dump
- Analyze retained objects
- Identify unexpected references
- Check caches and collections
- Check GC behavior

## Concurrency and Multithreading

### Process vs thread
- Process has its own memory space.
- Threads share process memory.
- Threads are lighter than processes.
- Synchronization is necessary when sharing data.

### Thread safety
- When multiple threads access the same object or piece of code at the same time, the program still behaves correctly, without data corruption or unexpected results

### Synchronization
- Used to control the execution of multiple processes or threads so that shared resources are accessed in a proper and orderly manner
- Synchronized Methods, used to lock an entire method so that only one thread can execute it at a time for a particular object
- Synchronized Blocks, allow locking only a specific section of code instead of the entire method
- Static Synchronization, used when static data or methods need to be protected in a multithreaded environment
- Process Synchronization ensures multiple processes or threads can execute safely while sharing common resources
- Thread Synchronization is used to coordinate and ordering of the execution of the threads in a multi-threaded program

### volatile
- Ensures that all threads have a consistent view of a variable's value
- Prevents caching of the variable's value by threads, ensuring that updates to the variable are immediately visible to other threads
- Does not guarantee atomicity
- Guarantees visibility

### volatile vs synchronized
- Volatile ensures visibility, Synchronized too with mutual exclusion
- Volatile does not use locking, Synchronized does 
- Volatile is not thread-safe, Synchronized is 
- Volatile no atomicity guarantee, Synchronized guarantees atomicity 
- Volatile lighter and faster, Synchronized heavy and slow (due to locking) 
- Volatile used simple variables and flags, Synchronized used for critical sections 

### Performance considerations
- Multithreading enhances performance with parallel execution but too many threads can overload system resource and become to complex to maintain for certain projects (too much locking, shared resources, etc)

### Locking
- ReentrantLock allows a thread to reacquire the lock it already holds
- ReadWriteLock allows multiple threads to read a resource concurrently, but only one thread to write, ensuring no reads happen during writing

### AtomicInteger, AtomicLong, AtomicBoolean, and AtomicReference
- These classes represent an int, long, boolean, and object reference respectively which can be atomically updated

### Concurrent Collections
- Thread Safety: Automatically handles synchronization, so you don’t need to worry about manual locking
- Performance: Uses segment-level locking, allowing multiple threads to read and write simultaneously
- Error Prevention: Eliminates common concurrency issues like inconsistent data or runtime exceptions
- Scalability: Optimized for multi-core systems to maintain high performance even under heavy concurrency
- ConcurrentHashMap is a thread-safe version of HashMap that allows concurrent read and write operations. Instead of locking the entire map, it locks only specific portions (buckets) for better performance
- CopyOnWriteArrayList is a thread-safe version of ArrayList where a new copy of the list is created whenever it is modified. Best suited for scenarios with frequent reads and few writes
- BlockingQueue is useful in scenarios where one thread produces the data and other thread consume it. It blocks the producer when the queue is full and blocks the consumer when it is empty (ReadWriteLock)
- ConcurrentSkipListMap is a thread-safe alternative to TreeMap that maintains elements in sorted order. Internally, it uses a Skip List for efficient concurrent access and navigation
- ConcurrentLinkedQueue is a thread-safe, non-blocking queue that uses lock-free algorithms. Ideal for high-performance systems where many threads access the queue simultaneously

### Race conditions
- A race condition occurs when multiple threads read and write the same variable, i.e., they have access to shared data and try to change it at the same time
- Use volatiles, synchronization, atomic and locks

### Wait vs Sleep
- wait releases the monitor
- sleep does not release locks
- wait is used for thread coordination

### What is a thread pool?
- A Thread Pool is a collection of reusable worker threads managed by an Executor Service. Depending on the implementation, a thread pool may maintain a fixed number of threads, dynamically create threads as needed, use a single worker thread, or schedule tasks for future execution.
- Fixed Thread Pool uses a fixed number of worker threads
- Cached Thread Pool creates threads as needed and reuses idle threads
- Single Thread Executor uses only one worker thread
- Scheduled Thread Pool executes tasks after a delay or periodically
- Work-Stealing Pool - uses multiple queues to improve parallel task execution

### Why shouldn't you create a new thread for every request?
- Resource consumption
- Context switching
- Scalability
- Controlled concurrency

### ExecutorService
- Simplifies running tasks in asynchronous mode. Generally speaking, ExecutorService automatically provides a pool of threads and an API for assigning tasks to it
- Can execute Runnable and Callable tasks
- ScheduledExecutorService exists

### ThreadPoolExecutor
- The ThreadPoolExecutor is an extensible thread pool implementation with lots of parameters and hooks for fine-tuning
- The pool consists of a fixed number of core threads that are kept inside all the time. It also consists of some excessive threads that may be spawned and then terminated when they are no longer needed

## ExecutorService vs ThreadPoolExecutor
- ExecutorService is a higher-level interface that simplifies thread management, while ThreadPoolExecutor provides more control over thread behavior and execution policies
- ThreadPoolExecutor allows you to customize parameters like core pool size, maximum pool size, keep-alive time, and work queue type, which can significantly affect performance

### How would you design a high-performance multithreaded system?
- Thread pools
- Avoiding unnecessary locking
- Reducing contention
- Concurrent collections
- Backpressure
- Bounded queues
- Monitoring latency and throughput

## Deadlocks
- Deadlock is a situation in multithreading where two or more threads are permanently blocked because each one is waiting for the other to release a required lock
- jps -l shows if a deadlock is happening
- Avoid Nested Locks
- Avoid Unnecessary Locks
- Use synchronization methods

### Mutual exclusion
- Previously mentioned locking mechanisms act as mutexes

### How would you debug a deadlock in a production Java application?
- Obtain thread dumps
- Analyze blocked threads
- Look for circular dependencies
- jps -l shows if a deadlock is happening

## Classes

### Interface vs Abstract
- An abstract class is a class that cannot be instantiated and is used as a base for other classes. It can contain both abstract (unimplemented) and concrete (implemented) methods
- An interface is a blueprint that defines a set of methods a class must implement. It focuses on behavior rather than implementation.
- Both can be thought of as contracts, with an interface being more restrictive

### Default methods
- Basically method that are default for an interface implemtation, with not need for an actual implementation of that method

### Checked vs Unchecked exceptions
- Checked exceptions are exceptions that are checked by the compiler at compile time. If a method can throw a checked exception, it must be either handled using a try-catch block or declared using the throws keyword
- Unchecked exceptions are exceptions that are not checked by the compiler at compile time. They occur during program execution and usually result from programming errors or incorrect logic.

### How do you design exception handling in a large production application?
- Errors != Exceptions
- Usually there is no way of recovering from an error without "manual" action, these can be turned into exceptions (known exceptions specific for each project)
- Exceptions represent something exceptional has happened and can be caught and recovered from without crashing the current flow or entire application.
- Ideally catching should be specific for 1..N exceptions, not generic, so recovering and giving feedback is simpler
- Error, Warning and Info logging (debug too) can and will be helpful for situations where the system cannot handle the exception (error)

## Depdency Injection in Spring
- Spring Boot simplifies RESTful API and microservice development by minimizing configuration. Its core concepts are Dependency Injection (DI) and Spring Beans
- Constructor Injection: Dependencies are injected through the class constructor. It ensures that all required dependencies are provided at the time of object creation, which promotes immutability
- Setter Injection: Dependencies are injected via setter methods. This approach allows changing dependencies even after the object is created and supports optional dependencies

### Spring Beans
- @Component: A general-purpose annotation to mark a class as a Spring Bean
- @Service: Specialized for service-layer classes
- @Repository: Specialized for persistence-layer classes
- @Controller: Specialized for controller classes in Spring MVC

## JPA and Hibernate
- Java provides JPA (Java Persistence API) as a specification for managing relational data, while Hibernate is a popular implementation of that specification. In simple terms, JPA defines the rules, and Hibernate provides the actual functionality.

### JPA
- JPA (Java Persistence API) is a Java specification that defines how ORM frameworks should work. It provides a standard set of interfaces and annotations to manage relational data but requires an implementation like Hibernate to perform actual operations
- @Entity, @Id, and @Table for mapping
- Allows switching between different ORM tools
- EntityManager for managing persistence operations

### Hibernate
- Hibernate is an open-source ORM framework that implements JPA and provides advanced features to simplify database operations. It maps Java objects to database tables and reduces the need for writing SQL queries manually
- Built-in caching
- Multiple databases with minimal configuration changes
