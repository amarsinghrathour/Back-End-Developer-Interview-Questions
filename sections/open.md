### [[↑]](../README.md#toc) <a name='open'>Open Ended Questions:</a>

#### Reinvent the Wheel
> When does it make sense to reinvent the wheel?

**Expert Answer:**

**The Short Answer:** 
Reinvent the wheel only when existing open-source libraries are unmaintained, massively bloated, or when your core business loop strictly requires a custom, high-performance solution.

**The Deep Dive:** 
As the saying goes: "Reinvent the wheel when the existing wheels are squares, or when you are building a Ferrari and the only wheels available are from a tractor."
You should pull in standard libraries for generic problems (like JWT parsing or JSON serialization). However, if an open-source library brings in 50 transitive dependencies and you only need 5% of its features, it increases your binary size and security attack surface. In that case, reinventing the specific function you need is safer. Furthermore, if you are building a high-frequency trading bot, a standard garbage-collected memory allocator won't work; you *must* reinvent a custom allocator.

**The Trade-offs (Pros/Cons):**
* **Pros (of reinventing):** Zero bloat; perfect fit for your specific domain; deep understanding of the underlying system.
* **Cons:** You are now solely responsible for maintaining it, fixing edge cases, and patching security vulnerabilities that the open-source community would have handled for you.

**Code Example:**
```go
// BAD: Importing a massive 10MB library just to reverse a string.
import "github.com/massive-bloat/utils" 

// GOOD: Reinventing the tiny wheel yourself to keep the binary small and safe.
func Reverse(s string) string {
    r := []rune(s)
    for i, j := 0, len(r)-1; i < j; i, j = i+1, j-1 {
        r[i], r[j] = r[j], r[i]
    }
    return string(r)
}
```

#### Not Invented Here
> Let's have a conversation about "reinventing the wheel", the "not invented here syndrome" and the "eating your own food" practice

**Expert Answer:**

**The Short Answer:** 
"Not Invented Here" is a toxic culture of rejecting external solutions, while "Dogfooding" is the healthy practice of forcing your own team to use the product you build.

**The Deep Dive:** 
*   **Not Invented Here (NIH):** A corporate culture where a company refuses to use standard, proven open-source tools (like Postgres or Redis) simply because they didn't write it. They spend millions of dollars building a proprietary, buggy database that ultimately slows down the entire engineering organization.
*   **Eating your own dog food (Dogfooding):** A mandate that a company must use its own product internally. If you are building an enterprise chat application, your engineering team *must* use it instead of Slack. It forces developers to feel the pain of their own bugs and UX flaws, drastically improving product quality.

**The Trade-offs (Pros/Cons):**
* **Pros (of Dogfooding):** Immediate, high-quality feedback loop; builds deep empathy for the customer.
* **Cons (of NIH):** Wastes massive amounts of engineering talent on non-business-critical infrastructure.

**Code Example:**
```bash
# NIH Syndrome in Action:
# Instead of `docker pull redis`, the company spends 2 years building:
./start-proprietary-in-memory-cache.sh # (It crashes every Tuesday)
```

#### Next Thing to Automate
> What's the next thing you would automate in your current workflow?

**Expert Answer:**

**The Short Answer:** 
I would automate database schema migrations in the CI/CD pipeline to remove the risk of manual human error during production deployments.

**The Deep Dive:** 
Currently, many teams run schema migrations manually via a CLI tool right before deploying the new backend binaries. This requires a developer to have production database access and relies on them remembering the exact sequence of commands. 
I would automate this by integrating a tool like `golang-migrate/migrate` directly into the GitHub Actions pipeline. When a Pull Request is merged, the pipeline automatically applies the schema migrations safely to the staging database, runs the integration tests, and then applies them to production before swapping the load balancer.

