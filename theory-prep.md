# Main Interview Framework Web Backend Focused
- Requirements (Functional/Non-Functional)
- Core Entities
- API (REST, GraphQL, etc)
- Data Flow (User Request/API call -> DTO parsing -> service/domain logic -> data retrieval request -> more logic -> convert data to dto for client) 
- High-level Design (Client, Gateway, CRUD Service, DB)
- Deep Dive (addressing all requirements, mainly functional; edge cases; improving design based on interviewer questions)

# Systems

## CAP Theorem
- You can only have two of three properties at once. Consistency (all nodes see the same data), Availability (every request gets a response), and Partition tolerance (system works even when network connections fail between nodes)
- Availability, if most clients can handle stale data but not a server being down. (Eventual consistency)
- Consistency matters for systems where stale data causes business problems.

## Networking
- Most systems will just be HTTP over TCP
- WebSockets and Server-Sent Events for real-time updates
- SSE is unidirectional, client opens connection, server pushes data
- WebSockets are bidirectional
- Both are stateful so a standard load balancer doesn't cut it
- gRPC for internal service-to-service communication when performance is critical
- if the system needs low latency globally, use regional deployments with data replicated or partitioned by geography. Use CDN (Content Delivery Network) to serve data in server close to the client.

### CDN
- Works by caching content on servers that are close to users. When a user requests content, the CDN routes the request to the closest server. If the content is cached on that server, the CDN will return the cached content. If the content is not cached on that server, the CDN will fetch the content from the origin server, cache it on the server, and then return the content to the user.
- Mostly used for static assets but can also be used for dynamic content like an API response

### Load Balancing
- a load balancer is needed wherever there are multiple machines capable of handling the same request
- Layer 7 (OSI) load balancers operate at the application level and can route based on the actual HTTP request content
- Layer 4 (OSI) load balancers work at the TCP level and are faster but dumber
- Layer 4 is typically used for WebSockets

### API Gateway
- In a microservice architecture, an API gateway sits in front of the system and is responsible for routing incoming requests to the appropriate backend service
- authentication, rate limiting, and logging

## Databases

### Blob
- Stores large amounts of data
- Slow, can be used in conjuction with another database type to store pointers/keys and other data for faster fetching from the blob storage
- Lower cost

### Relational
- Fast querying
- Joining
- Relations
- Migrations

### Non-Relational

#### Document
- Easy to update the schema (no migrations)
- Faster storage
- Different "tables" can be stored inside a document. For example, an `User` owning a collection of `Addresses`

#### Key-Value Store
- Basically a giant hashmap
- Good for querying per key and getting a bunch of information at once

#### Wide-Column 
- For huge amounts of data
- Names and format of the columns can vary across rows
- Column that exists in one row but not in another doesn't slow down querying (with nulls or defaults)

## Indexes

### Clustered Indexes
- Defines a physical order of rows
- Data can be sequential/sorted
- Column for key must be unique (PK are usually clustured)
- Faster for range-based queries and sorting

### Clustered Indexes
- Data is stored separatly from the table
- Multiple can exist on a table
- Stores a copy of the indexed columns with a pointer for the actual data
- Better for specific lookups on non-key columns

### Inverted Indexes
- Mostly used for full text search, using tokenization, stemming and fuzzy search
- Maps words to documents for faster querying

## Sharding 
- For fast queries keep data in the same shard, for example, for an user, addresses should live in the same shard
- Can be done with hashing, picking shards using modulo
- Range based sharding works well for something like multi tenant data (each client accesses only its own data)
- Trade off is queries for "global" data become more expensive
- Another trade off, cross-shard transactions become nearly impossible
- Resharding is not easy and requires moving a lot of data
- Usually not neeed until the number of writes and data gets too big

### Consistent Hashing
- Is used for caching and sharding so the hash used to get data doesn't change (with the number of shards for example)
- a ring works well for this, when adding a new server, only the keys between that new server and the previous server need to move. When removing a server, only its keys relocate to the next server on the ring. Everything else stays put

### ACID
- Atomicity (commit, abort, rollback)
- Consistency (data must remain valid before and after a transaction)
- Isolation (transcations run independently without affecting each other)
- Durability (after a transaction is commited, the changes are permanent)

## Message Queues
Just a simple queue for producing and consuming messages, with a broker in the middle

### Kafka 
- Smart consumer, simple broker 
- Basically append only log
- Retention in the append log
- Consumers track own offset for append log
- Can rewind offset and get older messages
- No Dead Letter Queue out of the box
- New Services can look at the entire (still persisted) history
- Ordered by partition (via a partition key)
- Faster broker, more latency due to batched consuming
- At Least Once delivery
- Exactly Once delivery (very limited)
- ZooKeeper/KRaft, partition management

### RabittMQ 
- Smart broker, simple consumer
- Producer -> Broker -> Consumer
- Broker routes, tracks and retries messages
- Dead Letter Queue
- Global order in a single consumer environment
- Ordering can depend on multiple consumers
- Slow broker
- At Least Once delivery
- Simple deployment for a small number of queues

