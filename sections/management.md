### [[↑]](../README.md#toc) <a name='management'>Questions about Software Lifecycle and Team Management:</a>

#### Agility
> What is agility?

**Expert Answer:**
Agility is the ability to adapt to changing requirements quickly, safely, and predictably. It is not just "working fast without documentation." True agility requires rigorous engineering practices (TDD, CI/CD, automated deployments) so that when a product manager pivots the requirements on a Tuesday, the team can ship the new feature on a Wednesday without breaking the production server.

#### Legacy Code
> How would you deal with legacy code?

**Expert Answer:**
According to Michael Feathers, "legacy code is simply code without tests." 
To deal with it safely:
1.  **Identify Seams:** Find places where you can isolate a piece of logic.
2.  **Write Characterization Tests:** Write tests that simply assert what the code *currently* does, even if it's wrong. You are pinning down the existing behavior.
3.  **Refactor:** Now that you have a safety net, extract the code into clean functions/structs.
4.  **Add Feature:** Add your new logic alongside the cleaned, tested code.

#### Legacy Code ELI5
> Say I'm your project manager, and I'm no expert in programming. Would you try explaining to me what legacy code is and why should I care about code quality?

**Expert Answer:**
"Imagine building a house out of Jenga blocks. Initially, it's fast and easy. But as it gets taller, every time we want to add a new room (a new feature), we have to pull blocks from the bottom. It becomes terrifyingly slow and dangerous. 'Legacy code' is that wobbly Jenga tower. If we don't spend time occasionally rebuilding the foundation (improving code quality), eventually, adding one tiny feature will make the entire system collapse, and all development will permanently halt."

#### Sell me Kanban
> I'm the CEO of your company. Explain to me Kanban and convince me to invest in it.

**Expert Answer:**
"Kanban is a system that ruthlessly exposes bottlenecks. Instead of working in 2-week sprints (Scrum) where we plan big batches of work, Kanban visualizes the exact flow of every feature across a board. Most importantly, it enforces 'Work In Progress' (WIP) limits. If QA is overloaded, developers are legally not allowed to start new work; they must stop and help QA clear the backlog. It guarantees that we stop starting things, and start finishing them, drastically reducing our Time-To-Market."

#### Agile vs Waterfall
> What is the biggest difference between Agile and Waterfall?

**Expert Answer:**
**Feedback Loops.** 
In Waterfall, the feedback loop from the customer happens at the very end of a 12-month project. If you built the wrong thing, you wasted a year.
In Agile, the feedback loop is 1 to 2 weeks. You build a tiny, working slice of the software, show it to the customer, and pivot immediately if they don't like it. Agile minimizes the cost of being wrong.

#### Death by Meetings
> Being a team manager, how would you deal with the problem of having too many meetings?

**Expert Answer:**
1.  **Async First:** Force all status updates (standups) into an asynchronous Slack channel or Jira board. Meetings should only be for making decisions, not reading reports.
2.  **Core Hours:** Block out 4 straight hours every day as "Maker Time" (e.g., 10 AM - 2 PM) where meetings are strictly prohibited. Developers need continuous flow to write good code.
3.  **No Agenda, No Attendance:** If a calendar invite doesn't have a specific agenda and a defined goal, the team is empowered to decline it.

#### Late Projects
> How would you manage a very late project?

