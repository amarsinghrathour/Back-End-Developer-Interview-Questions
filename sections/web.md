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
