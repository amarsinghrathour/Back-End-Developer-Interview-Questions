### [[↑]](../README.md#toc) <a name='architecture'>Questions about Software Architecture:</a>

#### No Cache
> When is a cache not useful or even dangerous?

**Expert Answer:**
A cache is dangerous when data is highly transactional and consistency is strictly required (e.g., banking balances). If the cache falls out of sync with the database, you might allow a user to spend money they don't have. It's not useful when the data has extremely high cardinality and zero repeat access (e.g., logging every unique API request), because the cache hit rate will be 0%, and you are just wasting memory and adding latency to manage the cache eviction.

#### Event-Driven Architecture
> Why does Event-Driven Architecture improve scalability?

**Expert Answer:**
EDA decouples the producer of an event from the consumer. When a user uploads a video, the HTTP handler just drops an `VideoUploaded` event onto a queue and returns a `202 Accepted` immediately. This scales because the web servers don't block waiting for video processing. Furthermore, you can dynamically scale up the number of worker nodes consuming from the queue depending on the backlog depth, without affecting the web tier.

#### Readability
> What makes code readable?

**Expert Answer:**
Readable code optimizes for the person reading it 6 months later, not the person writing it today. 
In Go, readability means:
1. Short, expressive variable names for local scope, descriptive names for global scope.
2. Avoiding deeply nested `if` statements (using early returns/guard clauses).
3. "Clear is better than clever." Avoid hyper-optimized, unreadable bitwise hacks unless absolutely necessary and well-commented.
4. Consistent formatting (automatically enforced by `gofmt`).

#### Emergent and Evolutionary
> What is the difference between emergent design and evolutionary architecture?

**Expert Answer:**
*   **Emergent Design:** Code-level. You don't try to architect every class upfront. You write simple code, and as patterns emerge, you refactor them into reusable components. TDD drives emergent design.
*   **Evolutionary Architecture:** System-level. Designing the architecture (servers, databases, deployment) so that it can easily change over time without massive rewrites. Utilizing microservices, fitness functions, and decoupled events ensures the architecture can evolve.

#### Scale-Out, Scale-Up
> Scale out vs scale up: how are they different? When to apply one, when the other?

**Expert Answer:**
*   **Scale Up (Vertical):** Buying a bigger server with more RAM and CPU. Use it for monolithic databases (PostgreSQL) where scaling out is architecturally difficult. It is easy but hits a hardware limit quickly.
*   **Scale Out (Horizontal):** Buying 100 cheap servers. Use it for stateless web applications (Go backends). It is infinite, but requires the application to be stateless and load-balanced.

#### Failures User Sessions
> How to deal with failover and user sessions?

**Expert Answer:**
Never store user sessions in the memory of the web server (stateful). If that server dies, the session is lost.
Instead, store the session ID in a signed cookie on the client, and the actual session data in a highly-available, fast, external data store like a Redis cluster. Any web server can query Redis. If a web server fails, the load balancer routes the user to another server, which fetches the session from Redis seamlessly.

#### CQRS
> What is CQRS (Command Query Responsibility Segregation)? How is it different from the oldest Command-Query Separation Principle?

**Expert Answer:**
*   **CQS (Code level):** A function should either change state (Command) or return state (Query), but never both.
*   **CQRS (Architecture level):** Separating the read models and write models into completely different databases or services. A Go service might write commands to an event store (Kafka), which asynchronously projects the data into an Elasticsearch cluster optimized purely for blazing-fast read queries.

#### n-tier
> Would you discuss the pros and cons of such an approach?

**Expert Answer:**
*   **Pros:** Enforces separation of concerns. The database can be swapped without touching the UI. Developers can specialize (Frontend vs DBA).
*   **Cons:** Over-engineering for simple apps. It often leads to "anemic domain models" where the application tier is just a dumb pipe passing data from the DB to the UI. It increases latency and deployment complexity.

#### Scalability
> How would you design a software system for scalability?

**Expert Answer:**
1.  **Stateless Compute:** All Go web servers must be stateless and horizontally scalable behind a load balancer.
2.  **Asynchronous I/O:** Move heavy processing to background workers via message queues (RabbitMQ/Kafka).
3.  **Caching:** Cache heavily at every layer (CDN for static assets, Redis for DB queries).
4.  **Database:** Use read-replicas to scale reads. Shard the database for write scaling if necessary.

#### C10K
> It may be interesting to discuss the strategies you know to deal with that problem. Would you like to try?

**Expert Answer:**
In the 90s, handling 10,000 concurrent sockets was impossible because OSs allocated a heavy OS-thread (1MB+ memory) per connection.
**The solution:** Asynchronous I/O (epoll on Linux, kqueue on BSD). 
In Go, this is magically handled for you. You spawn a lightweight `goroutine` for every connection (taking only 2KB of memory). The Go runtime scheduler multiplexes 10,000+ goroutines onto a small number of OS threads using `epoll` under the hood. Go solved the C10K (and C1M) problem natively.

#### CGI
> Can you discuss why CGI became obsolete, and was instead replaced with other architectural approaches?

**Expert Answer:**
CGI spawned a brand new OS Process for every single HTTP request. Spawning a process (forking) is extremely slow and memory-intensive. Under heavy load, the server would exhaust memory and crash. FastCGI improved this by keeping a pool of processes alive. Today, languages like Go embed a hyper-efficient HTTP server directly in the compiled binary, utilizing multiplexed threads instead of heavy processes, offering orders of magnitude better performance.

#### Vendor Lock-in
> How would you defend the design of your systems against vendor Lock-in?

**Expert Answer:**
Use the **Hexagonal Architecture (Ports and Adapters)**. 
Write your core business logic in pure Go with no dependencies. When you need to save data, define a `Repository` interface. Write an adapter that implements that interface using AWS DynamoDB. If AWS becomes too expensive, you write a new adapter for Google Cloud Spanner. The core business logic never changes, because it is entirely decoupled from the vendor SDK. 
*Caveat:* Abstracting too heavily prevents you from using the unique, powerful features of a specific cloud provider.

#### CPUs
> What's new in CPUs since the 80s, and how does it affect programming?

**Expert Answer:**
1.  **Multi-core:** CPUs stopped getting faster and started multiplying. This forced programmers to learn concurrent programming to utilize the hardware.
2.  **Cache Lines (L1/L2/L3):** Memory access is relatively slow compared to CPU speed. CPUs now pre-fetch memory into blazing-fast local caches. 
*Impact:* Data-Oriented Design. An array of structs (contiguous memory) is 100x faster to iterate over than an array of pointers to structs, because the pointers cause "cache misses" (fetching random memory addresses). Go's pass-by-value structs leverage CPU caches beautifully.

#### DDOS
> How could a denial of service arise not maliciously but due to a design or architectural problem?

**Expert Answer:**
The **Thundering Herd** problem. 
If an expensive database query's cache expires, and 1,000 concurrent requests hit the server, all 1,000 requests miss the cache simultaneously and query the database. The database CPU spikes to 100% and it crashes. 
*Fix:* Use cache-stampede protection (like Go's `golang.org/x/sync/singleflight`), ensuring only *one* request queries the database while the other 999 block and wait for the result.
