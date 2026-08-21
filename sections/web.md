### [[↑]](../README.md#toc) <a name='web'>Questions about Web development:</a>

#### 3rd Party Cookies
> Why are first-party cookies and third-party cookies treated so differently?

**Expert Answer:**

**The Short Answer:** 
First-party cookies are essential for core website functionality (like keeping you logged in), while third-party cookies are primarily used for cross-site tracking and advertising, presenting a massive privacy risk.

**The Deep Dive:** 
When you visit `github.com`, any cookie set by `github.com` is a first-party cookie. It tracks your session so you don't have to log in on every page. However, if `github.com` embeds an ad from `doubleclick.net`, that ad can set a cookie on your machine. When you later visit `nytimes.com` (which also embeds `doubleclick.net`), the ad network reads the exact same cookie, building a comprehensive profile of your browsing history across unaffiliated domains. Because of this privacy invasion, modern browsers are aggressively blocking third-party cookies by default.

**The Trade-offs (Pros/Cons):**
* **Pros (of blocking 3rd-party):** Drastically improves user privacy; stops shadow-profiling by advertising tech giants.
* **Cons:** Breaks legitimate cross-domain integrations (like embedded payment iframes or federated single-sign-on systems) which relied on third-party cookies to maintain state.

**Code Example:**
```go
// Setting a First-Party Cookie in Go safely
func LoginHandler(w http.ResponseWriter, r *http.Request) {
    cookie := http.Cookie{
        Name:     "session_id",
        Value:    "abc-123",
        Path:     "/",
        HttpOnly: true,  // Protect against XSS
        Secure:   true,  // HTTPS only
        SameSite: http.SameSiteLaxMode, // Prevents CSRF (modern 1st-party standard)
    }
    http.SetCookie(w, &cookie)
}
```

#### API Versioning
> How would you manage Web Services API versioning?

**Expert Answer:**

**The Short Answer:** 
API versioning is typically managed through URI routing (`/v1/users`), Custom Headers (`X-API-Version`), or Content Negotiation (`Accept` headers).

**The Deep Dive:** 
As APIs evolve, breaking changes (like renaming a field or changing a response structure) will destroy client applications. You must version the API.
1.  **URI Routing:** The most pragmatic approach. Easy to cache, route via load balancers, and share as links.
2.  **Custom Request Header:** Keeps URIs clean but makes debugging harder as you can't just share a URL in a browser.
3.  **Accept Header (Content Negotiation):** The most REST-pure approach, but often too complex for average API consumers to implement properly.
In Go, using URI routing is trivial and remains the enterprise standard.

**The Trade-offs (Pros/Cons):**
* **Pros (of URI Versioning):** Extreme simplicity; trivial to route traffic at the API Gateway level to different backend microservices (e.g., v1 goes to Legacy Service, v2 goes to New Service).
* **Cons:** Violates REST purity because the URI should represent the entity (the noun), not the structure of the representation.

**Code Example:**
```go
// API Versioning via URI routing using the chi router
r := chi.NewRouter()

r.Route("/api", func(r chi.Router) {
    // Legacy Version
    r.Route("/v1", func(r chi.Router) {
        r.Get("/users", handlers.GetUsersV1)
    })
    
    // Modern Version (Breaking changes)
    r.Route("/v2", func(r chi.Router) {
        r.Get("/users", handlers.GetUsersV2)
    })
})
```

#### SPAs
> From a backend perspective, are there any disadvantages or drawbacks on the adoption of Single Page Applications?

**Expert Answer:**

**The Short Answer:** 
SPAs introduce significant backend complexity regarding CORS configuration, stateless authentication (JWTs), and data over/under-fetching.

**The Deep Dive:** 
In traditional web apps, the backend renders HTML and manages simple server-side sessions. With an SPA (React/Vue), the frontend runs entirely in the browser and acts as a completely separate client. 
This forces the backend to manage CORS (Cross-Origin Resource Sharing) because the SPA and API often live on different subdomains. Furthermore, because SPAs don't play nicely with traditional session cookies across domains, backends are forced to adopt stateless JWTs, introducing severe complexity regarding token revocation and XSS-safe storage.

**The Trade-offs (Pros/Cons):**
* **Pros (of SPAs):** Incredibly snappy user experience; decouples frontend and backend teams; the backend API can be reused by mobile apps.
* **Cons (for backend devs):** Massive increase in security configuration (CORS, Preflight requests); forces complex data-fetching patterns (GraphQL/BFF) to prevent N+1 API calls from the client.