**The Trade-offs (Pros/Cons):**
* **Pros:** Eliminates manual human error; creates a perfect audit trail of when and how the database was mutated.
* **Cons:** If a migration contains a destructive command (like dropping a column), the automated pipeline will execute it without hesitation, potentially causing irreversible data loss.

**Code Example:**
```yaml
# Automating migrations in GitHub Actions
steps:
  - name: Run Database Migrations
    run: |
      migrate -path ./migrations -database $PROD_DB_URL up
```

#### Coding is Hard
> Why is writing software difficult? What makes maintaining software hard?

**Expert Answer:**

**The Short Answer:** 
Writing software is hard because it requires translating ambiguous human desires into unambiguous mathematical logic; maintaining it is hard because of the unrelenting passage of time and state changes.

**The Deep Dive:** 
*   **Writing is hard:** A stakeholder asks for a "simple search bar." The developer must translate that ambiguous request into a complex web of precise logic: Does it support typos? Does it search across all database tables? How is it paginated?
*   **Maintaining is hard:** When you write a script, it runs once. When you write a backend server, it runs for years. Over time, data schemas evolve, open-source library dependencies rot and get deprecated, and the physical state of the database diverges wildly from the assumptions made by the original code.

**The Trade-offs (Pros/Cons):**
* **Pros (of realizing this):** Developers who understand this write extensive documentation and tests.
* **Cons:** Acknowledging the difficulty of maintenance often makes developers highly hesitant to add "cool new features."

**Code Example:**
```go
// Writing is easy:
func IsAdult(age int) bool { return age >= 18 }

// Maintaining is hard (3 years later):
// "Wait, the legal age in this new country we launched in is 21!"
// "Wait, we just started storing birthdays instead of ages!"
func IsAdult(user User) bool { 
    // Must update everywhere, taking time zones into account...
}
```

#### Green Fields and Brown Fields
> Would you prefer working on green field or brown field projects? Why?

**Expert Answer:**

**The Short Answer:** 
I prefer Brown Field projects because they provide immediate, measurable business value to actual users, avoiding the "Analysis Paralysis" common in Green Field projects.

**The Deep Dive:** 
*   **Green Fields (Brand new projects):** They are exciting because you get to choose the newest, shiniest frameworks. However, they are often plagued by "Analysis Paralysis." Teams spend 3 months arguing about architecture and eventually build something the customer doesn't actually want.
*   **Brown Fields (Existing legacy projects):** They are messy, but they have *actual users* and *actual revenue*. Refactoring a messy, slow legacy codebase to be clean and performant provides immediate, massive business value. You know exactly what the system is supposed to do, so you can focus entirely on engineering excellence rather than guessing product requirements.

**The Trade-offs (Pros/Cons):**
* **Pros (Brown Field):** Clear requirements; immediate ROI on refactoring efforts.
* **Cons (Brown Field):** Dealing with undocumented "spaghetti code" can be deeply frustrating and cause burnout if management doesn't allow time for technical debt cleanup.

**Code Example:**
```go
// Brown Field Success:
// Taking a 5-second legacy API endpoint and dropping it to 50ms 
// by adding a simple Redis cache is incredibly satisfying.
func LegacyEndpoint() {
    // Replaced 100 lines of complex SQL with:
    data := redis.Get("cached_data")
}
```

#### Type "Google.com"
> What happens when you type google.com into your browser and press enter?

**Expert Answer:**

**The Short Answer:** 
The browser resolves the domain via DNS, establishes a secure TCP/TLS connection, sends an HTTP GET request, and parses the returned HTML to render the page.

**The Deep Dive:** 
1.  **DNS Resolution:** The browser checks its cache. If empty, it asks the OS, which asks the ISP's DNS resolver to find the IP address for `google.com` (e.g., `142.250.190.46`).
2.  **TCP Handshake:** The browser opens a TCP connection to that IP address on port 443 (HTTPS) using a 3-way handshake (SYN, SYN-ACK, ACK).
3.  **TLS Handshake:** The browser and server negotiate encryption algorithms and exchange cryptographic keys to establish a secure tunnel.
4.  **HTTP Request:** The browser sends an encrypted HTTP `GET /` request through the tunnel.
5.  **Server Response:** Google's load balancer routes the request to a backend Go/C++ server, which generates the HTML and sends it back.
6.  **Rendering:** The browser parses the HTML, builds the DOM tree, requests additional CSS/JS assets, executes the JavaScript, and paints the pixels to the screen.

