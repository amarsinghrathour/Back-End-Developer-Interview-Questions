### [[↑]](../README.md#toc) <a name='databases'>Questions about Databases:</a>

#### DB Migrations
> How would you migrate an application from a database to another, for example from MySQL to PostgreSQL? If you had to manage that project, which issues would you expect to face?

**Expert Answer:**
Migrating core database engines is incredibly risky and requires a phased approach:
1.  **Code Abstraction:** Ensure all database access in the Go application is hidden behind interfaces (e.g., a `UserRepository` interface).
2.  **Dual Write / Single Read:** Deploy code that writes to *both* MySQL and PostgreSQL, but only reads from MySQL.
3.  **Backfill:** Run a background job to copy all historical data from MySQL to PostgreSQL.
4.  **Verification:** Compare reads from both databases asynchronously to ensure data consistency.
5.  **Switchover:** Change the application configuration to read from PostgreSQL. 
6.  **Cleanup:** Remove MySQL writes and eventually decommission the database.
*Issues expected:* Differences in SQL syntax (e.g., MySQL's `AUTO_INCREMENT` vs Postgres' `SERIAL`), different transaction isolation behaviors, varying connection pool limits (Postgres often requires PgBouncer), and subtle differences in how `NULL` or timezones are handled.

#### NULL is special
> Why do databases treat null as a so special case? For example, why does `SELECT * FROM table WHERE field = null` not match records with null `field` in SQL?

**Expert Answer:**
In SQL, `NULL` does not mean "empty string" or "zero"; it means **"Unknown"** or **"Missing"**. 
If you compare an unknown value to another unknown value (`NULL = NULL`), the result is also "Unknown" (which evaluates to falsy in a `WHERE` clause), not `True`. This is based on Three-Valued Logic (True, False, Unknown). To properly check for missing data, SQL provides the `IS NULL` and `IS NOT NULL` operators.

#### ACID
> ACID is an acronym that refers to Atomicity, Consistency, Isolation and Durability, 4 properties guaranteed by a database transaction in most database engines. What do you know about this topic? Would you like to elaborate?

**Expert Answer:**
*   **Atomicity:** "All or nothing." If a transaction fails halfway (e.g., debiting an account succeeds, crediting fails), the entire transaction rolls back. 
*   **Consistency:** The database moves from one valid state to another. Constraints (foreign keys, uniqueness) are strictly enforced.
*   **Isolation:** Concurrent transactions execute as if they were sequential. (Though most DBs use lower isolation levels like "Read Committed" by default for performance, risking phantom reads).
*   **Durability:** Once a transaction is committed, it survives system crashes (usually achieved via Write-Ahead Logging or WAL).

#### Schema Migrations
> How would you manage database schema migrations? That is, how would you automate changes to database schema, as the application evolves, version after version?

**Expert Answer:**
Schema migrations must be treated as code. I would use a migration tool like `golang-migrate/migrate` or `pressly/goose`.
1.  Migrations are stored as versioned files (e.g., `001_create_users_table.up.sql` and `.down.sql`).
2.  These files are committed to version control.
3.  The migration tool tracks which versions have been applied via a tracking table in the database (e.g., `schema_migrations`).
4.  Migrations are run automatically during the CI/CD deployment pipeline *before* the new application code boots up.

#### Lazy Loading
> How is lazy loading achieved? When is it useful? What are its pitfalls?

**Expert Answer:**
Lazy loading is an ORM pattern where related entities (children) are not fetched from the database until they are explicitly accessed in the code.
*   **Useful:** When you load a massive "Order" object but rarely need its "OrderHistoryLogs". It saves initial memory and query time.
*   **Pitfalls:** It hides database queries behind simple property access (`order.Logs`). In a loop, this leads directly to the N+1 problem, silently crushing database performance. In Go, we generally avoid heavy ORMs with lazy loading in favor of explicit `JOIN` queries or tools like `sqlx` to maintain predictable performance.

#### N+1 Problem
> The so called "N + 1 problem" is an issue that occurs when code needs to load the children of a parent-child relationship with a ORMs that have lazy-loading enabled, and that therefore issue a query for the parent record, and then one query for each child record. How to fix it?

**Expert Answer:**
You fix it by using **Eager Loading**. 
Instead of doing 1 query for the parents, and N queries for the children (total N+1), you do either:
1.  **A single JOIN query:** `SELECT * FROM parents JOIN children ON...` (Though this duplicates parent data in the result set).
2.  **Two queries (The typical ORM fix):** 
    *   `SELECT * FROM parents WHERE id IN (...)`
    *   `SELECT * FROM children WHERE parent_id IN (...)` 
In Go ORMs like GORM, this is achieved using the `Preload("Children")` method.

#### Slowest Queries
> How would you find the most expensive queries in an application?

**Expert Answer:**
1.  **Database Level:** Enable slow query logs (e.g., `log_min_duration_statement` in Postgres) or use performance schemas (e.g., `pg_stat_statements`). These tools aggregate and rank queries by total execution time.
2.  **Application Level (APM):** Integrate Distributed Tracing (like OpenTelemetry or DataDog) into the Go application. By wrapping the `sql.DB` driver with a tracer, you get a beautiful flame graph of exactly which HTTP endpoints trigger which SQL queries and how long they take.
3.  **Explain Analyze:** Once the slow query is identified, run `EXPLAIN ANALYZE` on it to see if it's doing sequential table scans instead of using indexes.

#### Normalization
> In your opinion, is it always needed to use database normalization? When is it advisable to use denormalized databases?

**Expert Answer:**
Normalization (minimizing data redundancy) is the gold standard for OLTP (Online Transaction Processing) systems to ensure data integrity and fast writes.
However, it is *not* always needed. 
*   **Denormalization is advisable when:** The system is read-heavy and `JOIN`s are becoming a massive bottleneck. Storing pre-computed totals or duplicating a user's name on an invoice record (so the invoice doesn't change if the user changes their name later) are valid denormalization strategies. OLAP (Online Analytical Processing) databases and Data Warehouses are almost entirely denormalized (Star Schemas) to prioritize lightning-fast read performance.

#### Blue/Green Deployment
> One of the Continuous Integration's techniques is called Blue-Green Deployment: it consists in having two production environments, as identical as possible, and in performing the deployment in one of them while the other one is still operating, and than in safely switching the traffic to the second one after some convenient testing. This technique becomes more complicated when the deployment includes changes to the database structure or content. I'd like to discuss this topic with you.

**Expert Answer:**
Blue/Green deployments require **Backward and Forward Compatible Database Changes**. You can never drop a column or rename a column in a single deployment.
If you need to rename a column from `first_name` to `given_name`:
1.  **Deploy 1 (Database only):** Add `given_name` column. Add a trigger to copy writes from `first_name` to `given_name`.
2.  **Deploy 2 (App Code):** Deploy the new app (Green) that reads/writes to `given_name`. Switch traffic. Both old (Blue) and new (Green) apps function simultaneously during the switch.
3.  **Deploy 3 (Cleanup):** Drop the old `first_name` column and the trigger. 
This requires immense discipline but guarantees zero-downtime deployments.
