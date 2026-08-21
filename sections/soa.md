### [[↑]](../README.md#toc) <a name='soa'>Questions about Service Oriented Architecture and Microservices:</a>

#### Long-lived Transactions
> Why, in a SOA, long-lived transactions are discouraged and sagas are suggested instead?

**Expert Answer:**

**The Short Answer:** 
Long-lived transactions hold global database locks across the network, which destroys system concurrency and throughput. Sagas replace them with a series of quick, local, immediately-committed transactions.

**The Deep Dive:** 
A traditional ACID transaction holds locks on database rows until every step completes. In a distributed SOA, a "long-lived" transaction might need to wait for a 3rd party API, a manual user approval, or a slow cross-country network call. Holding a global database lock for seconds (or minutes) means no other user can access those rows, instantly crushing the application's throughput.
The Saga pattern solves this by breaking the massive transaction into a sequence of small, local transactions. Each step immediately commits to its local database, freeing the locks. If a later step fails, the Orchestrator runs *compensating transactions* to undo the previous steps, maintaining eventual consistency without ever holding a global lock.

**The Trade-offs (Pros/Cons):**
* **Pros (of Sagas):** Massive horizontal scalability; eliminates distributed deadlocks.
* **Cons:** Lack of Isolation (the "I" in ACID). Because local transactions commit immediately, other services might temporarily read "dirty" or intermediate states before a compensation runs.

**Code Example:**
```go
// A long-lived transaction (BAD for SOA)
func BadProcess() {
    tx.Begin()
    tx.Exec("UPDATE inventory SET reserved = true")
    
    // Holding the lock while waiting 5 seconds for Stripe! 
    // The entire database grinds to a halt.
    stripe.ChargeCreditCard() 
    
    tx.Commit() 
}
```

#### SOA and Micro Services
> What are the differences between SOA and microservice?

**Expert Answer:**

**The Short Answer:** 
SOA relies on a heavy, centralized Enterprise Service Bus (ESB) and often shares a monolithic database, whereas Microservices are completely decoupled, use lightweight pipes (HTTP/Kafka), and own their individual databases.

**The Deep Dive:** 
Microservices are an evolution (or a strict subset) of Service Oriented Architecture (SOA).
*   **SOA:** Typically enterprise-scale. It relies heavily on a massive central Enterprise Service Bus (ESB) for routing, message transformation, and orchestration. Services often share a massive central database. Communication is historically heavy (SOAP/XML).
*   **Microservices:** Built on the philosophy of "smart endpoints and dumb pipes." The services are completely independent and communicate via lightweight protocols (REST/gRPC/Kafka). Crucially, *each microservice completely owns its own database*. They are built around business capabilities rather than technical layers.

**The Trade-offs (Pros/Cons):**
* **Pros (Microservices):** True independent deployment and scaling.
* **Cons (Microservices):** Data duplication. Since databases aren't shared, the `User` data might be partially duplicated across the `Auth` service, `Billing` service, and `Email` service.

**Code Example:**
```yaml
# Microservices require Database per Service
# docker-compose.yml
services:
  auth-service:
    image: my-auth
    depends_on:
      - auth-db # Completely isolated DB
      
  billing-service:
    image: my-billing
    depends_on:
      - billing-db # Completely isolated DB
```

#### Versioning and Breaking Changes
> Let's talk about web services versioning, version compatibility and breaking changes.

**Expert Answer:**

**The Short Answer:** 
To handle breaking changes in microservices, you must deploy a new API version (e.g., `/v2`) alongside the old one (`/v1`), using `/v1` as a translation layer until all clients migrate.

**The Deep Dive:** 
In microservices, you cannot force all consumers (mobile apps, other internal services, 3rd party developers) to update simultaneously. 
*   **Non-Breaking Changes:** Adding a new field to a JSON response or adding a new endpoint. Consumers should be built using Postel's Law (ignore unknown fields) so these don't break them.
*   **Breaking Changes:** Renaming a field, changing a type (int to string), or removing an endpoint.
You handle this via URI versioning (or Header versioning). You run `/v1/users` and `/v2/users` simultaneously. The backend logic for `/v1` is rewritten to simply translate the old request format into the new `/v2` format, call the `/v2` internal logic, and translate the response back.

