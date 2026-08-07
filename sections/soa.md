### [[↑]](../README.md#toc) <a name='soa'>Questions about Service Oriented Architecture and Microservices:</a>

#### Long-lived Transactions
> Why, in a SOA, long-lived transactions are discouraged and sagas are suggested instead?

**Expert Answer:**
A traditional ACID transaction holds locks on database rows until it completes. In a distributed SOA, a "long-lived" transaction might need to wait for a 3rd party API, a manual approval, or a slow network call. Holding a global database lock across multiple microservices for seconds (or minutes) will instantly crush the system's concurrency and throughput.
The Saga pattern solves this by breaking the transaction into a sequence of small, local, immediately-committed transactions. If a later step fails, it runs *compensating transactions* to undo the previous steps, maintaining eventual consistency without holding global locks.

#### SOA and Micro Services
> What are the differences between SOA and microservice?

**Expert Answer:**
Microservices are an evolution (or a specific, heavily-constrained subset) of SOA.
*   **SOA (Service Oriented Architecture):** Typically enterprise-scale. Relies heavily on a massive central Enterprise Service Bus (ESB) for routing and transformation. Services often share a massive monolithic database. Communication is often heavy (SOAP/XML).
*   **Microservices:** "Smart endpoints and dumb pipes." The services are completely independent, communicate via lightweight protocols (REST/gRPC/Kafka), and crucially, *each microservice completely owns its own database*. They are much smaller in scope and built around business capabilities rather than technical layers.

#### Versioning and Breaking Changes
> Let's talk about web services versioning, version compatibility and breaking changes.

**Expert Answer:**
In microservices, you cannot force all consumers to update simultaneously. 
*   **Non-Breaking Changes:** Adding a new field to a JSON response, adding a new endpoint. Consumers should be built using Postel's Law (ignore unknown fields) so these don't break them.
*   **Breaking Changes:** Renaming a field, changing a type (int to string), or removing an endpoint.
To handle breaking changes, you must version the API (e.g., `/v1/` to `/v2/`). You run both versions simultaneously. The `/v1/` endpoint becomes a translation layer that maps the old request format to the new internal `/v2/` logic, until all clients have migrated.

#### Sagas and compensations
> What's the difference between a transaction and a compensation operation in a saga, in SOA?

**Expert Answer:**
*   A **transaction** (in the context of a Saga step) is a forward-moving action that commits a change to a local database (e.g., `UpdateOrderStatus(PENDING)`). It cannot be "rolled back" via SQL `ROLLBACK` because it has already been committed to make the locks available to other processes.
*   A **compensation** is a *new*, completely separate business operation that semantically undoes the transaction (e.g., `UpdateOrderStatus(CANCELLED_DUE_TO_PAYMENT_FAILURE)`). It does not erase history; it appends a new state that negates the previous one.

#### Too Micro
> When is a microservice too micro?

**Expert Answer:**
A microservice is "too micro" when the overhead of communicating with it (network latency, JSON parsing, TLS handshakes, distributed tracing) massively outweighs the computation it actually performs.
For example, a `MathService` that only has an `Add(a, b)` endpoint is too micro. If you find yourself constantly updating 5 different microservices simultaneously to ship a single business feature, or if your system relies on massive distributed joins across the network to render a single UI page, your services are too micro (you've built a "distributed monolith").

#### Micro Services Architecture
> What are the pros and cons of microservice architecture?

**Expert Answer:**
*   **Pros:** Independent deployment (one team can ship without waiting for another), independent scaling (scale only the heavy video-processing service, not the entire app), technology heterogeneity (use Go for high-performance networking, Python for ML), and smaller cognitive load per repository.
*   **Cons:** Exponentially increased operational complexity. You must master Kubernetes, distributed tracing, network failure handling (circuit breakers), and eventually consistent data (Sagas). Debugging a bug that spans 6 services is incredibly difficult compared to a monolith.
