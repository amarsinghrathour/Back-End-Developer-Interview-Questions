### [[↑]](../README.md#toc) <a name='nosql'>Questions about NoSQL:</a>

#### Eventual Consistency
> What is eventual consistency?

**Expert Answer:**
Eventual consistency is a consistency model used in distributed systems (like many NoSQL databases). It guarantees that if no new updates are made to a given piece of data, eventually all accesses to that item will return the last updated value.
In practical terms, it means there is a window of time where a read might return stale data after a write. This trade-off is made to achieve extremely high availability and low latency, as the system does not block reads/writes while synchronizing data across global clusters.

#### CAP Theorem
> Brewer's Theorem, most commonly known as the CAP theorem, states that in the presence of a network partition (the P in CAP), a system's designer has to choose between consistency (the C in CAP) and availability (the A in CAP). Can you think about examples of CP, AP and CA systems?

**Expert Answer:**
*   **CP (Consistent & Partition Tolerant):** If a network node fails, the system blocks writes to ensure all remaining nodes stay consistent. (Examples: MongoDB, Redis, HBase, Zookeeper).
*   **AP (Available & Partition Tolerant):** If a node fails, the system keeps accepting reads and writes. Data may become temporarily out of sync (eventual consistency). (Examples: Cassandra, CouchDB, DynamoDB).
*   **CA (Consistent & Available):** A system that cannot tolerate network partitions. In modern distributed systems, network partitions *will* happen, so true CA systems do not exist at scale. A single-node PostgreSQL instance is effectively CA, but the moment you add replication, you must choose C or A.

#### NoSQL
> How would you explain the recent rise in interest in NoSQL?

**Expert Answer:**
The rise of NoSQL was driven by the massive data scale introduced by companies like Google, Amazon, and Facebook in the 2000s. Relational databases scale *vertically* (buying bigger, more expensive servers), which hits a hard hardware ceiling. NoSQL databases were designed from the ground up to scale *horizontally* (adding more cheap commodity servers). 
Additionally, the rise of Agile development and JSON-heavy web APIs (where Go excels) made developers prefer schema-less (or flexible schema) document databases that didn't require rigid, upfront migrations for every small feature.

#### NoSQL and Scalability
> How does NoSQL tackle scalability challenges?

**Expert Answer:**
NoSQL tackles scalability through **Sharding** (horizontal partitioning). By abandoning the rigid constraints of ACID transactions and Foreign Keys across tables, NoSQL databases can split data effortlessly.
If a user collection grows to 100 TB, a NoSQL database can transparently distribute that data across 100 different servers based on a shard key (e.g., UserID). When a Go application requests User #50, the driver routes the request directly to Server #2. Relational databases struggle immensely with this kind of automated, distributed sharding.

#### Document and Relational DBs
> When would you use a document database like MongoDB instead of a relational database like MySQL or PostgreSQL?

**Expert Answer:**
You should use a Document Database (MongoDB) when:
*   Your data naturally fits into a denormalized JSON tree structure (e.g., a "Product" document that contains a list of "Reviews").
*   Your schema is highly fluid, polymorphic, or completely unstructured (e.g., storing arbitrary event logs or user-defined custom fields).
*   You require massive horizontal write scalability that exceeds the capabilities of a single-node Postgres instance.

You should stick to Relational (PostgreSQL) when:
*   Your data is highly structured with clear, stable relationships (e.g., financial transactions, invoicing).
*   You absolutely require ACID compliance across multiple entities.
*   You need to perform complex ad-hoc queries and deep `JOIN`s.
