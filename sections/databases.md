### [[↑]](../README.md#toc) <a name='databases'>Questions about Databases:</a>

#### DB Migrations
> How would you migrate an application from a database to another, for example from MySQL to PostgreSQL? If you had to manage that project, which issues would you expect to face?

**Expert Answer:**

**The Short Answer:** 
Migrating core database engines requires a phased approach: dual-writing to both databases, backfilling historical data, verifying consistency, and finally cutting over reads to the new database.

**The Deep Dive:** 
A "Big Bang" migration (shutting down the app, copying data, and restarting) results in unacceptable downtime. Instead, you must decouple the migration. 
1. **Abstract:** Ensure all DB access in Go is hidden behind interfaces.
2. **Dual-Write:** Modify the app to write to both MySQL and Postgres, but only read from MySQL.
3. **Backfill:** Run a background worker to copy all historical data from MySQL to Postgres.
4. **Verify:** Asynchronously compare reads from both databases to ensure 100% consistency.
5. **Switchover:** Flip a feature flag so the app reads from Postgres.
6. **Cleanup:** Remove the dual-write logic and decommission MySQL.

*Expected Issues:* You will face SQL syntax differences (MySQL `AUTO_INCREMENT` vs Postgres `SERIAL`), subtle transaction isolation differences, varying connection pool behaviors, and mismatched timezone handling.

**The Trade-offs (Pros/Cons):**
* **Pros (of Phased Migration):** Zero downtime; ability to rollback instantly if issues arise; guarantees data integrity.
* **Cons:** Extremely high engineering effort; temporary increase in latency due to dual-writes; complex edge cases resolving race conditions during the backfill.

**Code Example:**
```go
// Phase 2: Dual Write implementation
type DualWriteRepo struct {
    mysql    Repository
    postgres Repository
}

func (d *DualWriteRepo) SaveUser(u User) error {
    // Write to both. If Postgres fails, log it but don't fail the request yet!
    if err := d.postgres.SaveUser(u); err != nil {
        log.Error("Postgres migration write failed", err)
    }
    // MySQL is still the source of truth
    return d.mysql.SaveUser(u)
}
```

#### NULL is special
> Why do databases treat null as a so special case? For example, why does `SELECT * FROM table WHERE field = null` not match records with null `field` in SQL?

**Expert Answer:**

**The Short Answer:** 
In SQL, `NULL` does not mean "empty" or "zero"; it fundamentally means **"Unknown"** or **"Missing"**.

**The Deep Dive:** 
Because `NULL` means "Unknown", SQL uses Three-Valued Logic (True, False, Unknown). If you compare an unknown value to another unknown value (`NULL = NULL`), the result is conceptually "Unknown". Since a `WHERE` clause only returns rows that evaluate to exactly `True`, the "Unknown" result causes the row to be filtered out. To properly check for missing data without triggering Three-Valued Logic, SQL provides the explicit `IS NULL` and `IS NOT NULL` operators.

