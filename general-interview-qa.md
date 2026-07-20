# 100 General Software Engineering Interview Questions & Answers
### For Senior Roles — Language-Agnostic

---

## Part 1: Programming Fundamentals & OOP (Q1–Q15)

**Q1. What is dependency injection?**
Instead of a class creating the objects it needs, those dependencies are **handed to it** from outside (via constructor, setter, or framework).
*Analogy:* a chef doesn't build the oven — the kitchen provides one.
*Why it matters:* you can swap a real database for a fake one in tests, or change implementations without editing the class. It's how "depend on abstractions" becomes real code.

**Q2. What are the four pillars of OOP?**
- **Encapsulation** — bundle data + behavior, hide internals behind an interface (a capsule: you see the pill, not the chemistry)
- **Abstraction** — expose *what* something does, hide *how* (you press a car pedal; you don't manage fuel injection)
- **Inheritance** — a class reuses/extends another ("is-a": Dog is an Animal)
- **Polymorphism** — one interface, many behaviors (`shape.draw()` works for circles and squares)

**Q3. What is the difference between an interface and an abstract class?**
An **interface** is a pure contract: "anything claiming to be Printable must have `print()`" — no state, and a class can implement many. An **abstract class** is a partial implementation: shared code + some methods left for children to fill in — a class extends only one. Rule of thumb: interface for *capability*, abstract class for *shared skeleton*.

**Q4. What is composition over inheritance?**
Inheritance = "is-a"; composition = "has-a" (a Car *has* an Engine). Deep inheritance trees become rigid — one change at the top ripples everywhere, and children inherit things they don't want. Composition assembles small parts you can swap freely. Guideline: **prefer composition**; reserve inheritance for true, stable is-a relationships.

**Q5. What is coupling and cohesion?**
- **Coupling** — how much modules depend on each other. Low is good: change one part without breaking others.
- **Cohesion** — how focused a single module is on one job. High is good.
The goal of almost every design principle is the same sentence: **low coupling, high cohesion**.

**Q6. What is polymorphism — compile-time vs runtime?**
**Compile-time (static)**: method overloading — same name, different parameters, resolved when compiling. **Runtime (dynamic)**: method overriding — a subclass provides its own version, and the actual object's method runs even when accessed through a parent-type reference. Runtime polymorphism is what makes plug-in style designs possible.

**Q7. What is the difference between pass-by-value and pass-by-reference?**
Pass-by-value copies the data — the callee can't change the caller's variable. Pass-by-reference passes the variable itself. Nuance most languages have: object *references* are passed by value — so you can mutate the object's contents, but reassigning the parameter doesn't affect the caller. This distinction explains a huge class of "why did my data change?" bugs.

**Q8. What is immutability and why is it valuable?**
An immutable object never changes after creation — updates produce new copies. Benefits: safe to share across threads (no locks needed), safe to cache, trivially comparable, and no "spooky action at a distance" where distant code mutates your data. Cost: more allocations — usually worth it.

**Q9. What is a pure function?**
Same inputs → same output, and **no side effects** (no mutating globals, no I/O). Pure functions are trivially testable, cacheable, parallelizable, and easy to reason about. A good architecture pushes logic into a pure core and keeps side effects (DB, network) at the edges.

**Q10. What is the difference between static and dynamic typing?**
Static: types checked before running (Java, C#, Go, TypeScript) — errors caught early, better tooling/refactoring, some ceremony. Dynamic: types checked at runtime (Python, Ruby, JS) — faster to start, errors surface later. Senior take: on large, long-lived, multi-team codebases, static typing pays for itself — it's machine-checked documentation.

**Q11. What is memory management — stack vs heap, and garbage collection?**
The **stack** holds function calls and local values — automatic, fast, freed on return. The **heap** holds objects with longer lifetimes — must be freed manually (C/C++) or by a **garbage collector**, which finds objects nothing references anymore and reclaims them. GC removes whole bug classes (use-after-free, most leaks) at the cost of occasional pauses.

**Q12. What causes a memory leak in a garbage-collected language?**
GC frees only *unreachable* objects — so leaks come from things still referenced but no longer needed: ever-growing caches/maps, listeners registered but never removed, static/global collections, closures capturing large objects. Detection: memory grows and never drops; diff heap snapshots over time to find what's accumulating.

**Q13. What is exception handling done well?**
Catch exceptions only where you can **do something meaningful** (retry, translate, respond) — otherwise let them propagate to a central handler. Never swallow silently. Use specific exception types, add context when re-throwing, clean up resources with finally/using/defer. Design distinction: expected outcomes (validation failures) are results; exceptions are for the truly exceptional.

**Q14. What is recursion, and what are its risks?**
A function solving a problem by calling itself on a smaller piece, with a **base case** to stop. Natural for trees, graphs, divide-and-conquer. Risks: stack overflow on deep input and repeated re-computation (naive Fibonacci is exponential — fixed by memoization). Any recursion can be rewritten iteratively with an explicit stack when depth is a concern.

**Q15. What is the difference between a compiler and an interpreter (and JIT)?**
A compiler translates the whole program to machine code before running; an interpreter executes source line-by-line. Modern runtimes (JVM, .NET, V8) blend both: compile to bytecode, then a **JIT** (just-in-time) compiler turns hot paths into optimized machine code at runtime — interpreter-like flexibility with near-compiled speed.

---

## Part 2: Design Principles & Patterns (Q16–Q27)

**Q16. What are the SOLID principles?**
- **S**ingle responsibility — a module has one reason to change
- **O**pen/closed — open to extension, closed to modification (add new behavior via new code, not edits)
- **L**iskov substitution — a subtype must work anywhere its parent does, no surprises
- **I**nterface segregation — several small interfaces beat one fat one clients half-use
- **D**ependency inversion — high-level policy depends on abstractions, not concrete details
Net effect: code you can change without fear.

**Q17. Explain the Single Responsibility Principle with an example.**
A `ReportService` that computes the report, formats it as PDF, *and* emails it has three reasons to change (business rules, formatting, delivery). Split it into `ReportCalculator`, `PdfFormatter`, `EmailSender` — each changes independently and tests in isolation. "Responsibility" = a source of change, not "does one function."

**Q18. What is the Liskov Substitution Principle in practice?**
If code works with a `Bird`, it must work with every subclass — so `Penguin extends Bird` with a `fly()` that throws breaks LSP. The fix is rethinking the hierarchy (`FlyingBird` vs `Bird`). Symptom of violation: code doing `if (x instanceof Penguin)` checks — the abstraction has leaked.

**Q19. What are the DRY, KISS, and YAGNI principles?**
- **DRY** — Don't Repeat Yourself: one authoritative place per piece of *knowledge*. Caveat: duplicated code that represents *different* concepts shouldn't be merged — wrong abstraction is worse than duplication.
- **KISS** — Keep It Simple: the simplest design that works wins.
- **YAGNI** — You Aren't Gonna Need It: don't build for imagined future requirements; they usually arrive different or never.

**Q20. What is the Singleton pattern and why is it controversial?**
Ensures one instance globally (config, connection pool). Controversial because it's global state in disguise: hidden dependencies (nothing in a function's signature reveals it uses the singleton), hard to substitute in tests, and tricky under concurrency. Modern preference: create one instance at startup and **inject** it where needed.

**Q21. What are the Factory and Builder patterns?**
**Factory** — centralizes *which* class to create: `PaymentFactory.create("stripe")` returns the right processor; callers depend only on the interface. **Builder** — constructs complex objects step-by-step (`new RequestBuilder().url(...).timeout(...).build()`), replacing constructors with eight confusing parameters. Both exist to tame object creation.

**Q22. What is the Observer (pub/sub) pattern?**
Subjects notify subscribed observers when something happens — the subject doesn't know or care who's listening. It's the backbone of UI events, event emitters, and message-driven systems. Benefit: decoupling. Watch out for: forgotten subscriptions (leaks) and hard-to-trace chains of reactions.

**Q23. What is the Strategy pattern?**
Encapsulate interchangeable algorithms behind one interface and pick at runtime: shipping cost by `FlatRate`, `ByWeight`, or `Express` — the checkout code doesn't change when a new strategy is added. It's the classic cure for sprawling `if/else` chains keyed on type, and the Open/Closed Principle in action.

**Q24. What are the Adapter, Decorator, and Facade patterns?**
- **Adapter** — converts one interface to another so incompatible things work together (a plug adapter; wrapping a vendor SDK to match your interface)
- **Decorator** — wraps an object to add behavior without changing it (add caching or logging around a repository); middleware chains are decorators
- **Facade** — one simple front door hiding a messy subsystem (`Checkout.process()` orchestrating inventory, payment, email)

**Q25. What is the Repository pattern?**
An abstraction over data access: business code says `userRepo.findByEmail(x)` and has no idea whether Postgres, Mongo, or an in-memory fake answers. Benefits: swap storage, test business logic without a database, keep SQL/ORM details in one layer. Common in layered and clean architectures.

**Q26. What is inversion of control (IoC)?**
The general principle behind DI: instead of your code controlling construction and flow, a framework/container does, calling *your* code at the right moments ("don't call us, we'll call you" — the Hollywood principle). DI containers, web frameworks invoking your handlers, and event loops are all IoC.

**Q27. What is an anti-pattern? Give examples.**
A common "solution" that reliably makes things worse:
- **God object** — one class that knows/does everything
- **Spaghetti code** — tangled control flow with no structure
- **Golden hammer** — one favorite tool applied to every problem
- **Premature optimization** — complexity added before measuring
- **Big ball of mud** — a system with no discernible architecture
Recognizing them is as valuable as knowing patterns.

---

## Part 3: Data Structures & Algorithms (Q28–Q39)

**Q28. What is Big-O notation, in plain terms?**
How an algorithm's cost grows as input grows, ignoring constants: O(1) constant, O(log n) halving (binary search), O(n) linear, O(n log n) good sorts, O(n²) nested loops, O(2ⁿ) brute force. It answers "will this still work with 10 million items?" — the difference between milliseconds and hours.

**Q29. Array vs linked list — what's the trade-off?**
Array: contiguous memory → O(1) access by index, cache-friendly, but O(n) insert/delete in the middle (shifting). Linked list: nodes with pointers → O(1) insert/delete *at a known node*, but O(n) to find anything and cache-unfriendly. In practice arrays/dynamic arrays win most of the time; know *why* you'd deviate.

**Q30. How does a hash table work, and what about collisions?**
A hash function maps a key to a bucket index → average O(1) get/put. Two keys can land in the same bucket (**collision**), handled by chaining (a small list per bucket) or open addressing (probe for the next slot). When the table gets full, it resizes and rehashes. Worst case degrades to O(n) — why hash quality and load factor matter.

**Q31. Stack vs queue — and where does each show up in real systems?**
Stack = LIFO (undo history, call stack, expression parsing, DFS). Queue = FIFO (task/job queues, message brokers, BFS, request buffering). Variants worth naming: deque, priority queue (heap) — used for schedulers and "top-K" problems.

**Q32. What is a binary search tree, and why do balanced trees matter?**
A tree where left < node < right, giving O(log n) search/insert/delete — *if balanced*. Insert sorted data into a naive BST and it degrades into a linked list: O(n). Self-balancing trees (AVL, red-black) rotate to stay ~log n; they power ordered maps and database indexes (B-trees are the disk-friendly cousin).

**Q33. What is a heap and what is it used for?**
A tree stored in an array where every parent beats its children (min-heap: parent smallest). Peek best = O(1); insert/remove = O(log n). Uses: priority queues, schedulers, Dijkstra, "top K of a stream" without sorting everything.

**Q34. What is a graph, and BFS vs DFS?**
Nodes + edges: social networks, road maps, service dependencies. **BFS** explores level by level (queue) → shortest path in unweighted graphs. **DFS** dives deep first (stack/recursion) → cycle detection, topological sort, connectivity. Topological sort of a dependency graph is how build tools and package managers order work.

**Q35. Explain binary search and its key requirement.**
On **sorted** data, compare the middle, discard half, repeat → O(log n): a million items in ~20 steps. The requirement people forget: the data must be sorted (or monotonic). The pattern generalizes — "binary search the answer" solves many optimization problems.

**Q36. Compare common sorting algorithms.**
- **Quicksort** — O(n log n) average, in-place, O(n²) worst on bad pivots; usual default
- **Merge sort** — O(n log n) guaranteed, stable, needs extra memory; great for linked lists/external sorting
- **Heap sort** — O(n log n) guaranteed, in-place, not stable
- **Insertion sort** — O(n²) but fastest for tiny/nearly-sorted data (real libraries switch to it for small runs)
Knowing "stable" (equal items keep order) is a senior detail — it matters when sorting by multiple fields.

**Q37. What is memoization / dynamic programming?**
When a problem's subproblems repeat, cache their answers. Naive Fibonacci recomputes the same values exponentially; memoized it's O(n). Dynamic programming is the systematic version: solve overlapping subproblems once, build up (tabulation) or cache down (memoization). Real-world flavor: any expensive deterministic function is a caching candidate.

**Q38. What is a Bloom filter (and where would you use one)?**
A tiny probabilistic structure answering "have I seen this?" with **no false negatives** but possible false positives. Use: cheap pre-check before an expensive lookup — "is this URL malicious?", "is this key definitely not in the database?" — saving a disk/network hit most of the time.

**Q39. How do you approach an unfamiliar algorithm problem in an interview?**
A repeatable script: restate the problem + clarify constraints (sizes, duplicates, sorted?) → work a small example by hand → state the brute force and its complexity → look for structure (sorted? use two pointers/binary search; counting? hash map; overlapping subproblems? DP) → code cleanly → test edge cases (empty, one item, extremes). Narrate throughout — interviewers grade the *process*.

---

## Part 4: Databases (Q40–Q51)

**Q40. SQL vs NoSQL — how do you actually choose?**
Relational (Postgres/MySQL): structured data, relationships/joins, ACID transactions, ad-hoc queries — the correct **default** for business data. NoSQL when a specific shape fits: documents (Mongo) for flexible nested data, key-value (Redis) for speed/caching, wide-column (Cassandra) for huge write throughput, graph (Neo4j) for relationship-heavy queries. Decide by data shape, consistency needs, and query patterns — not hype.

**Q41. What are ACID properties?**
- **Atomicity** — all steps commit or none (debit AND credit, never one)
- **Consistency** — constraints always hold before and after
- **Isolation** — concurrent transactions don't see each other's half-finished work
- **Durability** — once committed, survives crashes
This is why money and inventory live in transactional databases.

**Q42. What is a database index, how does it work, and what does it cost?**
Typically a **B-tree**: a sorted, shallow structure turning full scans into a few hops — a book's index vs reading every page. Costs: storage + every write must update each index. Index what appears in WHERE/JOIN/ORDER BY; verify with the query planner (`EXPLAIN`); a composite index on (a, b) helps queries on `a` or `a AND b`, not `b` alone.

**Q43. What is normalization, and when do you denormalize?**
Normalization: each fact stored once, linked by keys — no duplication, no "updated it here but not there" anomalies (1NF/2NF/3NF formalize this). Denormalization: deliberately duplicate (store `author_name` on the post) to skip joins and read faster — accepting the sync burden. Transactional systems normalize; read-heavy views, caches, and analytics denormalize.

**Q44. What is the N+1 query problem?**
Fetch 100 orders (1 query), then loop fetching each order's customer (100 more) — 101 round trips instead of 2. The classic ORM trap; invisible in dev, brutal in production. Fix: joins, batched `IN` queries, or the ORM's eager loading.

**Q45. What are transaction isolation levels?**
The dial between correctness and concurrency:
- **Read uncommitted** — can see others' uncommitted data (dirty reads)
- **Read committed** — only committed data (common default)
- **Repeatable read** — same rows re-read identically within a transaction
- **Serializable** — as if transactions ran one at a time; safest, slowest
Senior point: know your DB's default and that anomalies like lost updates can occur below serializable.

**Q46. Optimistic vs pessimistic locking?**
**Pessimistic**: lock the row when reading (`SELECT ... FOR UPDATE`); others wait — safe under heavy contention, hurts concurrency. **Optimistic**: no lock; read a version number, and the update succeeds only if the version hasn't changed — retry on conflict. Optimistic wins when conflicts are rare (most web apps).

**Q47. What is replication, and what problem does replication lag cause?**
Copies of the database: a primary takes writes, replicas serve reads (read scaling + failover). Replication is usually asynchronous → **lag**: write to primary, immediately read a replica, and your own write isn't there yet ("read-your-writes" anomaly). Mitigations: read the primary after writes, sticky sessions, or synchronous replication at a latency cost.

**Q48. What is sharding (partitioning)?**
Splitting one dataset across machines by a **shard key** (e.g., user ID hash) when a single node can't hold the load. Costs are real: cross-shard queries and transactions get hard, hot keys create hot shards, resharding is painful. Senior take: exhaust indexing, caching, read replicas, and bigger hardware first.

**Q49. What is the CAP theorem in plain terms?**
When the network partitions (nodes can't reach each other), a distributed store must choose: **Consistency** — refuse to answer rather than risk stale data — or **Availability** — answer, possibly stale. Banking leans CP; social feeds lean AP with **eventual consistency** (replicas converge shortly after).

**Q50. How do you run schema migrations safely with zero downtime?**
Versioned migration files in git, applied automatically on deploy — and every change **backwards-compatible, in phases**: add new nullable column → deploy code writing both old and new → backfill → deploy code reading new → drop old. Never rename or drop in one step while old code is still running.

**Q51. Why is offset pagination problematic, and what's the alternative?**
`LIMIT 20 OFFSET 100000` scans and discards 100k rows — slower the deeper you page, and rows shift if data changes mid-scroll. **Keyset (cursor) pagination** — `WHERE id > last_seen ORDER BY id LIMIT 20` — is constant-time and stable. Use offset only for small, shallow lists.

---

## Part 5: Networking, Web & APIs (Q52–Q61)

**Q52. What happens when you type a URL and hit enter?**
The classic full-stack question: DNS resolves the domain to an IP (cache → resolver → authoritative servers) → TCP handshake → TLS handshake (certificates, key exchange) → HTTP request → server processes and responds → browser parses HTML, fetches assets, builds the render tree, paints. A senior answer names the layers and where caching happens at each (DNS cache, CDN, browser cache).

**Q53. TCP vs UDP?**
**TCP**: connection-based, ordered, reliable (lost packets re-sent) — web pages, APIs, anything that must arrive intact. **UDP**: fire-and-forget, no ordering or delivery guarantee, minimal overhead — live video, gaming, DNS, where "late" is worse than "lost." HTTP/3 runs on QUIC over UDP, re-implementing reliability smarter.

**Q54. What is HTTP — key methods and status codes?**
A stateless request/response protocol. Methods: GET (read, safe), POST (create/act), PUT (replace), PATCH (partial update), DELETE. Status families: 2xx success (200 OK, 201 Created, 204 No Content), 3xx redirects (301, 304 Not Modified), 4xx client errors (400, 401 unauthenticated, 403 unauthorized, 404, 409 conflict, 429 rate-limited), 5xx server errors (500, 503). Using them *correctly* is an API design signal.

**Q55. What makes an API RESTful, and what makes it a *good* API?**
REST: resources as nouns (`/users/42`), verbs via HTTP methods, statelessness (each request self-contained), proper status codes. A *good* API adds: consistent naming and error shapes, versioning strategy, pagination on collections, idempotency where needed, backwards compatibility discipline (add fields, never repurpose them), and clear docs. The API is a contract — breaking it breaks other people's code.

**Q56. What is idempotency and why does it matter?**
An operation that yields the same result no matter how many times it's repeated: `PUT /users/42`, `DELETE /users/42` — yes; `POST /orders` — no. It matters because **networks fail and clients retry**: payment systems accept an idempotency key so a retried request can't charge twice. Designing retryable systems starts here.

**Q57. REST vs GraphQL vs gRPC — when each?**
- **REST** — simple, HTTP-cacheable, universal; the default for public/CRUD APIs
- **GraphQL** — client picks exact fields in one request; shines with many varied clients (mobile/web); costs: caching complexity, resolver N+1s, server setup
- **gRPC** — binary, contract-first (protobuf), fast, streaming; ideal for internal service-to-service calls; weak for browsers
Choose per audience, not fashion.

**Q58. What is DNS and how does caching work in it?**
The internet's phone book: name → IP. Resolution walks resolver → root → TLD → authoritative servers, with answers cached at every layer for their **TTL**. Practical consequences: DNS changes propagate gradually (lower TTL before a migration), and DNS is itself a load-balancing/failover tool (round-robin, geo-DNS).

**Q59. How does HTTPS/TLS work at a high level?**
The client and server handshake: server presents a **certificate** (its public key, vouched for by a certificate authority the browser trusts) → they agree on keys using asymmetric crypto → then switch to fast **symmetric** encryption for the actual data. Guarantees: confidentiality (no eavesdropping), integrity (no tampering), authenticity (you're talking to the real site).

**Q60. What is a CDN and what problems does it solve?**
A network of edge servers holding copies of your static (and sometimes dynamic) content close to users. Solves: latency (physics — shorter distance), origin load (most requests never reach your servers), and traffic spikes/DDoS absorption. Key concepts: cache keys, TTLs, and purging/invalidation on deploy.

**Q61. WebSockets vs polling vs server-sent events?**
- **Polling** — client asks repeatedly; simple, wasteful, laggy
- **Long polling** — server holds the request until there's news; better, still request-per-message
- **SSE** — one HTTP stream, server → client only; great for feeds/notifications
- **WebSockets** — full duplex persistent connection; chat, games, collaborative editing
Trade-off: persistent connections need stateful infrastructure (sticky routing or a pub/sub backplane) to scale.

---

## Part 6: Operating Systems & Concurrency (Q62–Q69)

**Q62. What is the difference between a process and a thread?**
A **process** is a running program with its own isolated memory. A **thread** is an execution path *inside* a process — threads share the process's memory. Sharing makes threads cheap and communication easy, and also dangerous: two threads touching the same data concurrently is where race conditions live. Processes are safer but talk via slower IPC.

**Q63. What is a race condition?**
When correctness depends on the timing of concurrent operations. Classic: two threads read balance=100, both add 50, both write 150 — one update lost. The window is any read-modify-write that isn't atomic. Fixes: locks, atomic operations, or designing shared state away (immutability, message passing, single-writer).

**Q64. What is a deadlock, and how do you prevent it?**
Two+ threads each holding a lock the other needs — everyone waits forever (thread A holds lock 1 wants lock 2; B holds 2 wants 1). The four conditions must all hold (mutual exclusion, hold-and-wait, no preemption, circular wait); break any one. Most practical fix: **always acquire locks in a consistent global order**, plus lock timeouts as a safety net.

**Q65. What is a mutex vs a semaphore?**
**Mutex** — a lock: one holder at a time, and the holder must release it (protecting a critical section). **Semaphore** — a counter of permits: N holders allowed (limit "at most 10 concurrent DB connections"); any thread can signal. Mutex = one bathroom key; semaphore = N parking spots.

**Q66. Concurrency vs parallelism?**
**Concurrency** — *dealing with* many things at once: interleaving tasks, possibly on one core (a barista juggling orders). **Parallelism** — *doing* many things at the same instant on multiple cores (many baristas). Concurrency is a program structure; parallelism is an execution property. Async I/O gives concurrency without parallelism; data crunching on 8 cores is parallelism.

**Q67. What is blocking vs non-blocking (async) I/O?**
Blocking: the thread stops and waits for the disk/network — simple, but each waiting request holds a thread. Non-blocking: start the operation, do other work, get notified on completion (callbacks/futures/async-await) — one thread can juggle thousands of connections. This is the core idea behind event-loop servers, and why CPU-heavy work must be kept off that loop.

**Q68. What is virtual memory?**
The OS gives each process the illusion of its own large, private, contiguous memory, mapping pages to physical RAM on demand and swapping cold pages to disk. Benefits: process isolation (one crash can't scribble on another's memory), overcommit, memory-mapped files. When RAM runs out and the machine starts swapping heavily ("thrashing"), performance falls off a cliff.

**Q69. What is a thread pool, and why use one instead of spawning threads freely?**
A fixed set of worker threads pulling tasks from a queue. Creating threads is expensive and unbounded threads can exhaust memory/CPU with context switching. A pool caps concurrency, reuses workers, and gives a natural backpressure point (a bounded queue). Sizing intuition: CPU-bound → ~number of cores; I/O-bound → more, since threads spend time waiting.

---

## Part 7: System Design & Architecture (Q70–Q82)

**Q70. How do you approach a system design interview question?**
A repeatable script: (1) clarify requirements — functional + scale (users, QPS, read/write ratio, data size); (2) define the API; (3) high-level diagram: client → load balancer → stateless app servers → cache → database; (4) data model and key generation; (5) deep-dive the interesting component; (6) discuss bottlenecks, failure modes, and monitoring. Explicit trade-off talk ("I'd choose X over Y because...") is what's being graded.

**Q71. What is horizontal vs vertical scaling?**
Vertical: a bigger machine — easy, no code changes, but has a ceiling and is a single point of failure. Horizontal: more machines behind a load balancer — near-unlimited and fault-tolerant, but requires **stateless** servers (session data in Redis, files in object storage) so any instance can serve any request. Modern default: design for horizontal from day one.

**Q72. What does a load balancer do?**
Distributes traffic across instances (round-robin, least-connections), health-checks them and routes around failures, often terminates TLS. L4 balances on IP/port (fast, dumb); L7 understands HTTP (route by path/header, sticky sessions). It's both a scaling tool and the first line of high availability.

**Q73. What is caching, where do you cache, and what are the classic pitfalls?**
Layers, closest-first: browser → CDN → application cache (Redis/Memcached) → database's own caches. Common pattern is **cache-aside**: check cache → miss → load from DB → store with TTL. Pitfalls: **invalidation** (stale data — famously one of the two hard problems), **stampedes** (a hot key expires and thousands of requests hit the DB at once — use locks/jittered TTLs), and accidentally caching per-user data publicly.

**Q74. Monolith vs microservices — the senior answer?**
Monolith: one deployable — simpler to build, test, debug, and it scales further than people admit. Microservices buy independent deploys and scaling per team — at the price of distributed-systems pain: network failures, tracing, data consistency, operational overhead. Senior answer: **start with a well-modularized monolith**; extract services when team size, deploy contention, or divergent scaling genuinely demand it. Microservices solve an *organizational* problem more than a technical one.

**Q75. What is a message queue, and what problems does it solve?**
A buffer between producer and consumer (RabbitMQ, SQS, Kafka). Solves: making slow work async (respond to signup instantly, queue the welcome email), absorbing spikes, decoupling services, and safe retries. Must-know concepts: at-least-once delivery (so consumers must be **idempotent**), ordering guarantees, and dead-letter queues for poison messages.

**Q76. What is event-driven architecture?**
Services communicate by publishing events ("OrderPlaced") that others subscribe to, instead of calling each other directly. Wins: loose coupling (add a new consumer without touching the producer), natural audit log, resilience via buffering. Costs: eventual consistency, harder end-to-end reasoning ("what happens when an order is placed?" spans many services), and duplicate delivery to design for.

**Q77. How do you design for high availability?**
Eliminate single points of failure: redundant instances across zones, load balancing with health checks, database replication with automatic failover, and stateless services so instances are replaceable. Quantify with "nines" (99.9% ≈ 8.7h down/year; 99.99% ≈ 52min). Every added nine costs disproportionately more — availability targets are a business decision.

**Q78. What patterns make systems resilient to partial failure?**
Assume every network call can fail or hang: **timeouts** on everything; **retries with exponential backoff + jitter** (only for idempotent operations); **circuit breakers** — stop calling a failing service, fail fast, probe for recovery; **bulkheads** — isolate resource pools so one failing dependency can't drown the whole app; **graceful degradation** — serve cached/partial data instead of an error page.

**Q79. What is eventual consistency, and where is it acceptable?**
After a write, replicas converge to the same value *eventually* — a read in the gap may be stale. Acceptable where a brief inconsistency is harmless: like counts, product views, feeds. Not acceptable for balances, inventory at checkout, permissions. Design question to always ask: "what does the user experience if this read is 2 seconds stale?"

**Q80. What is a distributed transaction, and what is the Saga pattern?**
A business action spanning multiple services/databases (order + payment + inventory) can't use one ACID transaction. **Saga**: a sequence of local transactions, each publishing an event triggering the next — and if a step fails, run **compensating actions** to undo previous ones (refund the payment, restock). Trade: you exchange atomicity for availability and must design the "undo" path explicitly.

**Q81. What is observability (logs, metrics, traces)?**
Answering "what is happening in production?":
- **Logs** — structured (JSON) event records, correlated by request ID
- **Metrics** — numbers over time: latency p95/p99, error rate, throughput, saturation — feeding dashboards and alerts
- **Traces** — one request's path across services showing where the time went
Senior mindset: instrument before the incident; alert on symptoms users feel (error rate, latency), not just CPU.

**Q82. What are rate limiting and throttling, and common algorithms?**
Protecting a service from overload/abuse by capping requests per client. Algorithms: **token bucket** (tokens refill at rate R, each request spends one — allows short bursts), leaky bucket (smooths to a constant rate), fixed/sliding windows (counters per interval). Return 429 with a Retry-After header. Applies at the edge (per-IP), per-API-key, and between internal services.

---

## Part 8: Testing & Quality (Q83–Q90)

**Q83. What is the testing pyramid?**
Many fast **unit tests** (single functions, dependencies faked) → fewer **integration tests** (real components together — e.g., service + real database in a container) → few **end-to-end tests** (whole system through the UI/API). E2E tests are the most realistic but slowest and flakiest, so keep the pyramid bottom-heavy for a fast, trustworthy suite.

**Q84. What are mocks, stubs, and fakes?**
Test doubles replacing real dependencies. **Stub** — returns canned answers ("the user exists"). **Mock** — also verifies interactions ("email service was called exactly once with X"). **Fake** — a working lightweight implementation (in-memory repository). Guideline: don't over-mock — tests coupled to internal call patterns break on every refactor while proving little.

**Q85. What makes a *good* unit test?**
Fast, isolated, deterministic, and asserting **behavior, not implementation**: given inputs → expected outcome, one logical assertion per test, descriptive name that reads as a spec ("rejects expired coupon"). A good suite lets you refactor internals freely — if renaming a private method breaks 40 tests, the tests are testing the wrong thing.

**Q86. What is TDD, and what's the honest senior take?**
Red → green → refactor: write a failing test, make it pass minimally, clean up. Real benefits: testable design by construction, a safety net, executable documentation. Honest take: strict TDD shines for pure logic and bug fixes (reproduce the bug in a test *first*); it's clumsy for exploratory or UI-heavy work. The non-negotiable outcome is *meaningful tests exist* — the ceremony is optional.

**Q87. What is code coverage, and why is 100% coverage a trap?**
The percentage of code executed by tests. Useful as a *floor detector* — 30% means big blind spots. But coverage measures execution, not verification: a test can run every line and assert nothing. Chasing 100% breeds brittle, low-value tests of trivial code. Aim high on core business logic; judge tests by the bugs they'd catch.

**Q88. What do you look for in a code review?**
In priority order: correctness and edge cases → design (does it fit the architecture, right abstraction level?) → readability (will the next person understand it?) → tests (do they cover the risky paths?) → performance/security where relevant. Style nits go to linters, not humans. Senior conduct: review promptly, ask questions rather than command, and approve when it's *better*, not perfect.

**Q89. What is refactoring, and when should you do it?**
Improving code structure **without changing behavior** — rename, extract, simplify — with tests as the safety net. When: continuously in small doses (boy-scout rule: leave code slightly better), before adding a feature to hostile code ("make the change easy, then make the easy change"), and never as a big-bang rewrite mixed with behavior changes.

**Q90. How do you debug a hard problem systematically?**
Reproduce it reliably first — a bug you can trigger is half-solved. Then: read the actual error and logs, form a hypothesis, and **bisect** the search space (git bisect across commits, binary-search the data or code path, isolate layers). Change one variable at a time; keep notes. When truly stuck: explain it out loud (rubber-duck), check recent changes, and question your assumptions — the bug is usually in the code you were sure was fine.

---

## Part 9: Security (Q91–Q95)

**Q91. What are the OWASP-style top vulnerabilities every engineer must know?**
- **Injection** (SQL/command) — untrusted input becomes code
- **Broken authentication/session handling**
- **XSS** — attacker's script runs in other users' browsers
- **Broken access control** — users reaching data/actions they shouldn't (e.g., changing an ID in the URL)
- **Security misconfiguration** — default creds, open buckets, verbose errors
- **Vulnerable dependencies** — known-CVE libraries in your build
The theme: never trust input, and verify authorization on every request server-side.

**Q92. How do you prevent SQL injection and XSS?**
**SQL injection**: parameterized queries / prepared statements — never concatenate input into SQL; the input is data, never code. **XSS**: escape output by default (modern frameworks do), sanitize any user HTML you must render, set a Content-Security-Policy, use HttpOnly cookies so scripts can't steal sessions. Both share one root cause — mixing untrusted data into an interpreted context — and one cure: strict separation.

**Q93. Authentication vs authorization, and how are sessions vs tokens handled?**
**Authentication** = who are you (login). **Authorization** = what may you do (permissions) — must be checked server-side on *every* request. **Sessions**: server stores state, client holds a session ID cookie — easy revocation, needs shared session store to scale. **Tokens (JWT)**: server signs claims, verifies statelessly — scales easily, but revocation is hard (use short-lived access tokens + refresh tokens). Store tokens in HttpOnly cookies over localStorage when possible.

**Q94. How should passwords be stored?**
Never encrypted (reversible) and never fast-hashed (MD5/SHA-256 — GPUs try billions/sec). Use a **slow, salted, adaptive** hashing algorithm: bcrypt, scrypt, or argon2. The salt (unique per user) defeats rainbow tables; the tunable slowness defeats brute force. Add rate limiting and support MFA. If you can show a user their old password, something is very wrong.

**Q95. What is the principle of least privilege and defense in depth?**
**Least privilege**: every user, service, and API key gets the minimum access needed — the intern can't drop the prod database, the reporting service can only read. **Defense in depth**: multiple overlapping layers (network rules + auth + input validation + encryption + audit logs) so one failed control isn't game over. Together they turn a breach from a catastrophe into a contained incident.

---

## Part 10: DevOps, Delivery & Engineering Practice (Q96–Q100)

**Q96. What is CI/CD?**
**Continuous Integration**: every push automatically runs lint + tests + build — breakage surfaces in minutes, on the machine that matters. **Continuous Delivery/Deployment**: every passing build is automatically shippable (or shipped) to production. The payoff is risk reduction: many small reversible releases instead of rare terrifying big-bang deploys.

**Q97. What are containers, and how do they differ from virtual machines?**
A container (Docker) packages an app with its dependencies and runs it isolated on a **shared OS kernel** — starts in milliseconds, tiny footprint, "works on my machine" solved because the image *is* the environment. A VM virtualizes entire hardware with its own OS — stronger isolation, heavier. Orchestration (Kubernetes) then schedules, scales, heals, and rolls out containers across a fleet.

**Q98. What are blue-green and canary deployments, and feature flags?**
**Blue-green**: two identical environments; deploy to the idle one, flip traffic, instant rollback by flipping back. **Canary**: send ~5% of traffic to the new version, watch error/latency metrics, ramp up or abort. **Feature flags**: ship code dark and toggle features per user/percentage — decoupling *deploy* from *release*, enabling A/B tests and instant kill-switches. Together they make releases boring — which is the goal.

**Q99. What is technical debt, and how do you manage it as a senior engineer?**
Shortcuts that speed you up now and tax every future change — sometimes a rational trade (ship the MVP), toxic when invisible and unbounded. Manage it: make it **visible** (tickets, an explicit register), budget a steady slice of each cycle for paydown, refactor opportunistically in touched code, and translate it into business language ("this module costs us ~2 days per sprint in bugs") so prioritization is a shared decision, not an engineering complaint.

**Q100. What does "senior" actually mean beyond coding skill?**
Multiplying the team, not just yourself: making trade-offs explicit (build vs buy, fast vs right) and tying them to business impact; designing for the next engineer (clarity over cleverness); de-risking work (small increments, monitoring, rollback plans); mentoring through reviews and pairing; and communicating with non-engineers without jargon. In the interview itself, this shows up as answers structured "definition → why it matters → trade-offs → when I'd choose differently."

---

*Interview tip: for any question here, a senior-sounding answer follows the shape — plain definition → why it matters → a concrete example or trade-off → when you'd deviate. The last two are what separate seniors from people who memorized definitions.*
