### [[↑]](../README.md#toc) <a name='open'>Open Questions:</a>

#### Resistance to Change
> Why do people resist change?

**Expert Answer:**
Because change invalidates existing expertise. If a developer spent 5 years mastering Java, and the company decides to rewrite the backend in Go, that developer instantly goes from being the "expert" to a "beginner." This is terrifying. To overcome this, management must frame change not as discarding the past, but as adding a new tool to their belt, and provide dedicated paid time to learn the new technology.

#### Threading ELI5
> Explain threads to your grandparents

**Expert Answer:**
"Imagine you are cooking dinner. You are the CPU. If you cook without threads (single-threaded), you put the pasta in the boiling water, and you stand there doing nothing else for 10 minutes until it's done. Then you start chopping the tomatoes. 
If you cook *with* threads (multi-threaded), you put the pasta in the water (Thread 1), and while it's boiling, you switch to chopping the tomatoes (Thread 2). You are still just one person, but you are switching back and forth so fast that everything gets done at the same time."

#### Innovation and Predictability
> As a software engineer you want both to innovate and to be predictable. How those two goals can coexist in the same strategy?

**Expert Answer:**
Through **Timeboxing** and **Spikes**. If I want to innovate (e.g., trying out a new Go web framework instead of standard library `net/http`), I don't just start writing production code. I ask the PM for a 2-day "Spike" (a timeboxed research task). For exactly 2 days, I innovate and build a messy prototype. At the end of the 2 days, I throw the prototype away, present the findings, and *then* we predictably schedule the real implementation based on what I learned.

#### Good Code
> What makes good code good?

**Expert Answer:**
Good code is easy to delete.
It is decoupled, heavily tested, and narrowly scoped. If the business requirements change, good code allows you to delete a single struct or package without breaking the rest of the application. Clever code is rarely good code. Boring, predictable, and readable code is good code.

#### Streaming
> Explain streaming and how you would implement it.

**Expert Answer:**
Streaming is processing data in tiny chunks as it arrives, rather than loading the entire dataset into memory first.
If a user uploads a 10GB video, loading it into RAM will crash the server. In Go, you implement streaming using the `io.Reader` and `io.Writer` interfaces. You create a pipeline where data is read in 32KB buffers from the HTTP request, piped directly through a processing function, and written out to an S3 bucket, maintaining a constant, tiny memory footprint.

#### 1 Week Improvement
> Say your company gives you one week you can use to improve your and your colleagues' lifes: how would you use that week?

**Expert Answer:**
I would build an automated, hermetic local development environment. I would configure a `docker-compose.yml` or `Devcontainer` that perfectly mimics production (spinning up PostgreSQL, Redis, and the Go backend). I would write a single `make dev` script so that a new hire can clone the repo and have everything running locally in 2 minutes, without spending 3 days installing dependencies and matching versions.

#### Learnt this week
> What did you learn this week?

**Expert Answer:**
*(Example)* I learned how to use Go's `pprof` tool to trace a CPU bottleneck in a production microservice. I discovered that a seemingly innocent JSON serialization function was allocating massive amounts of memory inside a hot loop, causing heavy garbage collection pauses.

#### Aesthetic
> There is an aesthetic element to all design. The question is, is this aesthetic element your friend or your enemy?

**Expert Answer:**
It is a friend. Code aesthetics (like consistent indentation, standard casing, and vertical whitespace) reduce cognitive friction. Go enforces this strictly with `gofmt`. When code looks beautiful and uniform, the brain stops parsing the syntax and starts parsing the logic. Ugly, inconsistent code exhausts the reader before they even understand the algorithm.

#### Last 5 books
> List the last 5 books you read.

**Expert Answer:**
*(Example)*
1. *Designing Data-Intensive Applications* by Martin Kleppmann
2. *Concurrency in Go* by Katherine Cox-Buday
3. *The Phoenix Project* by Gene Kim
4. *Domain-Driven Design* by Eric Evans
5. *Clean Architecture* by Robert C. Martin

#### Introducing CI/CD
> How would you introduce Continuous Delivery in a successful, huge company for which the change from Waterfall to Continuous Delivery would be not trivial, because of the size and complexity of the business?

**Expert Answer:**
Do not mandate a "Big Bang" transition. Start with a single, low-risk, internal-facing microservice. Form a small "Tiger Team" to build a perfect CI/CD pipeline for that one service. Demonstrate that this team can ship features 10x faster with fewer bugs than the Waterfall teams. The success will breed envy. Slowly migrate other teams, using the first team as evangelists and mentors.

#### Reinvent the Wheel
> When does it make sense to reinvent the wheel?

**Expert Answer:**
When the existing wheels are squares, or when you are building a Ferrari and the only wheels available are from a tractor. 
Specifically, reinvent the wheel if the existing open-source library is unmaintained, bloated with 90% of features you don't need (adding massive binary size or attack surface), or if the performance of the core business loop strictly requires a custom, hand-rolled solution (e.g., writing a custom memory allocator for a high-frequency trading bot).

#### Not Invented Here
> Let's have a conversation about "reinventing the wheel", the "not invented here syndrome" and the "eating your own food" practice

**Expert Answer:**
*   **Not Invented Here (NIH):** A toxic corporate culture where a company refuses to use standard open-source tools (like Postgres or Redis) and instead builds their own buggy, proprietary database. It wastes millions of dollars.
*   **Eating your own dog food (Dogfooding):** A healthy practice where a company uses its own product internally. If you are building a chat app for enterprise, your company *must* use it instead of Slack. It forces developers to feel the pain of their own bugs.

#### Next Thing to Automate
> What's the next thing you would automate in your current workflow?