**The Trade-offs (Pros/Cons):**
* **Pros:** Zero downtime migrations; preserves backward compatibility for legacy mobile apps.
* **Cons:** Maintaining multiple API versions creates massive technical debt. You must rigorously track `/v1` usage metrics to know when it's safe to finally delete it.

**Code Example:**
```go
// V1 acts as a translation layer to V2 to prevent breaking older clients
func HandlerV1(w http.ResponseWriter, r *http.Request) {
    // V1 expects "first_name" and "last_name"
    // V2 internally expects a single "full_name"
    
    // Translate!
    internalV2User := translateV1toV2(r.Body)
    
    // Process with the new logic
    result := processV2(internalV2User)
    
    // Translate back to the V1 format for the response
    json.NewEncoder(w).Encode(translateV2toV1(result))
}
```

#### Sagas and compensations
> What's the difference between a transaction and a compensation operation in a saga, in SOA?

**Expert Answer:**

**The Short Answer:** 
A transaction is a forward-moving action committed to a database, while a compensation is a completely separate business operation designed to logically undo the effect of that transaction.

**The Deep Dive:** 
*   A **transaction** (in the context of a single Saga step) is a standard database commit (e.g., `UpdateOrderStatus(PENDING)`). Once committed, it cannot be "rolled back" via a SQL `ROLLBACK` command because the database lock was released to allow other processes to scale.
*   A **compensation** is a *new* business operation that semantically undoes the transaction (e.g., `UpdateOrderStatus(CANCELLED_DUE_TO_PAYMENT_FAILURE)`). It does not erase history or magically rewind the database state; it appends a new state that negates the previous one. If you sent a user a "Welcome" email (a transaction), the compensation isn't un-sending the email (impossible), it's sending a "Cancellation" email.

**The Trade-offs (Pros/Cons):**
* **Pros:** Allows you to undo actions across systems that do not share a database (like undoing an API call to Stripe).
* **Cons:** Designing compensating logic for every single forward action effectively doubles the amount of business logic you have to write and test.

**Code Example:**
```go
// 1. The Forward Transaction
func ReserveHotel(userID string) error {
    return db.Exec("UPDATE rooms SET status='RESERVED' WHERE id=1")
}

// 2. The Compensating Transaction (Not a SQL rollback!)
func CancelHotelReservation(userID string) error {
    return db.Exec("UPDATE rooms SET status='AVAILABLE' WHERE id=1")
}
```

#### Too Micro
> When is a microservice too micro?

**Expert Answer:**

**The Short Answer:** 
A microservice is "too micro" when the network and serialization overhead of communicating with it massively outweighs the actual business logic computation it performs.

**The Deep Dive:** 
Microservices should be built around *Bounded Contexts* (business capabilities), not individual functions. 
A `MathService` that only exposes an `Add(a, b)` endpoint is definitively too micro. 
You know your services are too small if:
1. You constantly have to update and deploy 5 different microservices simultaneously just to ship a single business feature.
2. Your system relies on massive distributed joins across the network (fetching data from 4 services) just to render a single UI page.
This anti-pattern is called a "Distributed Monolith."

**The Trade-offs (Pros/Cons):**
* **Pros (of slightly larger services/macroservices):** Lower network latency; easier to refactor internally; simpler deployment.
* **Cons (of being too micro):** Devastating latency; tracing bugs across 5 hops for a trivial action is a nightmare.

**Code Example:**
```go
// TOO MICRO: 
// 3 network hops just to get a user's full profile!
userData := GetFromUserService(id)       // HTTP call (10ms)
avatarData := GetFromImageService(id)    // HTTP call (10ms)
billingData := GetFromBillingService(id) // HTTP call (10ms)

// MACROSERVICE (Better):
// A single "UserProfile" service handles all highly-cohesive data.
fullProfile := GetFromUserProfileService(id) // 1 HTTP call (10ms)
```

#### Micro Services Architecture
> What are the pros and cons of microservice architecture?

**Expert Answer:**

**The Short Answer:** 
Microservices offer independent scaling and deployment at the cost of exponentially increased operational and architectural complexity.

**The Deep Dive:** 
Microservices solve organizational scaling problems, not just technical ones. When an engineering team grows beyond 50 people, a monolith causes merge conflicts and deployment bottlenecks. Microservices allow independent teams to move fast.
However, you are trading software complexity for operational complexity. You no longer have function calls; you have network calls, which means you must handle timeouts, retries, circuit breakers, and distributed tracing.

