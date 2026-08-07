### [[↑]](../README.md#toc) <a name='billgates'>Bill Gates Style Questions:</a>
This section collects some weird questions along the lines of the [Manhole Cover Question](https://en.wikipedia.org/wiki/Microsoft_interview#Manhole_cover_question).

#### Mirrors
> What would happen if you put a mirror in a scanner?

**Expert Answer:**

**The Short Answer:** 
The scanner will output a mostly black or very dark grey image, often with a bright band of light where the scanner bulb was exactly aligned with the sensors.

**The Deep Dive:** 
A scanner works by shining a bright light onto a document. White paper scatters that light (diffuse reflection) in all directions, so plenty of it bounces back into the scanner's optical sensors. A mirror does not scatter light; it reflects it cleanly away at an angle (specular reflection). Because the light bounces away from the sensors rather than into them, the sensors register an absence of light, resulting in a black image.

**The Trade-offs (Pros/Cons):**
* **Pros (of this question):** Tests basic understanding of physics and how hardware actually works under the hood.
* **Cons:** Has absolutely nothing to do with software engineering.

**Code Example:**
```go
// A purely conceptual joke representation
func Scan(object string) string {
    if object == "paper" {
        return "Image Data"
    } else if object == "mirror" {
        return "0x000000" // Pure Black
    }
    return "Error"
}
```

#### Clones
> Imagine there's a perfect clone of yourself. Imagine that that clone is your boss. Would you like to work for him/her?

**Expert Answer:**

**The Short Answer:** 
I would enjoy it for the flawless communication, but it would be terrible for the company's long-term success due to a lack of cognitive diversity.

**The Deep Dive:** 
Working for a clone would be incredibly efficient. We would share the exact same context, vocabulary, and architectural biases (like preferring Go and monolithic architectures initially). There would be zero friction in code reviews. 
However, as a company strategy, it is a disaster. Engineering teams require cognitive diversity. Two identical people will blindly agree on a flawed design and miss the exact same edge cases. A good boss pushes you to consider perspectives you naturally ignore.

**The Trade-offs (Pros/Cons):**
* **Pros:** Blazing fast development velocity in the short term.
* **Cons:** High probability of structural failure in the long term due to groupthink.

**Code Example:**
```go
// In Go, this is why we don't want clones (shallow copies)
type Developer struct { BlindSpots []string }

me := &Developer{BlindSpots: []string{"Security", "CSS"}}
clone := me // They share the same blind spots!
```

#### Revert
> Interview me.

**Expert Answer:**

**The Short Answer:** 
"You have 3 months to ship a critical feature. Do you bolt it onto the messy legacy system and meet the deadline, or refactor the legacy system first and miss the deadline by 2 weeks? Justify your choice."

**The Deep Dive:** 
This question forces the interviewer into a difficult scenario with no perfect answer, testing their pragmatism versus theoretical purity. 
If they choose to bolt it on, I want to hear how they plan to mitigate the technical debt later. If they choose to refactor, I want to hear how they plan to communicate the delay to the CEO and manage stakeholder expectations.

**The Trade-offs (Pros/Cons):**
* **Pros (of this tactic):** Fails early if the interviewer is dogmatic rather than pragmatic.
* **Cons:** Can come across as aggressive if not delivered with a collaborative tone.

**Code Example:**
```go
// The choice:
func ShipFeature() {
    if deadline == "strict" {
        BoltOnAndAccrueDebt()
    } else {
        RefactorAndDelay()
    }
}
```

#### Quora
> Why are Quora's answers better than Yahoo Answers' ones?

**Expert Answer:**

**The Short Answer:** 
Quora implemented structural incentives tying answer quality directly to real-world professional reputation, whereas Yahoo Answers relied on anonymity.

**The Deep Dive:** 
Yahoo Answers was largely anonymous, which incentivized low-effort trolling. Quora fundamentally changed the architecture by requiring real names, allowing users to link their professional credentials (like LinkedIn), and implementing a strict upvote/downvote algorithm that surfaced high-effort, authoritative answers. By tying the answer quality directly to the author's ego and professional reputation, Quora successfully crowdsourced its quality control.

**The Trade-offs (Pros/Cons):**
* **Pros (Real Identities):** Drastically increases signal-to-noise ratio and civility.
* **Cons:** Discourages users from asking embarrassing or highly sensitive questions.

**Code Example:**
```go
type Answer struct {
    AuthorIsAnonymous bool
    Upvotes           int
}

// The Quora algorithm conceptually:
func Rank(a Answer) int {
    if a.AuthorIsAnonymous {
        return -100 // Heavily penalized
    }
    return a.Upvotes * 10 
}
```

#### Cobol
> Let's play a game: defend Cobol against modern languages, and try to find as many reasonable arguments as you can.

**Expert Answer:**

**The Short Answer:** 
COBOL offers flawless fixed-point mathematical precision, unparalleled backward compatibility, and is highly readable to non-technical business domain experts.

**The Deep Dive:** 
1. **Mathematical Precision:** COBOL was designed for finance. Its fixed-point decimal arithmetic is perfectly precise out of the box. Modern languages (like JS) default to floating-point math, which introduces rounding errors that cost banks millions.
2. **Readability:** It reads almost exactly like English. A non-technical accountant can read a COBOL script and somewhat verify the business logic.
3. **Stability:** A COBOL program written in 1978 will compile and run flawlessly today on a modern IBM Z-series mainframe. Try running a Node.js project from 3 years ago without encountering massive `npm` dependency errors.

**The Trade-offs (Pros/Cons):**
* **Pros:** The bedrock of the global financial system; absolute stability.
* **Cons:** Extremely verbose; impossible to hire young developers to maintain it.

**Code Example:**
```cobol
* COBOL handles money perfectly without floating point errors
  01  ACCOUNT-BALANCE  PIC S9(7)V99.
  ADD DEPOSIT TO ACCOUNT-BALANCE.
```
```go
// The Go equivalent to avoid floating point errors requires a 3rd party package or big.Rat
// COBOL just does it natively!
```

#### 10 years
> Where will you be in 10 years?

**Expert Answer:**

**The Short Answer:** 
I will be operating at the architectural level, translating complex business needs into system architectures that coordinate fleets of AI agents, rather than writing boilerplate code.

**The Deep Dive:** 
In 10 years, AI (like LLMs) will likely be writing the vast majority of boilerplate syntax. The role of the "Software Engineer" will shift towards "Systems Architect." My goal is to master the fundamentals that AI struggles with: deep system architecture, understanding physical computing constraints, security, and translating highly ambiguous human business desires into structured prompts and system designs.

**The Trade-offs (Pros/Cons):**
* **Pros (of this mindset):** Future-proofs your career against automation.
* **Cons:** Requires stepping away from the comfort of pure coding and heavily developing soft skills and architectural vision.

**Code Example:**
```go
// Instead of writing this manually in 10 years:
func HTTPHandler(w http.ResponseWriter, r *http.Request) { ... }

// We will likely just define the architecture:
// System: "Deploy a high-throughput API gateway that authenticates via JWT."
```

#### Fire me
> You are my boss and I'm fired. Inform me.

**Expert Answer:**

**The Short Answer:** 
"Please have a seat. I'm letting you go, effective today, due to a failure to meet the performance expectations we've discussed over the past quarter."

**The Deep Dive:** 
When firing someone, you must be direct, professional, and definitive. You do not argue, you do not apologize profusely (which can invite legal liability), and you do not leave room for negotiation. 
"We've had several documented conversations regarding the expectations of this role, and unfortunately, we haven't seen the necessary improvement. HR is here to walk you through your severance package and the offboarding process. I appreciate the work you put in, and I wish you the best."

**The Trade-offs (Pros/Cons):**
* **Pros:** Minimizes confusion; protects the company legally; respects the employee's time by ripping the band-aid off.
* **Cons:** It is inherently cold and emotionally difficult to execute.

**Code Example:**
```go
// Firing must be an atomic operation. No partial states.
func TerminateEmployee(e *Employee) {
    e.RevokeAccess()
    e.IssueSeverance()
    e.Status = "Terminated"
    // Do not engage in an infinite loop of arguments!
}
```

#### From scratch
> I want to refactor a legacy system. You want to rewrite it from scratch. Argument. Then, switch our roles.

**Expert Answer:**

**The Short Answer:** 
*Me (Pro-Rewrite):* A rewrite allows us to drop dead tech, use modern tooling (Go), and ship faster in the long run.
*Me (Pro-Refactor):* A rewrite destroys 10 years of embedded bug fixes; refactoring incrementally guarantees we keep shipping features to users.

**The Deep Dive:** 
*   **Me (Pro-Rewrite):** "The legacy system is in an unsupported language with no tests. Every new feature takes a month because the code is spaghetti. A rewrite allows us to move to Go, implement microservices, and hire cheaper, younger talent. It's a short-term pause for a massive long-term investment in velocity."
*   **Me (Pro-Refactor):** "A rewrite is the biggest mistake a company can make (per Joel Spolsky). That messy legacy code contains 10 years of undocumented bug fixes and edge cases. If you rewrite it, you will spend 2 years recreating those same bugs, during which we ship zero features to customers. We must use the Strangler Fig pattern to refactor incrementally."

**The Trade-offs (Pros/Cons):**
* **Pros (Rewrite):** Clean slate; developer happiness.
* **Cons (Rewrite):** Massive business risk; usually takes 3x longer than estimated.

**Code Example:**
```go
// Pro-Refactor uses the Strangler Pattern:
func HandleRequest(req Request) {
    if isNewFeature(req) {
        NewGoService(req) // Route to new code
    } else {
        LegacyService(req) // Fallback to old code
    }
}
```

#### Telling lies
> Your boss asks you to lie to the company. What's your reaction?

**Expert Answer:**

**The Short Answer:** 
I refuse immediately and unequivocally.

**The Deep Dive:** 
Lying to stakeholders, investors, or customers compromises professional integrity and potentially creates severe legal liability (e.g., fraud). If a boss insists, I escalate to HR or their superior. If the company culture condones it, I hand in my resignation immediately. My reputation in the industry and my personal ethics are worth infinitely more than a single job.

**The Trade-offs (Pros/Cons):**
* **Pros:** Maintains your integrity; protects you from legal fallout.
* **Cons:** You might lose your job in the short term.

**Code Example:**
```go
// Professional integrity is a boolean. It cannot be compromised.
func ExecuteOrder(order Order) error {
    if order.IsUnethical || order.IsIllegal {
        return errors.New("cannot execute: violates integrity")
    }
    return nil
}
```

#### Your past self
> If you could travel back in time, which advice would you give to your younger self?

**Expert Answer:**

**The Short Answer:** 
Stop chasing every new JavaScript framework and spend all your time deeply mastering fundamentals: HTTP, TCP/IP, Database Indexing, and Data Structures.

**The Deep Dive:** 
Technologies and frameworks change every 2 to 3 years, but the fundamentals of computer science never change. I wasted hundreds of hours learning the specific syntax of Angular 1.0, which is now entirely useless. 
A developer who understands how a B-Tree index actually works on disk, how TCP handshakes operate, and how memory is allocated will easily adapt to whatever fancy new database or language is invented in 2030. Focus on the bedrock, not the wallpaper.

**The Trade-offs (Pros/Cons):**
* **Pros (Focusing on fundamentals):** Your knowledge compounds over time and never depreciates.
* **Cons:** You might briefly fail trivia questions in interviews about the newest hipster framework.

**Code Example:**
```go
// Frameworks come and go, but the standard library is forever.
// Focus on mastering this:
import (
    "net/http"
    "database/sql"
    "fmt"
)
```