**Code Example:**
```go
// Backend penalty for SPAs: Managing complex CORS preflight rules
import "github.com/go-chi/cors"

func setupCORS() *cors.Cors {
    return cors.New(cors.Options{
        AllowedOrigins:   []string{"https://my-spa-frontend.com"},
        AllowedMethods:   []string{"GET", "POST", "PUT", "DELETE", "OPTIONS"},
        AllowedHeaders:   []string{"Accept", "Authorization", "Content-Type"},
        AllowCredentials: true, // Required if the SPA uses HttpOnly cookies
        MaxAge:           300,  // Cache preflight OPTIONS requests
    })
}
```

#### Statelessness
> Why do we usually put so much effort for having stateless services? What's so good in stateless code and why and when is statefulness bad?

**Expert Answer:**

**The Short Answer:** 
Stateless services allow for infinite, trivial horizontal scaling because any server instance can process any incoming request.

**The Deep Dive:** 
If Server A holds a user's shopping cart in local memory (Stateful), that user is now "sticky" to Server A. If Server A crashes, the cart is lost. If the load balancer routes their next request to Server B, it won't know who they are. 
By pushing state completely out of the application code and into highly available external data stores (like Redis or PostgreSQL), the web servers become interchangeable compute nodes. This is the foundation of modern Cloud-Native architecture.

**The Trade-offs (Pros/Cons):**
* **Pros:** Effortless horizontal scaling; true high availability; zero-downtime rolling deployments (killing a node affects nobody).
* **Cons:** Increases request latency (requires network hops to the external cache/DB to fetch the state on every request); centralizes the point of failure to the database layer.

**Code Example:**
```go
// BAD: Stateful (Prevents horizontal scaling)
var InMemoryCarts = make(map[string][]Item)

// GOOD: Stateless (State is externalized)
func AddToCart(w http.ResponseWriter, r *http.Request) {
    sessionID := extractSession(r)
    // Server holds no state. It reads/writes to a shared Redis cluster.
    redisClient.SAdd(ctx, "cart:"+sessionID, newItem) 
}
```

#### REST vs SOAP
> REST and SOAP: when would you choose one, and when the other?

**Expert Answer:**

**The Short Answer:** 
REST is lightweight and highly scalable, making it the default for modern web APIs, whereas SOAP is heavy and rigid, reserved almost exclusively for legacy enterprise systems.

**The Deep Dive:** 
*   **REST** relies on standard HTTP methods, typically transmits JSON, and utilizes stateless communication. It is the pragmatic standard for microservices, mobile backends, and public APIs.
*   **SOAP** is an older, XML-based protocol. It is highly verbose. You would *only* choose SOAP today if you are integrating with legacy telecom, healthcare, or banking systems that absolutely mandate WS-Security specifications and strict WSDL (Web Services Description Language) contracts. For modern high-performance backends, if REST is insufficient, the industry migrates to gRPC, not SOAP.

**The Trade-offs (Pros/Cons):**
* **Pros (of REST):** Human-readable payloads (JSON); easily cacheable via standard HTTP semantics; massive ecosystem of tooling.
* **Cons (of REST):** Lacks strict, language-enforced contracts (though OpenAPI/Swagger helps).
* **Pros (of SOAP):** Extremely strict contracts (WSDL) guaranteeing payload structure.

**Code Example:**
```go
// Modern RESTful Endpoint in Go
// Simple, uses standard HTTP methods and JSON
func GetUser(w http.ResponseWriter, r *http.Request) {
    id := chi.URLParam(r, "id")
    user := db.Fetch(id)
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(user)
}
```

#### MVC and MVVM
> In web development, Model-View Controller and Model-View-View-Model approaches are very common, both in the backend and in the frontend. What are they, and why are they advisable?

**Expert Answer:**

**The Short Answer:** 
MVC separates data (Model), presentation (View), and routing (Controller) for backends, while MVVM heavily abstracts state synchronization for complex frontend UIs.

**The Deep Dive:** 
*   **MVC:** A classic backend pattern. The Controller receives the HTTP request, fetches the Model from the database, injects it into an HTML template (View), and returns it. It prevents "spaghetti code" where SQL queries are mixed into HTML files.
*   **MVVM:** Primarily a modern frontend pattern (Angular, Vue). The ViewModel exposes data and commands to the View via automated two-way data binding, decoupling the UI from the underlying logic.
*   **Advisability:** They force Separation of Concerns. However, in modern Go development, classic backend MVC is dying out in favor of headless REST/gRPC APIs, pushing the "View" entirely to the frontend SPA.

**The Trade-offs (Pros/Cons):**
* **Pros:** Enforces separation of concerns; allows designers to work on Views while engineers work on Controllers/Models.
* **Cons:** Overkill for simple scripts; in modern architectures, monolithic MVC is often split entirely, replacing the backend View layer with a decoupled React/Vue frontend.

**Code Example:**
```go
// Classic MVC in Go (Using html/template)
// The Controller
func ProfileHandler(w http.ResponseWriter, r *http.Request) {
    // The Model
    user := db.GetUser(123) 
    
    // The View (Merging Model with HTML Template)
    tmpl, _ := template.ParseFiles("profile.html")
    tmpl.Execute(w, user) 
}
```


