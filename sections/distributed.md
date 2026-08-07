### [[↑]](../README.md#toc) <a name='distributed'>Questions about Distributed Systems:</a>

#### Testing Distributed Systems
> How would you test a distributed system?

**Expert Answer:**
Testing a single Go service is easy, but testing a distributed system requires:
1.  **Contract Testing:** Using tools like Pact to ensure Service A's expectations match Service B's API without spinning both up.
2.  **Chaos Engineering:** Randomly terminating pods in Kubernetes, injecting artificial network latency, and killing databases in staging (Chaos Monkey) to verify the system self-heals.
3.  **End-to-End Tracing:** Firing a request at the API Gateway and using OpenTelemetry to verify the request successfully traverses 5 different microservices and updates the database correctly.

#### Async Communication
> When would you apply asynchronous communication between two systems?

**Expert Answer:**
Async communication (message queues, event buses) should be used when:
*   The work being requested takes a long time (e.g., video processing, sending emails) and the caller shouldn't wait for it to finish.
*   You need strict decoupling: The caller doesn't need to know *who* handles the event, it just broadcasts "UserCreated".
*   You need to protect downstream systems from traffic spikes (Load Leveling). A Kafka queue absorbs a massive spike of 10k requests/sec, and the Go worker process consumes them at a safe 500 requests/sec.

#### Pitfalls of RPC
> What are the general pitfalls of remote procedure calls?

**Expert Answer:**
RPC (like gRPC or JSON-RPC) tries to make network calls look like local function calls. The pitfalls include:
1.  **Ignoring Latency:** A local call takes nanoseconds. An RPC call takes milliseconds (or seconds if the network is bad). Treating them the same causes performance collapse.
2.  **Hidden Failures:** A local call rarely fails unless there's a panic. An RPC call can fail due to timeouts, network partitions, or DNS issues. The caller must implement complex retry logic with exponential backoff.
3.  **Coupling:** It tightly couples the client and server to a specific interface definition (like Protobuf).

#### Design of Distributed Systems
> If you are building a distributed system for scalability and robustness, what are the different things you'd think of if you are working in a closed and secure network environment versus when you are working in a geographically distributed and public system?

**Expert Answer:**
*   **Closed/Secure (Datacenter):** Latency is sub-millisecond. Bandwidth is nearly infinite. You can rely heavily on fast synchronous RPCs (gRPC) between services. Security is handled at the perimeter (VPN/VPC), so internal TLS might be skipped for performance (though zero-trust is preferred today).
*   **Geographically Distributed (Public Web):** Latency is high (100ms+ cross-Atlantic). Bandwidth is limited. Network partitions are guaranteed. You must design for failure using asynchronous replication, eventual consistency, heavy CDN caching, and strict mutual TLS (mTLS) for every single service-to-service communication.

#### Fault Tolerance
> How would you manage fault tolerance in a web application? What about in a desktop one?

**Expert Answer:**
*   **Web Application (Backend):** Implement retries with exponential backoff, circuit breakers (to stop hammering a failing database), rate limiting, and deploy the Go binaries across multiple availability zones behind a load balancer. If a node dies, the load balancer routes around it.
*   **Desktop Application:** Fault tolerance relies on local persistence. If the network drops, queue the user's actions locally (e.g., in SQLite). If the app crashes, ensure it writes a state-recovery file so it can resume exactly where it left off upon restart.

#### Failures
> How would you deal with failures in a distributed system?

**Expert Answer:**
Assume failure is the normal state.
1.  **Timeouts:** Never make a network call without a timeout. In Go, always pass a `context.WithTimeout` to database and HTTP calls.
2.  **Circuit Breakers:** If an external payment API fails 5 times in a row, the circuit "opens" and immediately returns an error for the next 30 seconds without even trying the network, giving the API time to recover.
3.  **Dead Letter Queues (DLQ):** If an async message fails processing 3 times, move it to a DLQ for manual inspection so the queue isn't blocked.

#### Network Partitions
> Let's talk about the several approaches to reconciliation after network partitions.

**Expert Answer:**
When a network split heals, diverged data must be reconciled:
1.  **Last Write Wins (LWW):** Using timestamps. Simple, but clock drift can cause data loss.
2.  **Vector Clocks:** Tracks the causal history of updates. If a conflict occurs, the system asks the application (or user) to resolve it (like Git merge conflicts). DynamoDB uses this.
3.  **CRDTs (Conflict-free Replicated Data Types):** Specialized mathematical data structures (like a G-Counter) that automatically and deterministically merge without conflicts, regardless of the order the updates are received.

#### Fallacies of Distributed Computing
> What are the fallacies of distributed computing?

**Expert Answer:**
First described by L. Peter Deutsch, they are false assumptions programmers make when moving from a single machine to a network:
1. The network is reliable.
2. Latency is zero.
3. Bandwidth is infinite.
4. The network is secure.
5. Topology doesn't change.
6. There is one administrator.
7. Transport cost is zero.
8. The network is homogeneous.
Ignoring these leads to fragile systems that crash on their first day in production.

#### Request/Reply vs Publish/Subscribe
> When would you use request/reply and when publish/subscribe?

**Expert Answer:**
*   **Request/Reply (HTTP/gRPC):** Use when you immediately need a response to proceed. (e.g., "Authenticate this password". I cannot proceed until I know if it's correct).
*   **Publish/Subscribe (Kafka/RabbitMQ):** Use when the publisher doesn't care about the result, and multiple independent systems need to react. (e.g., "OrderPlaced". The Shipping service prepares a box, the Email service sends a receipt, and the Analytics service increments a counter. The Order service just fires the event and moves on).

#### Implement Transactions
> Suppose the system you are working on does not support transactionality. How would you implement it from scratch?

**Expert Answer:**
I would implement the **Saga Pattern** using a coordinator (or orchestration).
Since I don't have database-level ACID transactions, I break the transaction into a sequence of local transactions. If Step 1 (Deduct Funds) and Step 2 (Reserve Inventory) succeed, but Step 3 (Charge Credit Card) fails, I execute **Compensating Transactions** in reverse order:
*   Compensate Step 2: Release Inventory.
*   Compensate Step 1: Refund Funds.
This ensures eventual consistency without requiring a global lock.