**The Trade-offs (Pros/Cons):**
*   **Pros:** 
    * Independent deployment (Team A ships without waiting for Team B).
    * Independent scaling (Scale the heavy video-processing service to 100 pods, keep the login service at 2 pods).
    * Technology heterogeneity (Use Go for high-performance networking, Python for Machine Learning).
*   **Cons:** 
    * Exponentially increased operational complexity (Kubernetes is mandatory).
    * Eventual consistency replaces ACID transactions.
    * Debugging a bug that spans 6 services requires advanced APM tools (like OpenTelemetry).

**Code Example:**
```go
// In a monolith, this is a safe, guaranteed function call.
email.Send(user)

// In microservices, you trade that simplicity for resilience.
// If the email service is down, we don't crash; we queue it.
func SendEmailMicroservice(user User) {
    msg, _ := json.Marshal(user)
    
    // Scale independently! We can have 50 workers processing this queue.
    kafkaProducer.Publish("email.send", msg)
}
```


#### Microservices vs Modular Monoliths
> Why is the industry seeing a backlash against microservices, and what is a Modular Monolith?

**Expert Answer:**

**The Short Answer:** 
Microservices introduced massive operational complexity (network latency, distributed transactions, deployment hell). A Modular Monolith keeps code in one deployable unit but strictly enforces internal domain boundaries.

**The Deep Dive:** 
Many startups blindly adopted microservices, resulting in a "Distributed Monolith"—50 services that all deploy together and crash if one fails. The network latency of 10 internal API calls to render a page is catastrophic. 
A Modular Monolith gives you the best of both worlds. The application compiles to a single binary and uses a single database, eliminating network latency and distributed data issues. However, the code is strictly separated into modules (e.g., `Billing`, `Shipping`) that communicate only through internal public interfaces, never by directly touching another module's database tables. When the company hits massive scale, you can easily extract a clean module into a true microservice.

**The Trade-offs (Pros/Cons):**
* **Pros:** Radically simpler to debug, test, and deploy; no network latency.
* **Cons:** Cannot scale different domains independently (e.g., you can't allocate 100 servers to just the `Billing` module).

#### Serverless Architectures (FaaS)
> When should you use Serverless Functions (AWS Lambda) versus Long-Running Containers (ECS/Kubernetes)?

**Expert Answer:**

**The Short Answer:** 
Use Serverless for highly sporadic, unpredictable workloads or event-driven tasks. Use Containers for high-throughput, consistent web traffic to avoid cold starts and high compute costs.

**The Deep Dive:** 
AWS Lambda charges you by the millisecond of execution. If you have an endpoint that receives 1 request an hour, a container costs $20/month to sit idle. A Lambda costs $0.00. Furthermore, if you get hit by a Super Bowl ad, AWS will instantly spin up 10,000 Lambdas in 2 seconds to handle it. 
However, if you have a service constantly processing 5,000 requests per second, 24/7, Lambda becomes astronomically expensive compared to a fixed cluster of containers. You also suffer from "Cold Starts"—the 1-2 second delay when AWS has to boot a brand new Lambda container to handle a new request.

**The Trade-offs (Pros/Cons):**
* **Pros:** Scale-to-zero pricing; zero infrastructure patching or OS management.
* **Cons:** Cold start latency; vendor lock-in; extremely difficult to test locally.

#### Event-Driven Architecture (Choreography)
> What are the dangers of a purely Event-Driven Architecture?

**Expert Answer:**

**The Short Answer:** 
While it perfectly decouples services, it makes the overall business workflow invisible, turning debugging into a nightmare without sophisticated tracing.

**The Deep Dive:** 
In an event-driven system, the `Checkout` service publishes an `OrderPlaced` event and immediately returns a success to the user. It has no idea who cares about that event. The `Inventory`, `Shipping`, and `Email` services all react to it. This is perfectly decoupled. 
The danger: How do you answer the CEO's question, "What happens when an order is placed?" No single piece of code defines the workflow. It only exists as an emergent property of 5 services listening to Kafka. If an order gets stuck, it is incredibly difficult to figure out *which* service failed to react to the event.

**The Trade-offs (Pros/Cons):**
* **Pros:** Services can be built, scaled, and deployed completely independently.
* **Cons:** Loss of the "Big Picture" workflow; requires massive investment in distributed tracing and observability.
