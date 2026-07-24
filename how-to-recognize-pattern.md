# DSA Patterns: How to Recognize Them and Build the Logic

*A personal reference. Not a list of solutions to memorize — a list of questions to ask, in order, so the solution builds itself. Every pattern below follows the same shape: how to recognize it, the story, how to build the logic without guessing, and the code skeleton.*

---

## The one method behind everything in this file

Before any pattern-specific stuff, here is the master method. Every pattern below is just this method, applied to a different situation.

**Three questions, always in this order, before writing any code:**

1. **What is the promise?** (In fancy terms: the loop invariant.) One sentence describing what must be true every time your loop finishes a turn. Example: "the window has no repeated letters," or "everything before `write` is finished, correct output."

2. **What question do I ask, every single turn, to keep that promise?** This question is what picks your data structure and your comparison. "Is X inside?" wants a set. "How many of X?" wants a counter. "Is this the same room?" wants nothing at all — just a pointer.

3. **What order of lines keeps the promise from ever being broken?** This is where you decide what happens before what. The answer is almost always: **check before you touch, read before you write, ask before you modify.**

If you only remember one thing from this whole file, remember this three-step ritual. Everything else is a worked example of it.

---

## The other master trick: sorting things into "order matters" vs "filler"

This one is specifically for **in-place array modification** problems (Move Zeroes, Remove Element, Remove Duplicates, String Compression, and dozens more like them).

**The question:** *"Which group of elements needs to keep its original relative order, and which group is just filler — all interchangeable, doesn't matter which one lands where?"*

- The group whose order matters → you scan for it and place it one at a time, using a **write pointer**.
- The filler group → you don't place these individually at all. You already know how many are left and what value goes there. You just stamp it in at the end, no thinking required.

This single question is why "collect the non-zero numbers, then fill the rest with zero" isn't some genius leap — it's the mechanical answer to one question, asked on purpose, every time you see this shape of problem.

---

## Two Pointers

### Recognize it
- In-place work on a string/array; "without extra space"; "without built-ins"
- Sorted array + pair with a property (sum, difference) → pointers start at both ends, walk inward
- Palindrome checks; comparing from both ends
- "Remove/move elements in place, keep relative order" → reader/writer flavor
- Any problem where the array has an obvious "front" and "back," or an obvious "keep" group and "filler" group

### The story
> **Two fingers, each responsible for a job. Either they walk toward each other from opposite ends, or one reads while the other writes, always staying at or behind the reader.**

There are two flavors — know which one you're in before coding:

- **Converging:** both pointers start at opposite ends, walk toward each other. Used for symmetric checks (palindrome) or search (two numbers summing to a target in a sorted array).
- **Reader/writer:** both start at the same end, but `read` always moves at least as fast as `write`. Used for in-place filtering, compressing, or de-duplicating.

