### [[↑]](../README.md#toc) <a name='nosql'>Questions about NoSQL:</a>

#### Eventual Consistency
> What is eventual consistency?

**Expert Answer:**

**The Short Answer:** 
Eventual consistency is a distributed systems model guaranteeing that if no new updates are made, all replicas of a database will *eventually* converge to the same value.

**The Deep Dive:** 
In highly distributed NoSQL databases (like Cassandra or DynamoDB), when a user writes data, the system quickly writes to one node and returns a success response to the user. It then asynchronously propagates that data to other nodes across the globe. There is a brief window (milliseconds to seconds) where reading from a different node might return stale data. This is the price paid for blazing-fast write performance and high availability.

**The Trade-offs (Pros/Cons):**
* **Pros:** Extremely low latency for global writes; massive high availability (the system stays up even if some nodes are partitioned).
* **Cons:** Pushes complexity to the application layer. Developers must design UIs that handle temporary stale data without confusing the user.

**Code Example:**
```go
// Handling Eventual Consistency in Go
func UpdateProfile(userID string, newBio string) {
    // 1. Write to DynamoDB (Returns immediately)
    dynamodb.PutItem(newBio)
    
    // 2. We CANNOT immediately read the data back expecting the new bio!
    // Stale read possible:
    // profile := dynamodb.GetItem(userID) 
    
    // Solution: If we need immediate read-your-own-writes consistency, 
    // we must request a ConsistentRead, which is slower and costs 2x read capacity.
    profile := dynamodb.GetItemWithConsistentRead(userID)
}
```

#### CAP Theorem
> Brewer's Theorem, most commonly known as the CAP theorem, states that in the presence of a network partition (the P in CAP), a system's designer has to choose between consistency (the C in CAP) and availability (the A in CAP). Can you think about examples of CP, AP and CA systems?

**Expert Answer:**

**The Short Answer:** 
In the event of a network failure, a CP system halts to prevent stale data, an AP system stays online but might serve stale data, and CA systems technically don't exist in distributed networks.

**The Deep Dive:** 
*   **CP (Consistent & Partition Tolerant):** If a node fails, the system blocks writes to ensure all remaining nodes stay consistent. (Examples: MongoDB, Redis Cluster, HBase, Zookeeper).
*   **AP (Available & Partition Tolerant):** If a node fails, the system keeps accepting reads and writes. Data may become temporarily out of sync (eventual consistency). (Examples: Cassandra, CouchDB, DynamoDB).
*   **CA (Consistent & Available):** A system that cannot tolerate network partitions. Because network partitions *will* happen in distributed systems, true CA systems do not exist at scale. A single-node PostgreSQL instance is effectively CA, but the moment you add replication, you must choose C or A.

**The Trade-offs (Pros/Cons):**
* **Pros of CP:** Financial/Medical data is always strictly accurate.
* **Pros of AP:** The system never goes down for users, maximizing revenue (e.g., Amazon shopping cart).

**Code Example:**
```go
// Configuring a database driver to prefer AP over CP (Cassandra example)
cluster := gocql.NewCluster("192.168.1.1", "192.168.1.2")

// Consistency QUORUM (CP approach): Requires majority of nodes to acknowledge
cluster.Consistency = gocql.Quorum 

// Consistency ONE (AP approach): Only requires ONE node to acknowledge.
// Highly available, but risks stale data if that one node dies before replicating.
cluster.Consistency = gocql.One 
```

#### NoSQL
> How would you explain the recent rise in interest in NoSQL?

**Expert Answer:**

**The Short Answer:** 
NoSQL rose to prominence because relational databases hit physical hardware limits when trying to scale for Web 2.0 traffic, and agile teams desired flexible, schema-less data structures.

**The Deep Dive:** 
The rise of NoSQL was driven by the massive data scale introduced by companies like Google, Amazon, and Facebook. Relational databases scale *vertically* (buying bigger servers), which hits a hard hardware ceiling. NoSQL databases were designed from the ground up to scale *horizontally* (adding more cheap commodity servers). 
Additionally, the rise of JSON-heavy web APIs (where Go excels) made developers prefer document databases that didn't require rigid, upfront schema migrations for every minor feature addition.

