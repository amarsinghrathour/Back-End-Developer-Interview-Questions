### [[↑]](../README.md#toc) <a name='management'>Questions about Software Lifecycle and Team Management:</a>

#### Agility
> What is agility?

**Expert Answer:**

**The Short Answer:** 
Agility is the ability to adapt to changing requirements quickly, safely, and predictably by relying on rapid feedback loops and rigorous engineering practices.

**The Deep Dive:** 
Agility is often mistaken for "working fast without documentation." In reality, true agility is highly disciplined. It means that when a product manager pivots the requirements on a Tuesday, the engineering team can confidently ship the new feature on a Wednesday without breaking the production server. This is only possible if the team has invested heavily in CI/CD pipelines, comprehensive automated testing, and decoupled architecture. Without a safety net, "moving fast" just creates catastrophic technical debt.

**The Trade-offs (Pros/Cons):**
* **Pros:** Minimizes the cost of building the wrong thing; highly responsive to market changes; happier stakeholders.
* **Cons:** Requires immense upfront investment in DevOps and testing infrastructure before the speed benefits are realized.

**Code Example:**
```yaml
# True agility requires automated safety nets (CI/CD)
# Example GitHub Actions workflow enabling Agile deployments
name: Agile Deploy
on:
  push:
    branches: [ main ]
jobs:
  test-and-deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - run: go test -race ./... # The safety net!
    - run: ./deploy.sh production # Automated release
```

#### Legacy Code
> How would you deal with legacy code?

**Expert Answer:**

**The Short Answer:** 
To safely deal with legacy code, you must first pin down its existing behavior with Characterization Tests before attempting any refactoring.

**The Deep Dive:** 
According to Michael Feathers, "legacy code is simply code without tests." 
To modify it safely, you follow a strict pattern:
1.  **Identify Seams:** Find places where you can isolate the specific piece of logic you need to change.
2.  **Write Characterization Tests:** Write tests that simply assert what the code *currently* does, even if it's wrong (e.g., if a bug exists, write a test asserting the bug happens). You are pinning down the existing behavior to prevent regressions.
3.  **Refactor:** With the safety net in place, extract the code into clean functions.
4.  **Add Feature:** Finally, add your new logic alongside the cleaned, tested code.

**The Trade-offs (Pros/Cons):**
* **Pros:** Prevents catastrophic regressions; slowly pays down technical debt over time (the Boy Scout Rule).
* **Cons:** Makes adding a "5-minute feature" take 3 days, which can frustrate management who don't understand the risks.

**Code Example:**
```go
// 1. The Legacy Code (No tests, complex)
func process(x int) int { return x * 2 /* imagine 1000 lines */ }

// 2. The Characterization Test (Pinning behavior)
func TestProcess(t *testing.T) {
    if process(5) != 10 { t.Fail() }
}

// 3. The Safe Refactor
func calculateTax(x int) int { return x * 2 }
func process(x int) int { return calculateTax(x) }
```

#### Legacy Code ELI5
> Say I'm your project manager, and I'm no expert in programming. Would you try explaining to me what legacy code is and why should I care about code quality?

**Expert Answer:**

**The Short Answer:** 
"Legacy code is like a wobbly Jenga tower; if we don't occasionally rebuild the foundation, eventually adding one tiny block will make the whole thing collapse."

**The Deep Dive:** 
"Imagine building a house out of Jenga blocks. Initially, it's fast and easy to slap blocks on top. But as the tower gets taller and more tangled, every time we want to add a new room (a new feature), we have to carefully pull blocks from the bottom. It becomes terrifyingly slow and dangerous. 'Legacy code' is that wobbly Jenga tower. If we don't spend time occasionally rebuilding the foundation (improving code quality), eventually, adding one tiny feature will make the entire system collapse, and all development will permanently halt."

**The Trade-offs (Pros/Cons):**
* **Pros (of using metaphors):** Bridges the communication gap between engineering and business; helps secure budget for technical debt.
* **Cons:** Over-simplifies the actual technical challenges (like database migrations or dependency upgrades) which can lead to unrealistic timelines.

**Code Example:**
```go
// Not applicable for ELI5 metaphors, but a good PM appreciates metrics!
// Show the PM the cyclomatic complexity dropping:
// Before refactor: Complexity 45 (Unmaintainable)
// After refactor:  Complexity 5  (Maintainable)
```