## Caching

### Cache-Aside
- Only cache needed data (previously requested)
- Data retrieval is done by the application and not the cache
- Cache miss takes longer due to the actual caching done after retrieving data.

### Write-Through
- Writes to cache and then to Database
- Slower writes (when caching)
- Too much caching might happen
- One write can fail, causing inconsistencies
- Used for reads that must always return fresh data

### Write-Back
- Async write to Database after writing to cache
- Cache failing may result in loss of data
- Used when some kind of data loss is ok, high write througtput

### Read-Through
- On cache miss, the cache itself requests the data from the database 

## Common Patterns

### Long-Running Tasks
- Message queues for job coordination and worker pools for processing
- When users submit heavy tasks, the web server instantly validates the request, pushes a job to a queue (like Redis or Kafka), and returns a job ID within milliseconds. Separate worker processes continuously pull jobs from the queue and execute the actual work. This provides fast user response times, independent scaling of web servers and workers, and fault isolation

### Contention
- Database-level pessimistic locking and optimistic concurrency control
- Distributed coordination mechanisms
- Understanding when to use atomicity and transactions versus explicit locking strategies. For distributed systems, distributed locks, two-phase commit protocols, or queue-based serialization

### Scaling Reads
-  Optimize read performance in the database through indexing and denormalization, scale horizontally with read replicas, then add external caching layers like Redis and CDNs
- Managing cache invalidation, handling replication lag in read replicas, and dealing with hot keys where millions of users request the same content

### Scaling Writes
- Horizontal sharding (distributing data across multiple servers), vertical partitioning (separating different types of data), and handling write bursts through queues and load shedding
- Good partition keys that distribute load evenly while keeping related data together
- For burst handling, write queues to buffer temporary spikes or implement load shedding to prioritize important writes during overload. Batching techniques help reduce per-operation overhead by grouping multiple writes together

### Handling Large Blobs
- Generating temporary scoped credentials that allow the client to direclty download large amounts of data
- CDNs also help

### Multi-Step Processes
- Single-server orchestration
- Sophisticated workflow engines and durable execution systems
- Modern workflow systems like Temporal or AWS Step Functions handle state management, failure recovery, and retry logic automatically

## Docker and Deployment

### Containers
- Lightweight, portable units that package an application and its dependencies together, allowing it to run consistently across different computing environments

### Images
- Is a standardized package (a read-only template) that includes everything needed to run a container, like files, binaries, libraries, and configuration. 

### Dockerfile
- Contains a list of instructions for building a Docker image. It automates the process of creating a container by specifying commands like which base image to use, what files to copy, and what commands to run

### Container vs VM
- Containers are lightweight and share the host operating system's kernel, while virtual machines (VMs) are more resource-intensive and run a complete operating system with their own kernel

### What is the difference between IaaS, PaaS and SaaS?
- PaaS (Platform as a Service) provides a managed environment for developing applications
- IaaS (Infrastructure as a Service) offers virtualized computing resources.
- SaaS (Software as a Service) delivers fully developed applications that users can access directly without managing the underlying infrastructure

### How would you design a highly available application in the cloud?
- Multiple instances
- Load balancing
- Database replication
- Monitoring
- Automatic recovery

### Horizontal scaling vs Vertical scaling
- Horizontal means more machines/nodes
- Vertical means more cpu, memory, basically a more powerful machine/node

# Production problems decision making

## A client's system suddenly becomes slow. What do you do?
- Finding the root cause by

### Understand the impact
- Is everyone affected?
- When did it start?
- What changed?

### Collect metrics
- CPU
- Memory
- Disk
- Network
- Database latency
- JVM metric

### Check logs
- Exceptions
- Errors
- Timeouts

### Identify bottlenecks
- Database
- Application
- Messaging
- Network
- External services

### Mitigate
- Roll back
- Scale
- Restart unhealthy services if appropriate
- Reduce load

## An application suddenly has 100% CPU usage. How do you debug it?
- Check OS-level processes
- Identify busy threads
- Take thread dumps
- Analyze stack traces
- Look for infinite loops or excessive computation
- Compare with application metrics

## The application works in development but not in the client's environment. What do you do?
- Configuration
- Environment variables
- Database versions
- Network
- Permissions
- Dependencies
- Infrastructure
- Resource limits

## How do you handle a client who reports a problem but cannot explain it clearly?
- Ask structured questions
- Understand business impact
- Collect logs and examples
- Reproduce the problem
- Avoid making assumptions
- Keep the client informed

## How do you explain a complex technical problem to a non-technical client?
- Try to understand the audience and avoid unnecessary technical terminology
- Explain the impact first, then the cause at an appropriate level of detail
- Explain what we are doing to solve it

## A client requests a customization that conflicts with the existing product architecture. What would you do?
- Understand the requirement
- Understand the business value
- Analyze technical impact
- Look for alternatives
- Avoid quick solutions that create technical debt
- Discuss trade-offs with stakeholder