**The Trade-offs (Pros/Cons):**
* **Pros:** Effortless horizontal scaling; high developer velocity due to flexible schemas.
* **Cons:** Lack of ACID transactions across multiple documents; no foreign-key constraints leading to orphaned data; less mature ecosystem compared to SQL.

**Code Example:**
```go
// Go natively speaks JSON, making NoSQL integration seamless
type Product struct {
    ID    string                 `bson:"_id"`
    Name  string                 `bson:"name"`
    // Flexible Schema: Arbitrary metadata stored as a map
    // Impossible in strict SQL without a dedicated JSONB column
    Specs map[string]interface{} `bson:"specs"` 
}

// Storing directly into MongoDB without defining schema migrations
collection.InsertOne(ctx, product)
```

#### NoSQL and Scalability
> How does NoSQL tackle scalability challenges?

**Expert Answer:**

**The Short Answer:** 
NoSQL tackles scalability through automated **Sharding** (horizontal partitioning) across clusters of commodity servers.

**The Deep Dive:** 
By abandoning the rigid constraints of ACID transactions and Foreign Keys across tables, NoSQL databases can split data effortlessly.
If a user collection grows to 100 TB, a NoSQL database can transparently distribute that data across 100 different servers based on a shard key (e.g., `UserID`). When a Go application requests `User #50`, the database driver hashes the ID and routes the request directly to the specific server holding that data. Relational databases struggle immensely with this kind of automated, distributed sharding.

**The Trade-offs (Pros/Cons):**
* **Pros:** Theoretically infinite storage capacity and read/write throughput.
* **Cons:** Choosing a bad Shard Key leads to "hot partitions" (one server getting 99% of the traffic while others sit idle), which completely destroys performance.

**Code Example:**
```go
// The importance of Sharding Keys
type EventLog struct {
    // BAD Shard Key: Date. 
    // All traffic for "today" hits a single server (Hot Partition).
    Date string 
    
    // GOOD Shard Key: UserID. 
    // Traffic is evenly distributed across 100 servers.
    UserID string 
    
    Event string
}
```

#### Document and Relational DBs
> When would you use a document database like MongoDB instead of a relational database like MySQL or PostgreSQL?

**Expert Answer:**

**The Short Answer:** 
Use Document databases for polymorphic, unstructured data that requires massive write-scaling; use Relational databases for highly structured data requiring strict ACID transactions and complex JOINs.

**The Deep Dive:** 
You should use a Document Database (MongoDB) when:
*   Your data naturally fits into a denormalized JSON tree structure (e.g., an "Article" document that embeds an array of "Comments").
*   Your schema is highly fluid or polymorphic (e.g., a catalog where every product has entirely different attributes).
*   You require massive horizontal write scalability.

You should stick to Relational (PostgreSQL) when:
*   Your data is highly structured with stable relationships (e.g., financial transactions, invoicing).
*   You require ACID compliance across multiple entities.
*   You need to perform deep `JOIN`s and complex analytics queries.

**The Trade-offs (Pros/Cons):**
* **Pros (Document):** Intuitive mapping to object-oriented code; fast reads for hierarchical data since everything is in one document.
* **Cons (Document):** Updating a user's name requires finding and updating every single document where their name was denormalized, risking data inconsistency.

**Code Example:**
```go
// Document DB approach: Embedding children (Fast read, risky updates)
type ArticleDocument struct {
    ID       string
    Title    string
    // Comments are embedded directly. No JOIN required!
    Comments []Comment 
}

// Relational DB approach: Normalization (Safe updates, slower reads)
type ArticleRow struct {
    ID    int
    Title string
}
type CommentRow struct {
    ID        int
    ArticleID int // Foreign Key. Requires JOIN on read.
    Text      string
}
```
