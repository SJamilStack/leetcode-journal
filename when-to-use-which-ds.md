# When to Use What: Picking the Right Data Structure

*A quick reference in plain English. One rule, a cheat table, and short notes for each structure.*

---

## The one rule that decides everything

> **Don't pick the container by what it holds. Pick it by the question you will ask it, over and over, inside your loop.**

Every structure is fast at one kind of question and slow at others. Find your loop's repeated question first. The structure picks itself.

---

## The cheat table

| The question your loop keeps asking | Use this | Why |
|---|---|---|
| "Is X inside?" | **set** | One hash lookup, O(1). A list would walk every item, O(n). |
| "How many of X are inside?" | **dict / Counter** | O(1) lookup *and* it counts. A set only knows yes/no. |
| "What maps to X?" (name → value) | **dict** | That is literally what a dict is. |
| "What is at position i?" / "keep things in order" | **list** | Position and order are what lists are for. |
| "What entered first?" (process in arrival order) | **deque (queue)** | `popleft()` is O(1). Popping a list's front is O(n) — everything shifts. |
| "What entered last?" (undo, matching, nesting) | **list as stack** | `append` / `pop` from the end are both O(1). |
| "What is the smallest/largest right now?" (again and again) | **heap** | Peek O(1), pop O(log n). Re-sorting a list every turn is O(n log n) each time. |
| "Building a string piece by piece" | **list, then `''.join()` once** | See the string rule below. |

---

## The string rule (the 1512 lesson)

**Never build a string with `+` or `+=` inside a loop.**

Python strings are immutable — frozen. Every `+` throws the old string away, makes a brand-new one, and copies everything so far. Copy 1 char, then 2, then 3... that triangle adds up to O(n²) for a loop that *looks* O(n).

The fix, always the same three steps:

```python
pieces = []            # 1. collect in a list
for ...:
    pieces.append(ch)  # 2. append is O(1)
return ''.join(pieces) # 3. glue once at the end — O(n) total
```

Small inputs hide this bug — it still passes. The habit is what doesn't scale.

---

## Set vs list (the LC 3 lesson)

The trap: a list feels like the default container, so the hand reaches for it.

The test: what question will you ask it?

- "Is X inside?" asked in a loop → **set**. List does it in O(n) per ask; across n asks, that's O(n²).
- Never asking "is X inside?", only walking items in order or by position → **list** is right.

A list here is usually not *wrong* — it passes, slowly. It answers a question nobody asked (order) and answers the real question (membership) at a crawl.

Upgrade path: the moment the question becomes "how *many* of X?", the set upgrades to a **Counter/dict**. Same idea, plus counting.

---

## Queue vs stack — one memory trick

Both answer "what do I handle next?" The difference is which end.

- **Queue** = a line at a shop. First in, first out. → BFS, level-order, "process in arrival order." Use `collections.deque`.
- **Stack** = a pile of plates. Last in, first out. → DFS, matching brackets, undo, "most recent unfinished thing." A plain list works.

Red flag: `my_list.pop(0)` in a loop. That's a queue built on the wrong structure — every pop shifts the whole list, O(n) each. Swap in a deque.

---

## Heap — the bouncer

Use when the loop keeps asking "smallest (or largest) *right now*?" while items keep arriving.

Classic tell: "top k", "k closest", "k most frequent", "merge k sorted...". Keep a heap of size k; every newcomer either beats the worst member and swaps in, or is turned away. That's O(n log k) instead of sorting everything.

If you only need the min/max **once**, don't reach for a heap — `min()` / `max()` is one O(n) pass and simpler.

---

## Red flags: your code telling you the structure is wrong

- `x in my_list` inside a loop → wanted a **set** (or dict).
- `s += ch` inside a loop → wanted a **list + join**.
- `my_list.pop(0)` inside a loop → wanted a **deque**.
- `my_list.sort()` inside a loop → probably wanted a **heap**.
- Parallel lists like `names[i]` matching `ages[i]` → wanted a **dict** (or list of tuples).

Each red flag is the same disease: the container answers your loop's question in O(n) when another answers it in O(1) — turning an O(n) algorithm into O(n²) without changing a line of logic.

---

## The 10-second ritual before coding

1. Say the loop's repeated question out loud.
2. Find it in the cheat table.
3. If two structures fit, take the simpler one — upgrade only when a new question appears.

> **The container did that, not the algorithm.** Same code shape, different structure, different complexity.