#### Sell me Kanban
> I'm the CEO of your company. Explain to me Kanban and convince me to invest in it.

**Expert Answer:**

**The Short Answer:** 
Kanban is a system that ruthlessly exposes bottlenecks and enforces Work-In-Progress (WIP) limits, guaranteeing that we stop starting new things and actually start finishing them.

**The Deep Dive:** 
"Instead of working in 2-week sprints (Scrum) where we plan big, stressful batches of work, Kanban visualizes the continuous flow of every feature across a board. Most importantly, it enforces 'WIP limits'. If QA is overloaded, developers are legally not allowed to start new coding tasks; they must stop and help QA clear the backlog. This prevents 'inventory buildup' (code that is written but not deployed). It drastically reduces our Time-To-Market and creates a smooth, predictable delivery pipeline without the artificial pressure of sprint deadlines."

**The Trade-offs (Pros/Cons):**
* **Pros:** Highly flexible to priority changes; reduces burnout; highlights exact organizational bottlenecks.
* **Cons:** Lacks the structured "reset" points of Scrum, requiring high discipline to prevent tasks from aging on the board forever.

**Code Example:**
```javascript
// A conceptual Kanban configuration enforcing WIP limits
const kanbanBoard = {
    columns: [
        { name: "To Do", wipLimit: Infinity },
        { name: "In Progress", wipLimit: 3 }, // Only 3 active coding tasks!
        { name: "Code Review", wipLimit: 2 }, // Bottleneck prevention
        { name: "Done", wipLimit: Infinity }
    ]
}
```

#### Agile vs Waterfall
> What is the biggest difference between Agile and Waterfall?

**Expert Answer:**

**The Short Answer:** 
The fundamental difference is the length of the feedback loop: Waterfall delays customer feedback until the end of a long project, while Agile requires continuous feedback every few weeks.

**The Deep Dive:** 
In Waterfall, requirements are locked in upfront. The feedback loop from the customer happens at the very end of a 12-month project. If the market changed or you misunderstood the requirements, you just wasted a year of engineering time.
In Agile, the feedback loop is 1 to 2 weeks. You build a tiny, working slice of the software (the MVP), show it to the customer, and pivot immediately if they don't like it. Agile minimizes the cost of being wrong by assuming that requirements will inevitably change.

**The Trade-offs (Pros/Cons):**
* **Pros (of Agile):** Highly adaptable; delivers value (ROI) to users much earlier.
* **Pros (of Waterfall):** Provides strict budgetary predictability and rigid timelines, which is often legally required for government or hardware contracts.

**Code Example:**
```go
// Waterfall mindset: Build the entire monolith before shipping.
func BuildSystem() {
    buildDatabase()
    buildAuth()
    buildUI()
    shipToCustomer() // 1 year later
}

// Agile mindset: Ship a tiny slice, get feedback.
func AgileSprint() {
    shipMockupUI() // 1 week later
    if customerLikesIt() {
        buildBackend()
    }
}
```

#### Death by Meetings
> Being a team manager, how would you deal with the problem of having too many meetings?

**Expert Answer:**

**The Short Answer:** 
Enforce an "Async-First" culture, establish uninterrupted "Maker Time" blocks, and require strict agendas for all synchronous meetings.

**The Deep Dive:** 
Developers require deep, uninterrupted concentration (flow state) to write good code. A 30-minute meeting in the middle of the afternoon destroys half a day of productivity.
1.  **Async First:** Force all status updates (like daily standups) into an asynchronous Slack thread. Meetings should only be for making difficult decisions, not reading reports.
2.  **Maker Time:** Block out 4 straight hours every day (e.g., 10 AM - 2 PM) where meetings are strictly prohibited organization-wide.
3.  **No Agenda, No Attendance:** If a calendar invite lacks a specific agenda and a defined goal, the team is empowered to decline it automatically.

**The Trade-offs (Pros/Cons):**
* **Pros:** Massive boost to developer velocity and morale.
* **Cons:** Async communication requires excellent written communication skills; poorly written Slack messages can lead to severe misunderstandings compared to a quick Zoom call.

**Code Example:**
```bash
# Automate the async standup via a script or Slack bot!
#!/bin/bash
echo "Automated Standup Prompt:"
echo "1. What did you ship yesterday?"
echo "2. What are you building today?"
echo "3. Are you blocked?"
# No meeting required!
```