**The Trade-offs (Pros/Cons):**
* **Pros (of this complex dance):** Creates a highly robust, secure, and distributed global network.
* **Cons:** Every step introduces latency, which is why CDNs (Content Delivery Networks) are required to put servers physically closer to users.

**Code Example:**
```bash
# You can see the DNS portion of this process manually:
dig google.com
# Output: google.com. 300 IN A 142.250.190.46
```

#### While idle
> What does an operating system do when it has got no custom code to run, and therefore it looks idle?

**Expert Answer:**

**The Short Answer:** 
The OS executes a specialized "idle loop" instruction that puts the CPU into a low-power sleep state until a hardware interrupt wakes it up.

**The Deep Dive:** 
When all user-space processes (like your web browser and Go server) are blocked waiting for I/O, the OS kernel scheduler has nothing to do. It executes an architectural instruction (like the `hlt` instruction on x86 processors). 
This puts the CPU into a low-power state, halting instruction execution and saving massive amounts of electricity and heat. The CPU sits there doing absolutely nothing until a hardware **Interrupt** occurs (e.g., the network card receives a packet, or a hardware timer ticks). The interrupt instantly wakes the CPU, the OS kernel handles the event, schedules the newly unblocked process, and resumes work.

**The Trade-offs (Pros/Cons):**
* **Pros:** Saves battery life on laptops and reduces cooling costs in massive datacenters.
* **Cons:** Waking up from deep C-states (sleep states) takes microseconds, which can introduce unacceptable jitter in ultra-low-latency high-frequency trading systems (which often disable sleep states entirely).

**Code Example:**
```assembly
// Conceptually, the OS kernel does this:
loop:
    sti     ; Enable interrupts
    hlt     ; Halt the CPU until an interrupt fires
    jmp loop; Repeat
```

#### Unicode
> Explain Unicode and database transactions to a 5 year old child.

**Expert Answer:**

**The Short Answer:** 
Unicode is a giant toy box with a block for every letter and picture in the world. A transaction is like a fair trade of Pokémon cards where both kids must let go at the exact same time.

**The Deep Dive:** 
*   **Unicode:** "Imagine you only had blocks with English letters on them. You couldn't spell your friend's name if they were from Japan or Russia. Unicode is a giant toy box that has a specific block for every single letter and picture (emoji) in the whole world, so everyone can play together."
*   **Transactions:** "Imagine we are trading Pokémon cards. I give you my Charizard, and you give me your Blastoise. A transaction means we both have to let go at the exact same time. If one of us pulls away, neither of us gets the new card. It's either a perfectly fair trade, or nothing happens at all. The computer does this so nobody gets cheated if the power goes out during the trade."

**The Trade-offs (Pros/Cons):**
* **Pros (of using analogies):** Demonstrates deep conceptual understanding; essential for communicating with non-technical stakeholders or PMs.
* **Cons:** Over-simplification can mask the actual technical complexity (like isolation levels in transactions).

**Code Example:**
```go
// Unicode allows this to compile and print perfectly in Go!
fmt.Println("Hello, 世界 🌍") 
```

#### Defending Monoliths
> Defend the monolithic architecture.

**Expert Answer:**

**The Short Answer:** 
Monoliths are infinitely easier to deploy, test, and debug than microservices, and a well-written monolith can easily handle massive enterprise scale.

