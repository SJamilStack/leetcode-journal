# The Interview Playbook: How to Approach Any Pattern Under Pressure

*A companion to the pattern series. The patterns are the weapons; this is how to fight. Plain English, built for the 40 minutes when it counts.*

---

## The universal script (works for every problem)

Six steps, always in this order. The first four happen **before any code**.

### Step 1 — Restate the problem in your own words (1 minute)

Say it back simply: "So I'm given X, and I need to return Y, and the catch is Z — correct?"

This catches misreadings when they cost nothing, and it buys thinking time that looks like communication, because it *is* communication.

### Step 2 — Poke the edges (1 minute)

Ask three questions out loud:

- "Can the input be empty? One element?"
- "Can values be negative? Duplicated? How big is n?"
- "What should I return when there's no answer?"

The answers matter (negatives just killed sliding window on a sum problem — see the table below). And asking them is itself scored: it's what working engineers do before writing code.

### Step 3 — Say the honest brute force (2 minutes)

"The brute force is to try every pair — that's O(n²). Let me see if a pattern beats it."

Never skip this. It proves you understand the problem, it gives you a safety net if the clever idea stalls, and it sets up the next step: the pattern is now *justified* as an improvement, not pulled from a hat. If time collapses later, a working brute force beats a broken clever solution — always.

### Step 4 — Recognize the pattern out loud (2 minutes)

Use the recognition table (below). Then say the *reason*, not just the name:

> "Longest substring with a condition — that's a contiguous chunk with state, so sliding window: grow right while clean, shrink left when dirty."

Naming the pattern **with its story** signals real understanding. Naming it without a reason signals memorization — interviewers can smell the difference.

### Step 5 — Derive with the three-question ritual (the coding)

Same ritual from every article in the series. Speak it while you code:

1. **The promise** — "my invariant is: at the end of every turn, ___."
2. **The question** — "each turn I must ask ___, which needs a ___ (set / counter / nothing)."
3. **The order** — "so the heartbeat is ask → fix → change → measure — read before I write."

Coding while narrating the invariant is the single strongest signal you can send. It also *prevents* bugs, because every line's position is forced by the promise.

### Step 6 — Trace, then state complexity (3 minutes)

Never say "I think it works." Pick a small nasty input — one that hits the dirty case, the duplicate, the boundary — and walk it line by line, out loud. Then:

> "Time is O(n) — nested loops, but the left pointer never backs up, so it's amortized. Space is O(k) for the counter."

Volunteering the amortized argument unprompted is a senior-level move.

---

## The recognition table: keywords → pattern

| You hear... | Think... | Say out loud... |
|---|---|---|
| substring, subarray, "longest/shortest chunk satisfying..." | **Sliding window** | "Contiguous chunk with state — grow right while clean, shrink left when dirty." |
| sorted array + find/compare pairs; palindrome; from both ends | **Two pointers (converging)** | "Two fingers walking inward; each step eliminates possibilities." |
| in-place removal/moving; "stable"; overwrite as you scan | **Two pointers (reader/writer)** | "Reader scans, writer marks where clean data ends." |
| linked list + cycle / middle / k-th from end | **Fast & slow** | "Two runners; if the track loops, the fast one laps the slow one." |
| "sum of range", many queries, "count subarrays with sum k" | **Prefix sums** | "Mile markers: any stretch is end minus start." Negatives allowed? Then it's this, not window. |
| sorted anything; "minimize the maximum"; "smallest x such that..." | **Binary search** | "Paint it NO-NO-YES-YES, hunt the first YES. My check is ___ and it's monotonic because ___." |
| overlapping meetings, ranges, scheduling | **Merge intervals** | "Sort by start, then walk: overlap the last one, or start fresh." |
| shortest path (unweighted), level by level, "minimum steps" | **BFS** | "Ripples in a pond — a queue processes distance 1, then 2." |
| all combinations/permutations, "generate every valid..." | **Backtracking / DFS** | "Explore the maze; dead end → walk back to the last fork." |
| "top k", "k closest", "k most frequent", merge k things | **Heap** | "A bouncer holds the k best; newcomers must beat the worst member." |
| next greater element, spans, "closest warmer day" | **Monotonic stack** | "Keep the pile sorted; whoever a newcomer evicts just found their answer." |
| count ways, min cost, "can it be done" over choices | **Dynamic programming** | "Answer small versions, write them down, build up." |