#### Server-Side Rendering (SSR) vs Client-Side Rendering (CSR)
> Why are modern web frameworks (like Next.js) shifting back from pure React SPAs to Server-Side Rendering?

**Expert Answer:**

**The Short Answer:** 
To fix the massive SEO penalties and slow First Contentful Paint (FCP) issues caused by forcing the user's phone to download and execute megabytes of JavaScript before rendering the UI.

**The Deep Dive:** 
A pure Client-Side SPA (Single Page Application) sends an empty HTML file (`<div id="root"></div>`) to the browser, followed by 2MB of JavaScript. The browser parses the JS, hits an API, waits for JSON, and finally renders the page. On a slow 3G phone, the user stares at a blank white screen for 5 seconds. Google's web crawler might see nothing. 
SSR (Next.js) renders the React components into fully populated HTML on the *server* and sends that down instantly. The user sees the page in milliseconds. The JavaScript then loads in the background to make the page interactive (Hydration).

**The Trade-offs (Pros/Cons):**
* **Pros:** Incredible SEO; blazingly fast perceived load times.
* **Cons:** Requires a NodeJS server running 24/7, destroying the cheap "static hosting" dream of pure SPAs; introduces complex caching challenges.

#### WebSockets vs Server-Sent Events (SSE)
> If you need to stream live stock prices to a browser, should you use WebSockets or Server-Sent Events?

**Expert Answer:**

**The Short Answer:** 
Use Server-Sent Events (SSE) because stock prices are a one-way stream of data from the server to the client. WebSockets are overkill unless you need bi-directional communication.

**The Deep Dive:** 
WebSockets open a persistent, bi-directional TCP connection. They are required for multiplayer games or chat apps where the user is constantly sending data *back* to the server. 
SSE works over standard HTTP/1.1 or HTTP/2. The browser makes a normal GET request, and the server simply keeps the connection open, pushing text data down whenever it wants. Because SSE uses standard HTTP, it automatically works with existing load balancers, corporate firewalls, and HTTP/2 multiplexing, whereas WebSockets often require special load balancer configuration (connection upgrades).

**The Trade-offs (Pros/Cons):**
* **Pros (SSE):** Simpler to implement; native browser auto-reconnection; firewall friendly.
* **Cons (SSE):** Strictly unidirectional (server to client only).

#### HTMX & The Return to HTML
> What is HTMX, and why is it gaining traction as an alternative to massive JavaScript frameworks?

**Expert Answer:**

**The Short Answer:** 
HTMX allows developers to build modern, dynamic, SPA-like experiences by sending HTML fragments directly from the server, eliminating the need for complex JSON APIs and React state management.

**The Deep Dive:** 
Over the last decade, backend developers were forced to build JSON REST APIs solely to feed React frontends. HTMX argues this is a mistake. With HTMX, a button click (`<button hx-post="/like">`) sends a request to the server. The server (written in Go, Python, or Ruby) updates the database and returns a tiny snippet of raw HTML: `<span>5 Likes</span>`. HTMX takes that HTML and swaps it into the DOM instantly. 
This allows backend engineers to build highly interactive web apps entirely in their language of choice without writing a single line of custom JavaScript.

**The Trade-offs (Pros/Cons):**
* **Pros:** Massively reduces codebase complexity; eliminates the need for duplicated state management on the client.
* **Cons:** Not suitable for highly complex offline applications or heavy canvas/WebGL games; splits UI rendering logic across backend templates.

#### HTTP/3 & QUIC
> Why is HTTP/3 replacing HTTP/2, and how does it solve the "Head-of-Line Blocking" problem?

**Expert Answer:**

**The Short Answer:** 
HTTP/3 replaces TCP with a new transport protocol called QUIC (built on UDP) to fix TCP's fatal flaw: Head-of-Line blocking over spotty mobile networks.

**The Deep Dive:** 
HTTP/2 allowed a browser to download 10 images over a single TCP connection simultaneously (multiplexing). However, TCP guarantees ordered delivery. If the very first packet of Image 1 is dropped by a bad cell tower, TCP pauses the entire connection, waiting for that one packet to be retransmitted, even if the packets for Images 2-10 have already arrived. This is Head-of-Line blocking. 
HTTP/3 uses QUIC (over UDP). If a packet for Image 1 is lost, Image 1 is delayed, but Images 2-10 finish downloading instantly because the streams are completely independent at the transport layer.

**The Trade-offs (Pros/Cons):**
* **Pros:** Drastically improves web performance on unstable mobile networks (subways, driving).
* **Cons:** UDP is sometimes blocked or throttled by aggressive corporate firewalls; requires high CPU usage for user-space cryptography.