#### Late Projects
> How would you manage a very late project?

**Expert Answer:**

**The Short Answer:** 
Stop adding people, aggressively cut scope to identify the bare Minimum Viable Product (MVP), and ship a small increment immediately to rebuild trust.

**The Deep Dive:** 
1.  **Stop adding people:** Brooks' Law states that adding manpower to a late software project makes it later (due to onboarding and communication overhead).
2.  **Cut Scope, Not Quality:** Triaging is essential. Sit with stakeholders and force them to identify the absolute MVP. Drop all "nice to have" features. Never cut testing or quality to save time, as the resulting bugs will slow the project down even further.
3.  **Ship Incrementally:** Get something—anything—into production immediately to rebuild trust with the business, even if it only does 20% of the original spec.

**The Trade-offs (Pros/Cons):**
* **Pros:** Saves the project from total cancellation; delivers core value.
* **Cons:** Requires difficult, highly political conversations with stakeholders who feel they are "losing" promised features.

**Code Example:**
```go
// Triage the codebase: Feature flags allow hiding unfinished work
// so you can ship the late project to production safely.
func DashboardHandler(w http.ResponseWriter, r *http.Request) {
    if featureflags.IsEnabled("NEW_ANALYTICS_PAGE") {
        // Unfinished, hide it from users
        renderError(w)
        return
    }
    renderLegacyDashboard(w) // Ship what works!
}
```

#### Agile Manifesto
> "Individuals and interactions over processes and tools" and "Customer collaboration over contract negotiation" comprise half of the values of the Agile Manifesto. Discuss

**Expert Answer:**

**The Short Answer:** 
These values emphasize that rigid bureaucracy (tools/contracts) should never prevent engineers from communicating directly to solve problems or adapt to user needs.

**The Deep Dive:** 
*   **Individuals over Processes:** Processes (like strict Jira workflows) often become a crutch. If a developer is blocked by a QA ticket, a rigid process says "wait for the ticket to transition state." The Agile manifesto says "Walk over to the QA engineer's desk (or hop on a huddle), talk to them (interaction), and fix it together."
*   **Customer Collaboration:** Instead of treating the customer as an adversary you are trying to trap with a rigid, signed Requirements Document, you treat them as a partner. You expect requirements to change, and you collaborate on the best way to pivot.

**The Trade-offs (Pros/Cons):**
* **Pros:** Highly autonomous, fast-moving, high-morale teams.
* **Cons:** Hard to scale across 1,000-person enterprises where some baseline processes (like compliance checklists) are legally required.

**Code Example:**
```go
// Process-heavy (Bad):
// Developer writes code, throws it over the wall, waits for Jenkins.

// Interaction-heavy (Good):
// Pair programming. Two individuals interacting to solve the problem instantly.
func ComplexAlgorithm() {
    // Authored by Alice and Bob pairing together
}
```

#### If I were the CTO
> Tell me what decisions would you take if you could be the CTO of your Company.

**Expert Answer:**

**The Short Answer:** 
I would mandate CI/CD, standardize the technology stack to reduce cognitive load, and institute dedicated time for paying down technical debt.

**The Deep Dive:** 
1.  **Mandate CI/CD:** No code gets merged without passing automated tests. "Works on my machine" is no longer an acceptable excuse.
2.  **Standardize the Stack:** Reduce cognitive load by choosing boring, effective tech (e.g., Go for backend, PostgreSQL for DB, React for frontend). This allows developers to move fluidly between teams without learning a bespoke framework every time.
3.  **Dedicated Tech Debt Time:** Institute a "20% time" (like Google) or a bi-monthly "Hack Week" to allow engineers to aggressively pay down technical debt and foster innovation without product managers interfering.

**The Trade-offs (Pros/Cons):**
* **Pros:** Massively increases engineering stability and long-term retention.
* **Cons:** Standardizing the stack might frustrate developers who want to use the newest, trendiest language; 20% time means 20% fewer product features shipped in the short term.

**Code Example:**
```bash
# CTO Mandate: Standardized tooling across all repos
# Every repository must have a Makefile with these exact commands:
make test
make lint
make build
make run
# Developers never have to guess how to start a project again.
```

#### PMs
> Are program managers useful?

**Expert Answer:**