Two traps the table already encodes:

- **"Subarray sum" splits on signs.** All positive → window. Negatives possible → prefix sums. (Asking about negatives in Step 2 is what saves you here.)
- **Sorted input is a hint, not a verdict.** Sorted + pairs → two pointers. Sorted + "find/threshold" → binary search. Sorted + intervals → merge. Ask what *question* you'll repeat, not what shape the input has.

---

## The follow-up game (what they'll ask next, per pattern)

Interviews are two rounds: your solution, then the twist. Know the standard twists and half your interview is pre-solved.

- **You used `split()` / built-ins** → "now without them." (That's where two pointers lives.)
- **You used a set for cycle detection** → "now with O(1) space." (Fast & slow.)
- **Sliding window solved** → "what if the array had negatives?" (Switch to prefix sums, and say *why* the window broke: grow no longer guarantees bigger.)
- **Binary search on an array** → "what if there's no array?" (Binary search on the answer — find the monotonic check.)
- **Recursion accepted** → "can you do it iteratively?" (Own stack/queue.)
- **Anything solved** → "what's the complexity, and can you prove the nested loop isn't n²?" (Total pointer movement, never loop levels.)

Offering the follow-up before they ask — "this passes, and if you want O(1) space, I'd switch to fast & slow pointers" — is how strong candidates end problems.

---

## When you're stuck (the honest protocol)

Stuck happens to everyone. The score isn't whether — it's how you behave while stuck.

1. **Say the story, not the syntax.** Go back to plain English: "what I want is to walk backwards and grab whole words..." The code hides in the grammar; the panic hides in the syntax.
2. **Trace a tiny example by hand.** Three elements, on the whiteboard. Where your hand does something your code doesn't — that's the bug, and now it has an address.
3. **Downgrade honestly.** "Let me get the O(n²) version working, then optimize." A running brute force plus a described optimization beats a broken clever attempt, every time, at every company.
4. **Ask a real question.** "I'm deciding between window and prefix sums — can values be negative?" Good questions are scored as collaboration, not weakness.
5. **Never go silent.** Silence is the only unrecoverable state. Narrate the dead end: "this direction fails because X, so let me try Y."

---

## The last five minutes (don't skip these)

- **Test before they ask.** Empty input, single element, the all-duplicates case, the answer-at-the-boundary case. Catching your own bug scores *higher* than never writing it.
- **State complexity unprompted**, with the reason: "O(n) time — both pointers move one way only; O(1) space."
- **Name the trade-off you made**: "the counter costs O(k) space to buy the O(1) membership answer." Trade-offs are the language of senior engineers.
- **Offer the extension**: "with more time, I'd handle the streaming version with..." — end the problem forward-looking.

---

## The one-page cram (night before)

The six phrases — each one rebuilds its whole pattern:

1. **Two pointers:** *"Walk backwards, skip spaces, grab whole words."*
2. **Sliding window:** *"Grow right while clean. Shrink left when dirty."*
3. **Fast & slow:** *"If the track loops, the fast runner laps the slow one."*
4. **Prefix sums:** *"Mile markers: any stretch is end marker minus start marker."*
5. **Binary search:** *"Paint it NO-NO-YES-YES, then hunt the first YES."*
6. **Choosing structures:** *"Pick by the question your loop asks, not by what the container holds."*

And the three rules that ordered every line of code in every pattern:

- **Read before you write** — ask the question while the world still shows the old state.
- **Bounds before contents** — check the box exists before looking inside it.
- **Resets restore the promise** — every update exists to make the loop's opening assumption true again.

Say the phrase. Derive the promise. Let the promise place the lines. That's the whole method — in practice, and in the room.