### Build the logic
1. **The promise:** for converging — "nothing outside `[left, right]` still needs checking." For reader/writer — "everything before `write` is finished, correct output."
2. **The question, every turn:** converging — "does this pair satisfy the condition?" or "should I skip this side?" Reader/writer — "does `chars[read]` belong in the kept group?"
3. **The order that protects the promise:**
   - If a pointer is standing on "junk" (doesn't belong), advance it and `continue` — don't let code below assume it's valid.
   - `write` only ever moves via `write += 1`, exactly once per thing actually written. Never derive it from another pointer.
   - When you need to remember a value across iterations (a group's character, a running check), snapshot it into its own variable — never compare against a box you're actively overwriting.
   - Bounds check always comes before contents check: `while j >= 0 and s[j] != ' '`, not the other way around — Python's `and` short-circuits left to right.

### Code skeleton (reader/writer)
```python
write = 0
for read in range(len(arr)):
    if should_keep(arr[read]):
        arr[write] = arr[read]
        write += 1
# if there's a "filler" group with a known value, backfill it here
for i in range(write, len(arr)):
    arr[i] = filler_value
return write
```

### Code skeleton (converging)
```python
left, right = 0, len(arr) - 1
while left < right:
    if not ready(arr[left]):
        left += 1
    elif not ready(arr[right]):
        right -= 1
    else:
        # both ready — act (swap, compare, etc.)
        left += 1
        right -= 1
```

### The phrase
**"Walk backwards, skip spaces, grab whole words."** (Or for reader/writer: **"Reader scans, writer marks where clean data ends."**)

---

## Sliding Window

### Recognize it
- "Longest / shortest substring or subarray satisfying a condition"
- "At most k distinct...", "without repeating...", "max sum of size k"
- A contiguous chunk whose *contents* matter — you'll track state about what's inside
- **Critical check:** for sum conditions, only works if all numbers are positive. Negatives present → this is the wrong pattern, use prefix sums instead.

### The story
> **A stretchy rubber band lies over the array. Grow the right end while the band is clean. When it gets dirty, shrink the left end until it is clean again.**

### Build the logic
1. **The promise:** "at the end of every turn, the window satisfies the condition."
2. **The question, every turn:** "is this new element allowed in?" — this question picks your state structure. "Is X inside?" → set. "How many of X?" → counter.
3. **The order that protects the promise:** **ask → fix → enter → measure.** Check for dirt *before* adding the newcomer — adding first makes your own addition indistinguishable from a real violation. The shrink is a `while`, not an `if` — one kick isn't always enough. Only measure the window (update your "best") after it's clean again.

### Code skeleton
```python
window_state = {}  # or set(), or a running sum
left = 0
best = 0
for right in range(len(arr)):
    # update window_state to include arr[right]
    while window_is_dirty(window_state):
        # remove arr[left] from window_state
        left += 1
    best = max(best, right - left + 1)
return best
```

### The phrase
**"Grow right while clean. Shrink left when dirty."**

---

## Fast & Slow Pointers

### Recognize it
- Linked list + cycle/loop detection
- "Middle of the list", "k-th from the end"
- "Detect if a sequence repeats" — even with no list in sight (a number transformed repeatedly, an array where index points to another index)
- The giveaway follow-up: "can you do it with O(1) space?" after a visited-set solution

### The story
> **Two runners start together. Slow takes 1 step per turn. Fast takes 2. If the track loops, Fast comes around from behind and catches Slow. If the track ends, Fast falls off the edge first.**

### Build the logic
1. **The promise:** "Fast is always exactly twice as far from the start as Slow."
2. **The question, every turn:** "can Fast safely take two steps?" and "are Slow and Fast now in the same spot?" Neither needs a data structure — that's the whole point of the O(1)-space upgrade over a visited-set.
3. **The order that protects the promise:** move both pointers *before* comparing them — they start in the same place, so comparing first gives a false positive. Bounds check (`fast and fast.next`) always comes before dereferencing.

### The proof, so you never have to take it on faith
Once inside a loop, look at the *gap* between the runners, not their positions. Slow moves 1, Fast moves 2 — the gap shrinks by exactly 1 every turn. A number shrinking by exactly 1 must pass through 0 — it can't jump over it. Gap 0 = same spot = caught.

### Code skeleton
```python
slow = fast = head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
    if slow is fast:
        return True  # cycle found
return False
```

### The phrase
**"If the track loops, the fast runner laps the slow one."**

---

## Prefix Sums

### Recognize it
- "Sum of a range", especially with many repeated queries
- "Count subarrays with sum/property k"
- Subarray sums where **negatives are allowed** — this is the tell that separates it from sliding window
- Disguises: "equal 0s and 1s" (map 0 → −1), "divisible by k" (markers by remainder)

### The story
> **Plant a mile marker at every step of the road. Then the length of any stretch is just: marker at the end minus marker at the start. No re-walking.**

### Build the logic
1. **The promise:** "`prefix[i]` always equals the sum of the first `i` elements — including the seeded 0 before the road begins."
2. **The question, every turn (for the counting variant):** "how many times have I seen the marker `current − k` before?" — a counting question, so a **counter/dict**, not a set (different earlier positions can share the same marker value, and each is a separate valid chunk).
3. **The order that protects the promise:** step forward (update your running marker) *before* asking the question — the question is about the marker at the current position. Ask *before* recording your own marker into the counter — recording first lets you match yourself, which is wrong.

### Code skeleton (range sum, precomputed)
```python
prefix = [0]
for x in nums:
    prefix.append(prefix[-1] + x)
# sum of index l..r inclusive:
range_sum = prefix[r + 1] - prefix[l]
```

### Code skeleton (count subarrays summing to k)
```python
count = 0
current = 0
seen = {0: 1}  # the zero marker, planted before the road
for x in nums:
    current += x
    count += seen.get(current - k, 0)
    seen[current] = seen.get(current, 0) + 1
return count
```

### The phrase
**"Mile markers: any stretch is end marker minus start marker."**

---

## Binary Search

### Recognize it
- Sorted array + find/threshold/first/last/insert position
- **"Minimize the maximum" / "maximize the minimum"** → binary search on the *answer*, not the array
- "Smallest x such that...", "least capacity/speed/days to..."
- The deep test: can you write a yes/no question that flips exactly once across the space? (This is called a **monotonic check**.) If yes, it's binary searchable — sortedness of the input was never the actual requirement.

### The story
> **Ask one question in the middle. The answer tells you which half is worthless. Throw that half away. Repeat.**

### Build the logic
1. **The promise:** "the first YES is always somewhere inside `[lo, hi]`."
2. **The question, every turn:** one yes/no check on the middle candidate. Before coding: identify what your check actually is, and confirm it's monotonic (NO-NO-NO-YES-YES, never flips back).
3. **The order that protects the promise — derive, don't memorize:** ask "could `mid` still be the answer?" YES → keep it as a suspect (`hi = mid`). NO → it's proven innocent, eliminate it (`lo = mid + 1`). Verify the room shrinks every turn either way (or you'll loop forever).

### Code skeleton
```python
lo, hi = 0, n - 1
while lo < hi:
    mid = (lo + hi) // 2
    if check(mid):      # mid might be the first YES — keep it
        hi = mid
    else:                # mid is proven NO — eliminate it
        lo = mid + 1
return lo
```

### The phrase
**"Paint it NO-NO-YES-YES, then hunt the first YES."**

---

## Merge Intervals

### Recognize it
- Words: intervals, ranges, meetings, bookings, schedules, timelines
- "Merge overlapping...", "can attend all...", "minimum rooms...", "insert into schedule"
- Pairwise overlap checks that look like O(n²) at first glance

### The story
> **Sort the meetings by start time. Then walk through them once, asking each one: "do you overlap the last block on my calendar?" If yes — stretch that block. If no — start a fresh block.**

### Build the logic
1. **The promise:** "after processing each interval, `merged` is correct and non-overlapping for everything seen so far."
2. **The question, every turn:** "does this interval overlap `merged[-1]`?" — just one comparison, because sorting already guarantees you can't overlap anything *earlier* than the last block.
3. **The order that protects the promise:** compare against `merged[-1]` — the *output* being built — never against the previous *input* element (after a merge, the block is bigger than any single input). When stretching, use `max(last.end, cur.end)`, never plain assignment — a nested interval must not shrink the block.

### Code skeleton
```python
intervals.sort(key=lambda x: x[0])
merged = [intervals[0]]
for cur in intervals[1:]:
    last = merged[-1]
    if cur[0] <= last[1]:
        last[1] = max(last[1], cur[1])
    else:
        merged.append(cur)
return merged
```

### The phrase
**"Sort by start; overlap the last or start fresh."**

---

## The three rules that show up in every pattern above

These aren't pattern-specific. They're the same three rules, worn as a different costume in every single pattern in this file. Once you see them once, you'll see them everywhere.

1. **Read before you write.** Ask your question while the world still reflects the old state. In sliding window: check for dirt before adding. In prefix sums: ask before recording your own marker. In two pointers: snapshot a value before it might get overwritten.

2. **Bounds before contents.** Check a position exists before looking inside it. `while j >= 0 and s[j] != ' '` — not the other way around. `while fast and fast.next` — existence before dereference.

3. **Resets restore the promise, nothing else.** Every pointer update exists to make the loop's opening assumption true again for the next turn. If you're not sure where an update goes, ask: "what does the top of my loop assume, and what update makes that true?"

---

## Data structure cheat sheet

Pick the container by the question your loop asks — not by what it holds.

| The question your loop keeps asking | Use this |
|---|---|
| "Is X inside?" | **set** |
| "How many of X are inside?" | **dict / Counter** |
| "What is at position i? Keep things in order?" | **list** |
| "What entered first? (process in arrival order)" | **deque (queue)** |
| "What entered last? (undo, matching, nesting)" | **list as stack** |
| "What is the smallest/largest right now, repeatedly?" | **heap** |
| "Building a string piece by piece" | **list, then `''.join()` once — never `+=` in a loop** |

---

## The recognition checklist (run this before writing any code)

1. Restate the problem in one sentence.
2. Ask about the edges: empty input? negatives? duplicates? how big is n?
3. Say the honest brute force out loud, and its complexity.
4. Match the shape to a pattern above — say the pattern's name **and its story**, not just the name. ("Contiguous chunk with state, so sliding window" — not just "sliding window.")
5. Run the three-question ritual: promise, question, order.
6. Trace one small, deliberately nasty example by hand before trusting the code.
7. State time and space complexity, unprompted — and if there's a nested loop, check whether it's actually O(n) via amortized movement before assuming O(n²).

---

## All the phrases, in one place

- **Two pointers:** *"Walk backwards, skip spaces, grab whole words."* (Reader/writer: *"Reader scans, writer marks where clean data ends."*)
- **Sliding window:** *"Grow right while clean. Shrink left when dirty."*
- **Fast & slow pointers:** *"If the track loops, the fast runner laps the slow one."*
- **Prefix sums:** *"Mile markers: any stretch is end marker minus start marker."*
- **Binary search:** *"Paint it NO-NO-YES-YES, then hunt the first YES."*
- **Merge intervals:** *"Sort by start; overlap the last or start fresh."*
- **In-place array split:** *"Scan and keep the group whose order matters. Whatever's left, I already know what goes there."*
- **Choosing structures:** *"Pick by the question your loop asks, not by what the container holds."*

Say the phrase before you write any code. Let the promise it implies place every line that follows.
