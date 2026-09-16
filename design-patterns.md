# Patterns

## System

### Load Balancer Pattern
- The load balancer pattern distributes incoming requests across multiple servers. This ensures no single server becomes overwhelmed, enhancing system availability and reliability. By balancing the load, this pattern improves user experience and system performance

### Sharding Pattern
- The sharding pattern divides a database into smaller, more manageable pieces called shards. Each shard handles a portion of the data, reducing the load on any single database instance. This approach enhances performance and storage capacity, making the system more scalable

### Circuit Breaker Pattern
- The circuit breaker pattern prevents a system from repeatedly attempting a failing operation. It helps maintain stability by isolating faults and preventing cascading failures. This pattern improves system resilience and reduces downtime

### Retry Pattern
- The retry pattern automatically retries a failed operation after a specified interval. This approach helps recover from transient faults without manual intervention. It ensures higher reliability and better user experience by minimizing disruptions

### Caching Pattern
- The caching pattern stores frequently accessed data in a cache to reduce database load. This results in faster response times and improved overall performance. Caching is crucial for systems with heavy read operations, enhancing user satisfaction

### Bulkhead Pattern
- The bulkhead pattern isolates different parts of a system to prevent failure in one area from affecting others. This pattern enhances system robustness by containing faults and limiting their impact. It ensures that a failure in one component does not bring down the entire system

### Rate Limiting Pattern
- The rate limiting pattern controls the number of requests a system can handle within a given time frame. This prevents system overload and ensures consistent performance. Rate limiting is essential for managing traffic spikes and protecting the system from abuse

### Event Sourcing Pattern
- The event sourcing pattern stores the state of a system as a sequence of events. This approach helps in reconstructing the state and improving fault tolerance. Event sourcing is useful for systems requiring audit trails and historical data analysis, ensuring data integrity and reliability

## Programming

### Singleton 
- Restricts the instantiation of a Class and ensures that only one instance of the class exists

### Factory 
- Pattern is used when we have a superclass with multiple subclasses and based on input, we need to return one of the subclasses

### Abstract Factory
- Has a factory class for each subclass and then an abstract factory class that will return the subclass based on the input factory class

### Builder
- Solves the issue with a large number of optional parameters and inconsistent state by providing a way to build the object step-by-step and provide a method that will actually return the final Object

### Prototype
- Used when the Object creation is costly and requires a lot of time and resources, and you have a similar Object already existing. So this pattern provides a mechanism to copy the original Object to a new Object and then modify it according to our needs

### Adapter
- Used so that two unrelated interfaces can work together. The object that joins these unrelated interfaces is called an adapter

### Composite
- Used when we have to represent a part-whole hierarchy. When we need to create a structure in a way that the objects in the structure have to be treated the same way, we can apply the composite design pattern

### Proxy
- Provides a placeholder for another Object to control access to it

### Flyweight
- Used when we need to create a lot of Objects of a Class, can be applied to reduce the load on memory by sharing Objects