**The Short Answer:** 
Great Product/Program Managers are highly useful because they act as "shielders" who clarify requirements and manage dependencies, keeping engineers focused on writing code.

**The Deep Dive:** 
A great PM protects the engineering team from upper management noise. They take vague business requirements ("Make the app more engaging") and translate them into actionable, edge-case-tested acceptance criteria. They manage external dependencies (e.g., coordinating a release with the marketing and legal teams). 
A bad PM simply acts as a router, forwarding emails to engineers and asking "is it done yet?" every day during standup.

**The Trade-offs (Pros/Cons):**
* **Pros (of great PMs):** Engineers can spend 90% of their time in a flow state writing code, instead of answering stakeholder emails.
* **Cons:** If a PM is non-technical, they might promise impossible timelines to stakeholders without consulting engineering first.

**Code Example:**
```javascript
// A good PM writes clear Acceptance Criteria that translates directly to tests:
describe("User Login", () => {
    it("should lock account after 5 failed attempts (AC #1)", () => {})
    it("should send an email alert upon lockout (AC #2)", () => {})
})
```

#### Team Organization
> Organize a development team using flexible schedules (that is, no imposed working hours) and "take as you need" vacation policy

**Expert Answer:**

**The Short Answer:** 
To succeed without strict hours, a team must adopt an extreme **Async-First Culture** heavily reliant on meticulous documentation and output-based performance metrics.

**The Deep Dive:** 
1.  **Documentation is God:** Because developers are sleeping while others are coding, you cannot rely on tribal knowledge or Zoom calls. All technical decisions must be recorded in Architecture Decision Records (ADRs) and detailed Jira tickets.
2.  **Overlapping Core Hours:** Require everyone to be online for a tiny window (e.g., 2 hours a day) for absolutely necessary synchronous pairing or unblocking.
3.  **Output over Input:** Evaluate performance purely by pull requests shipped, bugs fixed, and features delivered, not by hours logged with a green dot on Slack.

**The Trade-offs (Pros/Cons):**
* **Pros:** Attracts top-tier global talent; incredible work-life balance and retention.
* **Cons:** Requires highly senior, self-motivating engineers. Junior developers who need constant hand-holding will fail in this environment.

**Code Example:**
```markdown
# ADR 004: Using Redis for Caching
* **Date:** 2023-10-24
* **Status:** Accepted
* **Context:** We need faster read times. (Async communication record)
* **Decision:** We will use Redis over Memcached because...
```

#### Turn Over
> How would you manage a very high turn over and convince developers not to leave the team, without increasing compensation? What could a company improve to make them stay?

**Expert Answer:**

**The Short Answer:** 
Developers leave toxic cultures and bad managers. To retain them without raising pay, provide extreme autonomy, opportunities for mastery, and psychological safety.

**The Deep Dive:** 
1.  **Autonomy:** Give them ownership over the architecture. Present them with the business problem, but let *them* choose how to solve it technically.
2.  **Mastery:** Pay for conference tickets, provide O'Reilly subscriptions, and give them dedicated time during the week to learn new languages (like Go or Rust) that advance their careers.
3.  **Psychological Safety:** Foster a blame-free post-mortem culture. When a developer breaks production, treat it as a failure of the CI/CD system, not a personal failure. Never publicly shame an engineer.

**The Trade-offs (Pros/Cons):**
* **Pros:** Builds intense loyalty and creates a high-functioning, fearless engineering culture.
* **Cons:** None. Treating engineers like human beings instead of code-monkeys is always a net positive for the company.

**Code Example:**
```go
// Blame-free culture in code reviews:
// BAD: "You wrote this wrong, you forgot to check the error."
// GOOD: "Nice approach! Do you think we should add an error check here 
// just in case the network drops? What do you think?"
```

#### Qualities
> What are the top 3 qualities you look for in colleagues, beyond their code?

**Expert Answer:**

**The Short Answer:** 
I look for Low Ego, High Empathy, and Insatiable Curiosity.

**The Deep Dive:** 
1.  **Low Ego:** They do not attach their self-worth to their Pull Requests. They are happy to throw away their code if a teammate presents a better, simpler solution.
2.  **Empathy:** They write comprehensive documentation, clear commit messages, and readable code because they genuinely care about the developer who has to maintain it 6 months later.
3.  **Curiosity:** When a weird bug happens in production, they don't just apply a band-aid fix to close the ticket. They dive deep into the source code of the underlying framework to understand *why* it happened.

