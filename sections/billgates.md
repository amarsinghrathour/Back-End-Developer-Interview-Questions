### [[↑]](../README.md#toc) <a name='billgates'>Bill Gates Style Questions:</a>
This section collects some weird questions along the lines of the [Manhole Cover Question](https://en.wikipedia.org/wiki/Microsoft_interview#Manhole_cover_question).

#### Mirrors
> What would happen if you put a mirror in a scanner?

**Expert Answer:**
The scanner will output a mostly black or very dark grey image, often with a bright band of light where the scanner bulb is located. A scanner works by shining a light onto a document and reading the scattered light that reflects *back* into its sensors. A mirror does not scatter light; it reflects it cleanly away at an angle (specular reflection), meaning the sensors receive almost no light back, except precisely when the bulb is perfectly aligned to reflect straight down.

#### Clones
> Imagine there's a perfect clone of yourself. Imagine that that clone is your boss. Would you like to work for him/her?

**Expert Answer:**
Yes, because communication would be flawlessly efficient. We would share the exact same context, vocabulary, and architectural biases (like preferring Go and monolithic architectures initially). However, as a company strategy, it would be terrible. An engineering team needs cognitive diversity. Two identical people will blindly agree on a flawed design and miss the exact same edge cases.

#### Revert
> Interview me.

**Expert Answer:**
"You have 3 months to ship a new critical feature. You can either (A) bolt it onto the existing messy legacy system and meet the deadline, or (B) refactor the legacy system first to make it clean, but you will miss the deadline by 2 weeks. Which do you choose, and how do you justify it to the CEO?"
*(This tests the interviewer's pragmatism vs theoretical purity).*

#### Quora
> Why are Quora's answers better than Yahoo Answers' ones?

**Expert Answer:**
Because of structural incentives and identity. Yahoo Answers was largely anonymous, which incentivized low-effort trolling. Quora required real names, allowed linking to professional credentials (like LinkedIn), and implemented a strict upvote/downvote algorithm that rewarded high-effort, authoritative answers. By tying the answer quality directly to the author's professional reputation, Quora crowdsourced quality control.

#### Cobol
> Let's play a game: defend Cobol against modern languages, and try to find as many reasonable arguments as you can.

**Expert Answer:**
1. **Mathematical Precision:** COBOL was designed for finance. Its fixed-point decimal arithmetic is perfectly precise. Modern languages (like JS) use floating-point math, which can introduce rounding errors that cost banks millions.
2. **Readability:** It reads like English. A non-technical accountant can read a COBOL script and somewhat understand the business logic.
3. **Stability:** A COBOL program written in 1978 will compile and run flawlessly today on a modern IBM Z-series mainframe. Try running a Node.js project from 3 years ago without dependency errors.

#### 10 years
> Where will you be in 10 years?

**Expert Answer:**
In 10 years, AI will likely be writing the vast majority of boilerplate code. My goal is to be operating at the architectural and product-strategy level—translating complex human business needs into system architectures that coordinate fleets of AI agents, while maintaining deep expertise in system performance, security, and the physical constraints of computing.

#### Fire me
> You are my boss and I'm fired. Inform me.

**Expert Answer:**
"Please have a seat. I'm letting you go, effective today. We've had several documented conversations over the past quarter regarding the performance expectations of this role, and unfortunately, we haven't seen the necessary improvement. HR is here to walk you through your severance package, COBRA benefits, and the offboarding process. I appreciate the work you put in while you were here, and I wish you the best in your next role." *(Keep it direct, professional, and do not argue or apologize).*

#### From scratch
> I want to refactor a legacy system. You want to rewrite it from scratch. Argument. Then, switch our roles.

**Expert Answer:**
*   **Me (Pro-Rewrite):** "The legacy system is in an unsupported language with no tests. Every new feature takes a month because the code is spaghetti. A rewrite allows us to use modern tooling (Go), microservices, and hire cheaper, younger talent who don't want to learn the legacy language. It's an investment in velocity."
*   **Me (Pro-Refactor):** "A rewrite is the biggest mistake a company can make (per Joel Spolsky). The legacy code contains 10 years of undocumented bug fixes and edge cases. If you rewrite it, you will spend 2 years chasing those same bugs, during which we ship zero features to customers. We must strangle it slowly by refactoring incrementally."

#### Telling lies
> Your boss asks you to lie to the company. What's your reaction?

**Expert Answer:**
I refuse, immediately and unequivocally. Lying compromises my professional integrity and potentially creates legal liability. If the boss insists, I escalate to HR or their superior. If the company culture condones it, I hand in my resignation. My reputation in the industry is worth more than a single job.

#### Your past self
> If you could travel back in time, which advice would you give to your younger self?

**Expert Answer:**
"Stop chasing every new JavaScript framework. Technologies change every 2 years, but fundamentals never change. Spend your time deeply mastering HTTP, TCP/IP, Database Indexing, System Architecture, and Data Structures. A developer who understands how a B-Tree index actually works on disk will easily adapt to whatever fancy new database is invented in 2030."