**The Deep Dive:** 
A monolith is a single executable containing all business logic. 
By using a monolith, you completely eliminate the "Fallacies of Distributed Computing." You don't have to worry about network latency between services, distributed transactions (Sagas), or complex circuit breakers. A single Go monolith compiled into one binary, backed by a robust PostgreSQL database, can easily handle millions of dollars in revenue and thousands of requests per second. 
Microservices primarily solve organizational problems (scaling engineering teams beyond 50+ people to prevent merge conflicts), not technical ones. Until your engineering department is massive, stick to a modular monolith.

**The Trade-offs (Pros/Cons):**
* **Pros:** Simple CI/CD pipeline; trivial end-to-end testing; easy to trace bugs.
* **Cons:** A memory leak in the "Image Processing" module will crash the entire application, taking down the "Checkout" module with it.

**Code Example:**
```go
// In a monolith, this is a safe, guaranteed, in-memory function call.
// No network timeouts or JSON serialization required.
err := checkoutModule.ProcessOrder(user)
```

#### Professional Developers
> What does it mean to be a "professional developer"?

**Expert Answer:**

**The Short Answer:** 
A professional developer understands business value, communicates technical tradeoffs clearly, and has the courage to say "No" to protect the integrity of the system.

**The Deep Dive:** 
If a product manager demands a massive feature in 2 weeks that will take 4 weeks to build safely, an amateur says "I'll try," works weekends, and ships broken, unmaintainable code. 
A professional developer says "No." They explain the technical tradeoffs, provide alternative solutions (e.g., "We can hit that deadline if we cut these two sub-features"), and refuse to compromise the structural integrity of the system just to meet an arbitrary deadline. They write tests, document their code, and take responsibility for their bugs without blaming others.

**The Trade-offs (Pros/Cons):**
* **Pros:** Builds long-term trust with management; ensures the codebase remains healthy for years.
* **Cons:** Saying "No" requires excellent soft skills and political capital to avoid being labeled as "difficult."

**Code Example:**
```go
// An amateur pushes this to production to meet a deadline:
func Calc() int { return 42 /* TODO: Actually write this */ }

// A professional pushes back on the deadline to write the real logic and tests.
```

#### It's an art
> Is developing software Art, Engineering, Crafts or Science? Your opinion.

**Expert Answer:**

**The Short Answer:** 
Software development is a Craft; it blends the functional requirements of engineering with the creative problem-solving of art.

**The Deep Dive:** 
*   It's not pure **Science** because it's too messy and human-driven.
*   It's not pure **Engineering** because we lack the strict physical tolerances, standardized materials, and mathematical proofs of civil engineering.
*   It's not pure **Art** because it must serve a strict functional purpose for the business.
Like woodworking or blacksmithing, it is a **Craft**. It requires mastering tools, learning from apprenticeships (senior mentors), and taking pride in building something that is both beautiful on the inside (clean code) and highly functional on the outside.