**The Trade-offs (Pros/Cons):**
* **Pros:** Creates a highly collaborative, non-toxic team environment.
* **Cons:** Sometimes highly empathetic developers struggle to give harsh-but-necessary critical feedback during code reviews.

**Code Example:**
```go
// An empathetic developer writes code for others, not themselves.
// BAD:
// x := get()

// GOOD:
// Fetch the user's active billing profile.
// Note: This returns nil if the user is legacy (pre-2020).
billingProfile := fetchActiveBillingProfile(userID)
```

#### 3 Things About Code
> What are the top 3 things you wish non-technical people knew about code?

**Expert Answer:**

**The Short Answer:** 
1. Coding is creative work (like writing a book). 2. Maintenance is 90% of the cost. 3. "Just adding a button" is rarely just a button.

**The Deep Dive:** 
1.  **It's creative, not assembly-line work:** Interrupting a developer for a "quick 5-minute question" destroys the fragile mental context they spent an hour building. It takes 30 minutes to get back into the flow state.
2.  **Maintenance is the real cost:** Building the V1 feature is cheap. Keeping it running, secure, library-updated, and compatible for the next 5 years is where the true cost lies. Every new feature adds permanent weight.
3.  **The "Just a Button" fallacy:** A button in the UI might require altering the database schema, writing an API migration, updating security roles, altering the mobile app, and writing dozens of tests.

**The Trade-offs (Pros/Cons):**
* **Pros (of educating stakeholders):** Better project timelines and a mutual respect between product and engineering.
* **Cons:** Business stakeholders might misinterpret "maintenance cost" as engineering incompetence.

**Code Example:**
```html
<!-- The stakeholder sees this: -->
<button>Export to PDF</button>

<!-- The engineer sees: 
1. Adding a message queue worker.
2. Installing a headless Chrome binary on the server.
3. Uploading to S3.
4. Sending an async WebSocket notification to the frontend.
-->
```

#### 1 month's revolution
> Imagine your company gives you 1 month and some budget to improve your and your colleagues' daily life. What would you do?

**Expert Answer:**

**The Short Answer:** 
I would invest entirely in Developer Experience (DX) by automating local environments, speeding up the CI pipeline, and buying premium hardware.

**The Deep Dive:** 
1.  **Local Dev:** I would containerize the entire stack so any new hire can run `docker-compose up` and have the database, frontend, and backend running locally in 5 minutes, rather than spending 3 days installing dependencies.
2.  **CI Speed:** If tests take 20 minutes to run, developers switch context and lose focus. I would invest time parallelizing tests and optimizing Docker builds to get the feedback loop under 3 minutes.
3.  **Hardware:** Developer salaries are massive. Cheaping out on a $300 ergonomic chair or a 4K monitor that boosts a developer's productivity by even 5% is mathematically foolish. I would buy the team whatever gear they requested.

**The Trade-offs (Pros/Cons):**
* **Pros:** Massive boost to long-term velocity and morale.
* **Cons:** Producing zero business features for a month can be a hard sell to investors or the board of directors.

**Code Example:**
```yaml
# A 1-month revolution deliverable: A perfect docker-compose.yml
# Now, nobody has to manually install Postgres or Redis locally ever again.
version: '3'
services:
  api:
    build: .
    ports: ["8080:8080"]
  db:
    image: postgres:15
  cache:
    image: redis:alpine
```

#### Remote & Hybrid Team Culture
> How do you maintain team culture and prevent burnout in a fully remote or hybrid engineering team?

**Expert Answer:**

**The Short Answer:** 
By moving away from synchronous "forced fun" and prioritizing asynchronous communication, clear boundaries, and outcome-based performance metrics.

**The Deep Dive:** 
Culture isn't a ping-pong table; it's how a team treats each other during a Sev-1 outage. In a remote setting, I mandate asynchronous-first communication. We replace daily 30-minute standups with a slack thread. This gives engineers large blocks of uninterrupted "maker time." To prevent burnout, I strictly monitor holiday usage (forcing people to take time off) and enforce a "no Slack on weekends" policy for anyone not on-call. 

**The Trade-offs (Pros/Cons):**
* **Pros:** Empowers engineers with deep focus work; allows hiring global talent; significantly reduces meeting fatigue.
* **Cons:** Junior engineers can feel isolated and struggle to onboard without "over-the-shoulder" pairing sessions.