**Expert Answer:**
*(Example)* I would automate database schema migrations in the CI pipeline. Currently, developers run them manually in staging. I would integrate `golang-migrate/migrate` into the GitHub Actions pipeline so that when a PR is merged, the schema migrations are applied automatically and safely to the staging database before the new binary is deployed.

#### Coding is Hard
> Why is writing software difficult? What makes maintaining software hard?

**Expert Answer:**
Writing software is hard because it requires translating ambiguous human desires into unambiguous mathematical logic. 
Maintaining software is hard because of "State" and "Time." When you write a script, it runs once. When you write a server, it runs for years. Data schemas evolve, library dependencies rot and get deprecated, and the state of the database diverges from the assumptions of the original code.

#### Green Fields and Brown Fields
> Would you prefer working on green field or brown field projects? Why?

**Expert Answer:**
I prefer **Brown Field** projects (existing codebases). 
Green fields are exciting, but they are often full of "Analysis Paralysis" where teams spend 3 months arguing about which framework to use, and eventually build something the customer doesn't actually want. Brown field projects have *actual users* and *actual revenue*. Refactoring a messy legacy codebase to be clean and performant provides immediate, massive, measurable business value.

#### Type "Google.com"
> What happens when you type google.com into your browser and press enter?

**Expert Answer:**
1. Browser checks its DNS cache. If empty, it asks the OS. The OS asks the ISP's DNS resolver to find the IP address for `google.com`.
2. Browser opens a TCP connection to that IP address on port 443.
3. TLS handshake occurs to establish a secure encrypted connection.
4. Browser sends an HTTP GET request for `/`.
5. Google's load balancer receives it, routes it to a backend server.
6. The server generates the HTML and sends the HTTP response.
7. The browser parses the HTML, builds the DOM tree, downloads CSS/JS, and renders the page.

#### While idle
> What does an operating system do when it has got no custom code to run, and therefore it looks idle?

**Expert Answer:**
The OS executes an "idle loop" (like the `hlt` instruction on x86 processors). This puts the CPU into a low-power sleep state. The CPU sits there doing absolutely nothing until a hardware **Interrupt** occurs (e.g., a network card receives a packet, or a timer ticks). The interrupt wakes the CPU, the OS kernel handles the event, schedules any necessary user-space processes (like a Go web server), and then goes back to sleep.

#### Unicode
> Explain Unicode and database transactions to a 5 year old child.

**Expert Answer:**
*   **Unicode:** "Imagine you only had blocks with English letters on them. You couldn't spell your friend's name in Japanese. Unicode is a giant toy box that has a block for every single letter and picture (emoji) in the whole world, so everyone can play together."
*   **Transactions:** "Imagine we are trading Pokémon cards. I give you my Charizard, and you give me your Blastoise. A transaction means we both have to let go at the exact same time. If one of us pulls away, neither of us gets the new card. It's either a perfectly fair trade, or nothing happens at all."

#### Defending Monoliths
> Defend the monolithic architecture.

**Expert Answer:**
A monolith is a single executable containing all business logic. 
It is infinitely easier to deploy, test, and debug. You don't have to worry about network latency, distributed transactions, or circuit breakers. A single Go monolith compiled into one binary, backed by a robust PostgreSQL database, can easily handle millions of dollars in revenue and thousands of requests per second. Microservices solve organizational problems (scaling engineering teams), not technical ones. Until you have 50+ developers, stick to a monolith.

#### Professional Developers
> What does it mean to be a "professional developer"?

**Expert Answer:**
A professional developer says "No." If a product manager demands a feature in 2 weeks that will take 4 weeks to build safely, a junior developer says "I'll try," works weekends, and ships broken code. A professional developer explains the technical tradeoffs, provides alternative solutions (cutting scope), and refuses to compromise the structural integrity of the system just to meet an arbitrary deadline.

#### It's an art
> Is developing software Art, Engineering, Crafts or Science? Your opinion.

**Expert Answer:**
It is a **Craft**. 
It's not pure Science (it's too messy and human-driven). It's not pure Engineering (we lack the strict physical tolerances and mathematical proofs of civil engineering). It's not pure Art (it must serve a strict functional purpose). Like woodworking, it requires mastering tools, learning from apprenticeships, and taking pride in building something that is both beautiful on the inside and highly functional on the outside.

#### People who like this also like...
> "People who like this also like... ". How would you implement this feature in an e-commerce shop?

**Expert Answer:**
I would use **Collaborative Filtering**. 
I would not use a relational database for this. I would stream all user purchase events into a Graph Database (like Neo4j). When a user looks at an iPhone, I traverse the graph: "Find all Users who bought this iPhone. Now find all other Products those Users bought. Rank those Products by frequency." This graph traversal takes milliseconds and provides highly accurate recommendations.

#### Corporations vs Startups
> Why are corporations slower than startups in innovating?

**Expert Answer:**
Because corporations have something to lose. Startups have 0 customers, so if they ship a bug, nobody cares. They can move at blistering speeds. A corporation has 10 million paying customers. If they ship a bug, they lose millions of dollars and end up in the news. Therefore, corporations build massive compliance, QA, and security processes (red tape) to protect their existing revenue, which inherently kills the speed required for raw innovation.

#### I'm proud of
> What have you achieved recently that you are proud of?

**Expert Answer:**
*(Example)* I recently identified a memory leak in our core Go microservice that was causing Kubernetes to OOM-kill the pods every 12 hours. By profiling the application with `pprof`, I found an unclosed HTTP response body in a 3rd party SDK. I submitted a patch to the open-source repository and fixed the bug internally, resulting in 100% uptime for the last 30 days and reducing our AWS bill by 15%.