**The Trade-offs (Pros/Cons):**
* **Pros (of the Craft mindset):** Encourages continuous learning, mentorship, and pride in one's work (The Software Craftsmanship movement).
* **Cons:** Can lead to over-engineering (spending days polishing code that doesn't actually add business value).

**Code Example:**
```go
// A craftsman cares about the small details, like aligning structs
// to minimize memory padding, even if it's just a tiny optimization.
type CleanStruct struct {
    ID     int64 // 8 bytes
    Active bool  // 1 byte
    // 7 bytes of padding automatically added here
}
```

#### People who like this also like...
> "People who like this also like... ". How would you implement this feature in an e-commerce shop?

**Expert Answer:**

**The Short Answer:** 
I would implement Collaborative Filtering by streaming purchase events into a Graph Database (like Neo4j) to traverse relationships rapidly.

**The Deep Dive:** 
Attempting to do this in a relational database (SQL) requires massive, slow, recursive `JOIN` operations. Instead, I would use a Graph Database.
The schema consists of Nodes (`Users`, `Products`) and Edges (`BOUGHT`). 
When a user views an iPhone, I query the graph: "Find all `Users` who `BOUGHT` this iPhone. Now traverse outward to find all other `Products` those `Users` `BOUGHT`. Rank those `Products` by the frequency of the connections." 
Because graph databases store relationships natively as pointers, this traversal takes milliseconds, providing highly accurate, real-time recommendations.

**The Trade-offs (Pros/Cons):**
* **Pros:** Extremely fast and naturally models complex human relationships and recommendation engines.
* **Cons:** Requires introducing a completely new database paradigm (Cypher/Gremlin query languages) and maintaining infrastructure separate from the primary SQL database.

**Code Example:**
```cypher
// A Cypher query in Neo4j to find recommendations
MATCH (p:Product {id: "iphone"})<-[:BOUGHT]-(u:User)-[:BOUGHT]->(recs:Product)
RETURN recs.name, count(*) AS frequency
ORDER BY frequency DESC LIMIT 5
```

#### Corporations vs Startups
> Why are corporations slower than startups in innovating?

**Expert Answer:**

**The Short Answer:** 
Corporations have massive existing revenue streams to protect, requiring layers of compliance and QA "red tape" that inherently kill the speed required for raw innovation.

**The Deep Dive:** 
Startups have 0 customers and 0 revenue. If they push a bug to production, nobody cares. They can move at blistering speeds, rewrite their entire stack over the weekend, and pivot immediately.
A corporation has 10 million paying customers. If they push a bug, they lose millions of dollars, breach SLAs, and end up in the news. Therefore, they build massive compliance checks, security audits, and multi-week QA testing phases to protect their existing revenue. This red tape guarantees stability, but fundamentally destroys the ability to ship experimental innovations rapidly.

**The Trade-offs (Pros/Cons):**
* **Pros (Startup):** High speed, exciting, cutting-edge technology.
* **Cons (Startup):** High stress, unstable code, high probability of company failure.
* **Pros (Corp):** Job stability, massive scale, polished products.

**Code Example:**
```bash
# Startup deployment:
git push origin main # Instantly deploys to production

# Corporate deployment:
# Wait for Security Review -> Wait for QA Signoff -> Wait for Change Advisory Board
# -> Wait for next month's Release Window
```

#### I'm proud of
> What have you achieved recently that you are proud of?

**Expert Answer:**

**The Short Answer:** 
*(Example)* I identified and resolved a complex memory leak in our core Go microservice by utilizing `pprof`, preventing daily server crashes and reducing our cloud infrastructure bill.

**The Deep Dive:** 
*(Example)* Our primary API gateway was being OOM-killed (Out Of Memory) by Kubernetes every 12 hours. The team was just scaling up the RAM to temporarily fix it. I took ownership of the issue, attached the Go `pprof` profiler to the production binary, and analyzed the heap dumps. I discovered an unclosed HTTP response body inside a deeply nested 3rd-party SDK we were using. I submitted a patch to the open-source repository and applied a fix internally. This resulted in 100% uptime for the last 30 days and reduced our AWS bill by 15% since we could scale the pod memory back down.

**The Trade-offs (Pros/Cons):**
* **Pros (of taking initiative):** Solves root causes rather than symptoms; establishes you as a senior problem solver.
* **Cons:** Deep debugging can consume days of time where you aren't shipping visible product features.

**Code Example:**
```go
// The bug I fixed: failing to close the body leaks the TCP connection
// and memory!
resp, err := http.Get("http://api.com")
if err != nil { return }
defer resp.Body.Close() // I added this one line!

// And this is how I found it:
import _ "net/http/pprof"
// go tool pprof http://localhost:8080/debug/pprof/heap
```


#### The Shift Away from Open Source (BSL/SSPL)
> Why are companies like Redis, Elastic, and HashiCorp changing their licenses away from true Open Source?

**Expert Answer:**

**The Short Answer:** 
To prevent massive cloud providers (like AWS) from taking their free software, repackaging it as a managed service, and taking all the revenue without contributing back.

**The Deep Dive:** 
For years, companies built powerful open-source tools (Elasticsearch, Redis, Terraform) using the permissive Apache or MIT licenses. AWS would then offer "Amazon Elasticsearch Service," making billions while the creators struggled to monetize. To survive, these companies shifted to licenses like the Server Side Public License (SSPL) or Business Source License (BSL). These licenses keep the code "source-available" (free for users to read and run), but explicitly ban competitors from offering the software as a managed cloud service.

**The Trade-offs (Pros/Cons):**
* **Pros:** Ensures the financial survival of the companies actually building the software.
* **Cons:** Fractures the community (leading to massive open-source forks like OpenSearch or OpenTofu); enterprise legal teams often ban the use of BSL software, stifling adoption.

#### Open Source Supply Chain Attacks
> How do attackers compromise open source projects, and how do companies defend against it?

**Expert Answer:**

**The Short Answer:** 
Attackers inject malicious code into heavily downloaded NPM or PyPI packages, compromising every company that installs them. Defense requires SBOMs (Software Bill of Materials) and strict dependency locking.

**The Deep Dive:** 
Modern applications rely on thousands of open-source dependencies. If an attacker gains access to a popular package maintainer's GitHub account (via phishing or leaked credentials), they publish a new version containing a backdoor. Millions of CI pipelines auto-download the "update," and the attackers gain access to thousands of corporate networks (e.g., the `event-stream` or `xz-utils` hacks). 
To defend, companies generate an SBOM (a cryptographic inventory of every library used), use tools like Dependabot to scan for known CVEs, and strictly lock dependency versions so malicious updates aren't auto-downloaded.

**The Trade-offs (Pros/Cons):**
* **Pros:** Drastically reduces the attack surface of the software supply chain.
* **Cons:** Constant maintenance burden to manually review, test, and upgrade third-party dependencies safely.

#### InnerSource
> What is "InnerSource" and why are large enterprises adopting it?

**Expert Answer:**

**The Short Answer:** 
InnerSource applies the culture and tooling of the Open Source community (pull requests, transparent discussions, shared ownership) entirely within the private walls of a corporation.

**The Deep Dive:** 
In traditional enterprises, code is heavily siloed. If Team A needs a feature in Team B's API, they submit a Jira ticket and wait 6 months. With InnerSource, all company code is readable by all employees. Team A simply forks Team B's repository, writes the feature themselves, and submits a Pull Request. Team B reviews it and merges it. This breaks down silos, accelerates development, and fosters a collaborative culture identical to how Linux or Kubernetes is built, but kept strictly proprietary.

**The Trade-offs (Pros/Cons):**
* **Pros:** Massively reduces cross-team bottlenecks and duplicates effort; improves code quality via wider peer review.
* **Cons:** Requires a massive cultural shift; middle managers often resist losing absolute control over their codebases.

#### The AI Copyright Dilemma
> How is Generative AI threatening the foundations of Open Source licensing?

**Expert Answer:**

**The Short Answer:** 
AI models (like GitHub Copilot) are trained on billions of lines of Open Source code, but often reproduce snippets without adhering to the required attribution licenses (like GPL), sparking massive legal battles.

**The Deep Dive:** 
Open Source is built on copyright law. A GPL license explicitly says: "You can use this code, *but* you must make your derivative project open-source too." LLMs ingest this code and then spit out near-identical functions to enterprise developers building proprietary, closed-source software. This arguably violates the open-source license. The tech industry is currently fighting over whether training an AI on licensed code constitutes "Fair Use" or massive copyright infringement.

**The Trade-offs (Pros/Cons):**
* **Pros (of AI training):** Produces powerful tools that make all developers faster.
* **Cons:** Threatens the social contract of Open Source; developers may stop publishing code publicly if they know mega-corporations will scrape it for profit without attribution.