#### AI in the SDLC
> How do you integrate AI coding assistants into your team's workflow, and how do you measure their impact?

**Expert Answer:**

**The Short Answer:** 
I deploy AI tools to eliminate boilerplate and toil, measuring success by the reduction in PR cycle time and developer satisfaction, rather than raw lines of code produced.

**The Deep Dive:** 
AI (like Copilot, Cursor, or Gemini) is a pair programmer, not an autonomous agent. I encourage its use for writing unit tests, scaffolding CRUD operations, and generating regex. However, I mandate stricter code reviews, as AI can confidently hallucinate subtle bugs or security flaws. I explicitly do *not* measure impact by "Lines of Code" (which AI inflates). Instead, I track DORA metrics (specifically Lead Time for Changes) and use developer surveys (the SPACE framework) to measure if the team feels less bogged down by repetitive tasks.

**The Trade-offs (Pros/Cons):**
* **Pros:** Drastic reduction in time spent on mundane tasks; faster unblocking when learning new frameworks.
* **Cons:** Risk of "AI fatigue" during code reviews (reviewing massive blocks of AI-generated code); potential IP/security concerns depending on the vendor.

#### Platform Engineering vs DevOps
> When does it make sense to transition from a "you build it, you run it" DevOps model to a dedicated Platform Engineering team?

**Expert Answer:**

**The Short Answer:** 
You shift to Platform Engineering when developer cognitive load becomes so high that they spend more time configuring Kubernetes and Terraform than writing business logic.

**The Deep Dive:** 
Early on, full-stack DevOps is great. But as an organization scales to 50+ engineers, forcing every frontend developer to understand IAM roles and Helm charts destroys velocity. Platform Engineering treats the internal developers as customers. The Platform team builds an Internal Developer Portal (IDP) that provides "Golden Paths"—pre-configured, compliant templates for deploying a new service with one click. 

**The Trade-offs (Pros/Cons):**
* **Pros:** Massively reduces cognitive load for product engineers; ensures standardized security and compliance across all services.
* **Cons:** Platform teams can become an accidental bottleneck if they mandate strict compliance without providing self-service tools.

#### Managing Low Performers (PIPs)
> Walk me through how you handle a consistently underperforming engineer.

**Expert Answer:**

**The Short Answer:** 
I address the issue early with clear, documented feedback, attempt to remove any external blockers, and if necessary, execute a fair, objective, and time-bound Performance Improvement Plan (PIP).

**The Deep Dive:** 
Underperformance is usually a symptom of a mismatched role, personal issues, or unclear expectations. Step one is a private 1-on-1: "Your PR output has dropped, and the last three had major bugs. Is everything okay?" If it's a skill gap, I provide pairing and mentorship. If the behavior continues, I move to a PIP. A good PIP is not a firing mechanism; it is a rigid, 30-day plan with weekly milestones that are objective (e.g., "Complete 3 tickets from the core queue with zero rollback incidents"). 

**The Trade-offs (Pros/Cons):**
* **Pros:** Protects the high performers on the team from having to constantly clean up after the underperformer; protects the company legally.
* **Cons:** PIPs are incredibly stressful for the employee and drain massive amounts of time and emotional energy from the manager.

#### DORA Metrics & The SPACE Framework
> How do you measure engineering productivity without creating a toxic, gamified culture?

**Expert Answer:**

**The Short Answer:** 
I track system-level metrics (DORA) to measure pipeline efficiency and survey-based metrics (SPACE) to measure human well-being, strictly avoiding individual stack-ranking.

**The Deep Dive:** 
Measuring "commits per day" or "story points" is toxic because developers will instantly game the system by making tiny, meaningless commits. Instead, I use DORA metrics (Deployment Frequency, Lead Time, MTTR, Change Failure Rate) at the *team* level to find bottlenecks in our CI/CD pipeline. To balance this, I use the SPACE framework, focusing on Satisfaction and well-being via anonymous pulse surveys. If Deployment Frequency is high but Satisfaction is low, we are burning the team out and need to slow down.

**The Trade-offs (Pros/Cons):**
* **Pros:** Provides an objective, holistic view of both technical velocity and human burnout.
* **Cons:** Requires sophisticated tooling to accurately extract DORA metrics from Jira/GitHub without manual data entry.
