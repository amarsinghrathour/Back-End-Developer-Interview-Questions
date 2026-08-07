### [[↑]](../README.md#toc) <a name='web'>Questions about Web development:</a>

#### 3rd Party Cookies
> Why are first-party cookies and third-party cookies treated so differently?

**Expert Answer:**
First-party cookies are set by the domain the user is actively visiting (e.g., `github.com`). They are essential for maintaining sessions, user preferences, and shopping carts.
Third-party cookies are set by domains *other* than the one the user is visiting (e.g., embedded ads or analytics trackers like `doubleclick.net`). Because they can track a user's behavior across multiple unaffiliated websites, they are considered a massive privacy risk. Modern browsers aggressively block third-party cookies by default to prevent cross-site tracking, while preserving first-party cookies to keep the web functional.

#### API Versioning
> How would you manage Web Services API versioning?

**Expert Answer:**
There are several ways, each with trade-offs:
1.  **URI Routing (Most Common):** e.g., `/api/v1/users`. Very explicit, easy to route via load balancers. Breaks REST purity slightly because the URI should represent the resource, not the version.
2.  **Custom Request Header:** e.g., `X-API-Version: 2.0`. Keeps URIs clean but makes debugging slightly harder as you can't just share a URL.
3.  **Accept Header (Content Negotiation):** e.g., `Accept: application/vnd.company.v2+json`. The most REST-pure approach, but often too complex for average consumers to use properly.
In Go, using URI routing is trivial with routers like `chi` or `gorilla/mux` (e.g., `r.Route("/v1", func(r chi.Router) { ... })`), making it the pragmatic standard for enterprise backends.

#### SPAs
> From a backend perspective, are there any disadvantages or drawbacks on the adoption of Single Page Applications?

**Expert Answer:**
1.  **CORS Overhead:** Because the SPA and Backend are often hosted on different subdomains (or domains), the backend must handle preflight `OPTIONS` requests and CORS headers, adding latency and configuration complexity.
2.  **Stateless Auth Complexity:** SPAs usually require token-based authentication (JWTs) instead of simple server-side sessions. Managing token revocation, refresh token rotation, and XSS-safe storage (HttpOnly cookies) is significantly more complex for the backend.
3.  **Over-fetching/Under-fetching:** Traditional REST APIs designed for SPAs often return too much or too little data per view, leading to N+1 network requests from the client. This drives the backend to adopt complex patterns like GraphQL or Backend-for-Frontend (BFF).

#### Statelessness
> Why do we usually put so much effort for having stateless services? What's so good in stateless code and why and when is statefulness bad?

**Expert Answer:**
Stateless services do not store client session data between requests. Every HTTP request contains all the context needed to process it.
*   **The Good:** Horizontal scaling becomes trivial. You can spin up 100 identical Go backend instances behind a load balancer, and any instance can handle any request. It simplifies deployment and eliminates sticky-session bugs.
*   **The Bad:** Statefulness (like holding a user's cart in memory on Server A) ruins horizontal scaling. If Server A crashes, the user loses their cart. If the load balancer routes the next request to Server B, it doesn't know who the user is. State should be pushed out of the application code and into highly available data stores (Redis, PostgreSQL).

#### REST vs SOAP
> REST and SOAP: when would you choose one, and when the other?

**Expert Answer:**
*   **REST** uses standard HTTP methods, is lightweight, leverages JSON, and is highly scalable. It is the default choice for 99% of modern web APIs, public APIs, and microservices.
*   **SOAP** is an older, heavy XML-based protocol. It is highly rigid and verbose. You would *only* choose SOAP today if you are integrating with legacy enterprise systems (banking, telecom) that mandate WS-Security and strict WSDL contracts, or where the infrastructure heavily relies on ESBs (Enterprise Service Buses). For modern Go backends, if REST is insufficient, you migrate to gRPC, not SOAP.

#### MVC and MVVM
> In web development, Model-View Controller and Model-View-View-Model approaches are very common, both in the backend and in the frontend. What are they, and why are they advisable?

**Expert Answer:**
*   **MVC (Model-View-Controller):** A pattern that separates data (Model), presentation (View), and routing/input logic (Controller). In a classic web backend, the Controller receives the HTTP request, fetches the Model from the database, injects it into an HTML template (View), and returns it. It prevents "spaghetti code" where SQL queries are mixed into HTML.
*   **MVVM (Model-View-ViewModel):** Primarily a frontend pattern (Angular, Vue). The ViewModel exposes data and commands to the View via two-way data binding. It abstracts the View's state away from the Model.
*   **Advisability:** They force Separation of Concerns. However, in modern Go development, classic backend MVC is dying out in favor of purely headless APIs (Controllers returning JSON) and SPAs handling the Views.