**The Trade-offs (Pros/Cons):**
* **Pros (of NULLs):** Perfectly represents the absence of data (e.g., a user who hasn't set a birthdate yet, vs a user born on Unix epoch 0).
* **Cons:** Introduces extreme complexity in complex `JOIN`s and `NOT IN` queries; causes frustrating bugs for developers used to boolean logic.

**Code Example:**
```go
// BAD SQL Query: Returns 0 rows, even if name is NULL
// db.Query("SELECT * FROM users WHERE name = NULL")

// GOOD SQL Query: Correctly uses IS NULL
// db.Query("SELECT * FROM users WHERE name IS NULL")

// In Go, `sql.NullString` is used to safely map these database NULLs
var name sql.NullString
err := db.QueryRow("SELECT name FROM users WHERE id = 1").Scan(&name)
if name.Valid {
    fmt.Println("Name is:", name.String)
} else {
    fmt.Println("Name is NULL/Missing in DB")
}
```

#### ACID
> ACID is an acronym that refers to Atomicity, Consistency, Isolation and Durability, 4 properties guaranteed by a database transaction in most database engines. What do you know about this topic? Would you like to elaborate?

**Expert Answer:**

**The Short Answer:** 
ACID properties guarantee that database transactions are processed reliably, completely, and safely, even in the event of power failures or concurrent access.

**The Deep Dive:** 
*   **Atomicity:** "All or nothing." If a transaction fails halfway (e.g., debiting an account succeeds, but crediting fails), the entire transaction rolls back. 
*   **Consistency:** The database strictly moves from one valid state to another. Constraints (foreign keys, uniqueness) are enforced.
*   **Isolation:** Concurrent transactions execute as if they were sequential. (Though most DBs default to lower isolation levels like "Read Committed" to boost performance).
*   **Durability:** Once a transaction is committed, it survives system crashes, usually achieved via a Write-Ahead Log (WAL) written to disk before confirming success to the client.

**The Trade-offs (Pros/Cons):**
* **Pros:** Absolute data integrity; developers don't have to manually handle rollback logic for partial failures.
* **Cons:** Enforcing strict ACID (especially Serializable Isolation) severely bottlenecks performance and prevents massive horizontal scaling (leading to the rise of BASE/NoSQL databases).

**Code Example:**
```go
// Atomicity in Go: Both operations succeed, or neither do.
tx, _ := db.Begin()
defer tx.Rollback() // Automatically rolls back if not committed

_, err = tx.Exec("UPDATE accounts SET balance = balance - 100 WHERE id = 1")
if err != nil { return err }

_, err = tx.Exec("UPDATE accounts SET balance = balance + 100 WHERE id = 2")
if err != nil { return err }

// Durability and Consistency are guaranteed upon commit
return tx.Commit() 
```

#### Schema Migrations
> How would you manage database schema migrations? That is, how would you automate changes to database schema, as the application evolves, version after version?

**Expert Answer:**

**The Short Answer:** 
Schema migrations must be treated as version-controlled code and applied automatically during the CI/CD pipeline using a migration tool.

**The Deep Dive:** 
Manually executing SQL scripts on production databases is a recipe for disaster. Instead, use tools like `golang-migrate/migrate` or `pressly/goose`. Migrations are stored as sequential, versioned files (e.g., `001_create_users.up.sql` and `001_create_users.down.sql`). These files are committed to Git. The migration tool creates a `schema_migrations` tracking table in the database to remember which files have been applied. During deployment, the CI/CD pipeline runs the migration tool to apply any missing `.up.sql` files *before* the new application code boots up.

**The Trade-offs (Pros/Cons):**
* **Pros:** Deterministic, reproducible database state; easy collaboration across large teams; automated rollbacks (`.down.sql`).
* **Cons:** Requires discipline to never modify applied migration files; long-running migrations (like creating an index on a billion-row table) can block CI/CD pipelines.

**Code Example:**
```bash
# Example using golang-migrate in a CI/CD pipeline
# 1. Generate new migration files locally
migrate create -ext sql -dir db/migrations -seq add_status_to_users

# 2. In CI/CD, run the migrations against production before app start
migrate -database ${POSTGRES_URL} -path db/migrations up
```

#### Lazy Loading
> How is lazy loading achieved? When is it useful? What are its pitfalls?

**Expert Answer:**

**The Short Answer:** 
Lazy loading is an ORM pattern where related entities (like a User's Posts) are not queried from the database until they are explicitly accessed in the code.

**The Deep Dive:** 
When an ORM fetches a "User" object, it replaces the "Posts" relationship with a proxy object. Only when the developer calls `user.GetPosts()` does the ORM fire a secondary SQL query to fetch them. This is achieved via proxy classes or getter interception. 
It is useful for memory conservation when loading massive objects where child relationships are rarely accessed. However, it is a dangerous anti-pattern in modern development because it hides database I/O behind innocent-looking property access, leading directly to the notorious N+1 Problem. In Go, heavy lazy-loading ORMs are generally avoided.

**The Trade-offs (Pros/Cons):**
* **Pros:** Saves initial memory and query time; intuitive object-oriented developer experience.
* **Cons:** Hides expensive I/O operations; completely destroys performance when iterating over collections (N+1 problem); makes API response times unpredictable.

**Code Example:**
```go
// Conceptual Lazy Loading Pitfall
users := orm.GetUsers() // 1 query: SELECT * FROM users

for _, u := range users {
    // Hidden I/O! Fires a query ON EVERY ITERATION.
    // If there are 1,000 users, this executes 1,000 queries!
    fmt.Println(u.LazyLoadPosts()) 
}
```

#### N+1 Problem
> The so called "N + 1 problem" is an issue that occurs when code needs to load the children of a parent-child relationship with a ORMs that have lazy-loading enabled, and that therefore issue a query for the parent record, and then one query for each child record. How to fix it?

**Expert Answer:**

**The Short Answer:** 
Fix the N+1 problem by using **Eager Loading**, explicitly instructing the ORM to fetch all related children in a single batched query upfront.

**The Deep Dive:** 
Instead of doing 1 query for the parents, and N queries for the children inside a loop (total N+1), Eager Loading pulls everything in advance. Under the hood, the ORM either executes a single massive `JOIN` query, or it executes two highly efficient queries: 
1. `SELECT * FROM parents WHERE id IN (...)`
2. `SELECT * FROM children WHERE parent_id IN (...)` 
The ORM then stitches the objects together in memory. In Go ORMs like GORM, this is triggered using the `Preload()` method.

**The Trade-offs (Pros/Cons):**
* **Pros:** Massive performance improvements; predictable database load; eliminates hidden I/O loops.
* **Cons:** Fetches data that might not ultimately be used; single `JOIN` strategies can result in massive Cartesian products duplicating data over the network.

**Code Example:**
```go
// BAD (N+1 Problem):
// db.Find(&users)
// for _, u := range users { db.Model(&u).Association("Posts").Find(&u.Posts) }

// GOOD (Eager Loading in GORM):
// Executes exactly 2 queries, regardless of how many users exist.
var users []User
db.Preload("Posts").Find(&users)

for _, u := range users {
    // Zero database queries fired here. Data is already in memory.
    fmt.Println(u.Posts) 
}
```

#### Slowest Queries
> How would you find the most expensive queries in an application?

**Expert Answer:**

**The Short Answer:** 
Find expensive queries by enabling database-level slow query logs, utilizing Application Performance Monitoring (APM) tracing, and running `EXPLAIN ANALYZE`.

**The Deep Dive:** 
1.  **Database Level:** Enable `log_min_duration_statement` in Postgres or use `pg_stat_statements` to aggregate and rank queries by total execution time natively.
2.  **Application Level (APM):** Integrate Distributed Tracing (like OpenTelemetry). By wrapping Go's `sql.DB` driver with a tracer, you generate flame graphs showing exactly which HTTP endpoints trigger which SQL queries, and how long they take.
3.  **Analysis:** Once the slow query is caught, run `EXPLAIN ANALYZE` on it to ensure the database planner is using indexes rather than doing catastrophic Sequential Table Scans.

**The Trade-offs (Pros/Cons):**
* **Pros:** Pinpoints exactly where optimization is needed; prevents blind guessing; APM tracing connects slow DB queries to specific user actions.
* **Cons:** Tracing and logging add slight overhead to database performance; analyzing `EXPLAIN` plans requires deep specialized knowledge of database internals.

**Code Example:**
```go
// In Go, you can wrap the SQL driver with an OpenTelemetry tracer 
// to automatically record the execution time of every query.
import (
    "database/sql"
    "github.com/XSAM/otelsql"
)

// Now every query executed via `db` is timed and shipped to your APM dashboard
db, _ := otelsql.Open("postgres", dsn, otelsql.WithTracerProvider(tracerProvider))
```

#### Normalization
> In your opinion, is it always needed to use database normalization? When is it advisable to use denormalized databases?

**Expert Answer:**

**The Short Answer:** 
Normalization is critical for transactional (OLTP) systems to prevent data anomalies, but Denormalization is advisable for read-heavy systems where complex `JOIN`s become a performance bottleneck.

**The Deep Dive:** 
Normalization (minimizing data redundancy) is the gold standard for OLTP systems to ensure fast writes and strict data integrity. However, it is *not* always needed. 
**Denormalization is advisable when:** 
1. **Performance:** A high-traffic dashboard requires joining 8 tables. Storing a pre-computed "total_orders" column on the User table (denormalization) drastically speeds up reads at the cost of slightly slower writes.
2. **Immutability:** Storing a copy of a user's name on an `Invoice` record. If the user changes their name later (in the `Users` table), the historical invoice should *not* change. 
3. **Data Warehouses (OLAP):** Analytics databases use highly denormalized Star Schemas to prioritize lightning-fast aggregations.

**The Trade-offs (Pros/Cons):**
* **Pros (of Denormalization):** Blazing fast reads; simplifies complex query logic; creates historical snapshots of data.
* **Cons:** Increases storage size; introduces Write Anomalies (updating a user's profile picture requires updating it in 5 different tables); requires complex application logic to keep redundant data in sync.

**Code Example:**
```go
// Normalization: Slower read, requires JOIN
// SELECT users.name, COUNT(orders.id) FROM users JOIN orders...

// Denormalization: Fast read, but requires updating multiple places on write
type User struct {
    ID          int
    Name        string
    TotalOrders int // Denormalized field!
}

func CreateOrder(userID int) {
    db.Exec("INSERT INTO orders...")
    // Must remember to update the denormalized field!
    db.Exec("UPDATE users SET total_orders = total_orders + 1 WHERE id = ?", userID)
}
```

#### Blue/Green Deployment
> One of the Continuous Integration's techniques is called Blue-Green Deployment: it consists in having two production environments, as identical as possible, and in performing the deployment in one of them while the other one is still operating, and than in safely switching the traffic to the second one after some convenient testing. This technique becomes more complicated when the deployment includes changes to the database structure or content. I'd like to discuss this topic with you.

**Expert Answer:**

**The Short Answer:** 
Blue/Green deployments with databases require strictly **Backward and Forward Compatible** schema changes, ensuring both the old and new app versions can function simultaneously.

**The Deep Dive:** 
You can never drop or rename a column in a single deployment, because the "Blue" (old) app will instantly crash when it tries to read the missing column. 
If you need to rename `first_name` to `given_name`, you must split it into three deployments:
1.  **Deploy 1 (Database only):** Add the `given_name` column. Add a DB trigger to copy writes from `first_name` to `given_name`.
2.  **Deploy 2 (App Code):** Deploy the Green app that reads/writes to `given_name`. Switch traffic. During the switch, Blue writes to `first_name` (trigger copies to `given_name`), and Green writes to `given_name` (trigger copies back to `first_name`).
3.  **Deploy 3 (Cleanup):** Drop the old `first_name` column and the triggers.

**The Trade-offs (Pros/Cons):**
* **Pros:** True zero-downtime deployments; instant rollback capabilities (just switch the load balancer back to Blue).
* **Cons:** Immense engineering discipline required; breaks "simple" schema changes into grueling multi-week tasks; requires double the server infrastructure.

**Code Example:**
```sql
-- Deploy 1: Add the column, but don't drop the old one yet!
ALTER TABLE users ADD COLUMN given_name VARCHAR(255);

-- (Optional) Create a trigger to keep both columns in sync 
-- while both Blue and Green apps are actively receiving traffic.
CREATE TRIGGER sync_names BEFORE INSERT OR UPDATE ON users...
```


#### Distributed SQL (NewSQL)
> How do Distributed SQL databases like CockroachDB or Google Spanner achieve horizontal scale without losing ACID guarantees?

**Expert Answer:**

**The Short Answer:** 
They shard data geographically but use advanced consensus protocols (like Raft) and synchronized atomic clocks (TrueTime) to guarantee strict consistency across shards.

**The Deep Dive:** 
Traditional SQL (Postgres) is single-writer. NoSQL (Cassandra) is multi-writer but sacrifices ACID. Distributed SQL splits the difference. When you insert a row, the database determines which geographic shard should own it. It uses the Raft consensus algorithm, requiring a majority of replicas (e.g., 2 out of 3) to acknowledge the write before returning success. To prevent distributed transaction conflicts (where clocks drift on different servers), Spanner uses atomic clocks and GPS receivers to ensure absolute global time ordering.

**The Trade-offs (Pros/Cons):**
* **Pros:** The holy grail—developer-friendly SQL with the infinite scale and survivability of NoSQL.
* **Cons:** The consensus network hops make write latency significantly higher than a traditional single-node Postgres database.

#### Change Data Capture (CDC)
> Why is Change Data Capture (CDC) replacing traditional dual-writes in microservice architectures?

**Expert Answer:**

**The Short Answer:** 
Dual-writing to a database and a message queue causes inconsistencies if one fails. CDC reads the database's internal transaction log, guaranteeing 100% accurate event emission.

**The Deep Dive:** 
If an application saves a user to Postgres and then publishes a "UserCreated" event to Kafka, the network might drop the Kafka message *after* the DB commit. The systems are now permanently out of sync. 
CDC tools (like Debezium) solve this. The application *only* writes to Postgres. Debezium tails the Postgres Write-Ahead Log (WAL) at the filesystem level. When it sees the commit, Debezium publishes the event to Kafka. If Kafka goes down, Debezium waits and resumes from where it left off, guaranteeing eventual delivery without application-level complexity.

**The Trade-offs (Pros/Cons):**
* **Pros:** Absolute data consistency; completely decouples event publishing from business logic.
* **Cons:** Complex infrastructure to set up (requires Kafka, Debezium, Kafka Connect) and requires careful handling of schema evolution.

#### Database Connection Pooling at Scale
> How do serverless architectures (like AWS Lambda) break traditional database connection pools, and how do you fix it?

**Expert Answer:**

**The Short Answer:** 
Serverless functions scale massively and instantly, exhausting the database's connection limits. You fix it by placing a connection proxy (like PgBouncer or AWS RDS Proxy) between them.

**The Deep Dive:** 
A traditional monolith maintains a pool of 50 persistent connections to Postgres. If you migrate to AWS Lambda and receive a spike of 10,000 concurrent requests, AWS spins up 10,000 Lambda containers. Each container opens a new connection to Postgres. Postgres spends all its CPU trying to manage 10,000 connections and instantly crashes. 
The solution is a proxy layer (PgBouncer). All 10,000 Lambdas connect to the proxy (which is lightweight). The proxy multiplexes those 10,000 requests over a safe pool of 200 persistent connections to the actual database.

**The Trade-offs (Pros/Cons):**
* **Pros:** Prevents serverless autoscaling from DDOSing your relational database.
* **Cons:** Adds a network hop (minor latency) and introduces a new single point of failure that must be clustered.

#### MVCC (Multi-Version Concurrency Control)
> How does MVCC allow databases to avoid locking tables during concurrent reads and writes?

**Expert Answer:**

**The Short Answer:** 
MVCC creates hidden versions of rows so that "readers never block writers, and writers never block readers."

**The Deep Dive:** 
If a transaction starts updating a million rows, traditional databases would lock the table, forcing all read queries to wait (causing downtime). With MVCC (used in Postgres), the database doesn't overwrite the old row; it inserts a *new* version of the row with a higher transaction ID. 
When a read query arrives, it looks at the timestamp. It ignores the uncommitted new rows and safely reads the old rows. Both transactions proceed at full speed without waiting. Later, a background process ("Vacuuming" in Postgres) cleans up the dead rows once no active transactions need them.

**The Trade-offs (Pros/Cons):**
* **Pros:** Massive throughput for highly concurrent environments; read queries are blisteringly fast because they don't wait for locks.
* **Cons:** Updates cause "bloat" (wasted disk space) until the garbage collection (vacuum) reclaims it, which can cause CPU spikes.