**Expert Answer:**
1.  **Stop adding people.** (Brooks' Law: Adding manpower to a late software project makes it later due to onboarding overhead).
2.  **Cut Scope, Not Quality.** Triaging is essential. Force the stakeholders to identify the absolute MVP (Minimum Viable Product). Drop all "nice to have" features.
3.  **Ship smaller increments.** Get something into production immediately to rebuild trust, even if it only does 20% of the original spec.

#### Agile Manifesto
> "Individuals and interactions over processes and tools" and "Customer collaboration over contract negotiation" comprise half of the values of the Agile Manifesto. Discuss

**Expert Answer:**
Processes and tools (like strict Jira workflows) often become a crutch. If a developer is blocked by a QA ticket, a rigid process says "wait for the ticket to transition." The Agile manifesto says "Walk over to the QA engineer's desk, talk to them (interaction), and fix it together."
Customer collaboration means you view the customer as a partner in building the product, rather than an adversary you are trying to trap with a rigid, signed Requirements Document.

#### If I were the CTO
> Tell me what decisions would you take if you could be the CTO of your Company.

**Expert Answer:**
1.  Mandate CI/CD. No code gets merged without passing automated tests.
2.  Standardize the tech stack to reduce cognitive load (e.g., Go for backend, React for frontend) to allow developers to move fluidly between teams.
3.  Institute a "20% time" or "Hack Week" policy to pay down technical debt and foster innovation without product managers interfering.

#### PMs
> Are program managers useful?

**Expert Answer:**
Yes, highly useful, if they act as "shielders" rather than "dictators." A great PM protects the engineering team from upper management noise, clarifies vague business requirements into actionable acceptance criteria, and manages external dependencies (e.g., coordinating a release with the marketing team). A bad PM just asks "is it done yet?" every day.

#### Team Organization
> Organize a development team using flexible schedules (that is, no imposed working hours) and "take as you need" vacation policy

**Expert Answer:**
This requires immense trust and an **Async-First Culture**.
1.  **Documentation is God:** Since people work at different times, all technical decisions must be recorded in Architecture Decision Records (ADRs) and Jira, not in undocumented Zoom calls.
2.  **Overlapping Core Hours:** Require everyone to be online for just 2 specific hours a day (e.g., 10am-12pm PST) for necessary synchronous pairing or meetings.
3.  **Output over Output:** Evaluate performance purely by pull requests shipped and features delivered, not by hours logged on Slack.

#### Turn Over
> How would you manage a very high turn over and convince developers not to leave the team, without increasing compensation? What could a company improve to make them stay?

**Expert Answer:**
Developers usually leave bad managers or toxic cultures, not just low pay.
1.  **Autonomy:** Give them ownership over the architecture. Let them choose how to solve the problem.
2.  **Mastery:** Pay for conference tickets, training, and give them time during the week to learn new languages (like Go or Rust).
3.  **Purpose:** Connect their work directly to user impact. Don't treat them like "ticket-closing machines."
4.  **Psychological Safety:** Foster a blame-free post-mortem culture where mistakes are treated as system failures, not personal failures.

#### Qualities
> What are the top 3 qualities you look for in colleagues, beyond their code?

**Expert Answer:**
1.  **Low Ego:** They are happy to throw away their code if a better solution is presented. They don't attach their self-worth to their PRs.
2.  **Empathy:** They write documentation, clear commit messages, and readable code because they care about the developer who comes after them.
3.  **Curiosity:** They dive deep to understand *why* a bug happened, rather than just applying a band-aid fix.

#### 3 Things About Code
> What are the top 3 things you wish non-technical people knew about code?

**Expert Answer:**
1.  **It's more like writing a book than building a wall.** It's creative work. Interruptions destroy the mental context required to write it.
2.  **90% of the cost is in maintenance.** Building the feature is cheap. Keeping it running, secure, and compatible for the next 5 years is the real cost.
3.  **"Just adding a button" is rarely just a button.** It often requires database migrations, API updates, security checks, and mobile app releases.

#### 1 month's revolution
> Imagine your company gives you 1 month and some budget to improve your and your colleagues' daily life. What would you do?

**Expert Answer:**
I would invest entirely in **Developer Experience (DX)**.
1.  Fix the local development environment so any new hire can run a single command (`docker-compose up`) and have the entire stack running in 5 minutes.
2.  Optimize the CI pipeline. If builds take 20 minutes, developers lose focus. I would parallelize tests to get the feedback loop under 3 minutes.
3.  Buy everyone the best hardware, monitors, and ergonomic chairs they request. Developer salaries are huge; cheaping out on a $300 monitor that boosts productivity by 5% is bad math.
