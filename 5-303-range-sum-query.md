---
title: "Prefix Sums, Explained So Simply You Can Never Forget It"
published: false
description: "The prefix sum pattern taught as a story: mile markers on a highway — any stretch is end marker minus start marker. With traces, the LC 560 hashmap move, and a practice ladder."
tags: algorithms, python, interview, leetcode
---

*This is part of a series where I learn DS & Algo patterns as plain-English stories first, and only then turn the stories into code. The goal is muscle memory — being able to rebuild the code from the story, any time, from scratch. Previously: Two Pointers, Sliding Window, Fast & Slow Pointers.*

---

## The story in one sentence

> **Plant a mile marker at every step of the road. Then the length of any stretch is just: marker at the end minus marker at the start. No re-walking.**

Think of a highway. Mile markers count total distance from the start: 0, 5, 12, 20... To know how long the stretch between marker 5 and marker 20 is, you don't drive it again. You subtract: 20 − 5 = 15.

Prefix sums are mile markers for arrays. That is the entire pattern.

---

## What problem are we solving?

Two, in sequence:

- **LeetCode 303 — Range Sum Query, Immutable** — the pure form of the pattern.
- **LeetCode 560 — Subarray Sum Equals K** — the famous interview version, where the pattern combines with a hashmap.

LC 303 in plain words: given an array, answer many questions of the form *"what is the sum of boxes l through r?"* — fast.

---

## The setup: boxes and mile markers

```
nums   =  [ 3,  1,  4,  2 ]
boxes       0   1   2   3
```

Build a second array, `prefix`, where `prefix[i]` = *sum of the first i numbers*. Read that carefully: first **i** numbers, not "up to box i."

```
prefix =  [ 0,  3,  4,  8, 10 ]
             ↑   ↑   ↑   ↑   ↑
        sum of: 0   1   2   3   4  numbers
```

Two things to stare at:

- `prefix` has **one extra slot**, and it starts with **0**. That leading zero is the mile marker planted *before* the road begins — "zero miles traveled." It looks pointless. It is the most important slot in the array.
- Each marker is the previous marker plus one number: `prefix[i+1] = prefix[i] + nums[i]`. Building all the markers is a single O(n) walk.

---

## The one formula

> **Sum of boxes `l` through `r` = `prefix[r+1] − prefix[l]`**

Why it works: `prefix[r+1]` is everything from the start through box `r`. `prefix[l]` is everything from the start *before* box `l`. Subtract, and the shared beginning cancels. What remains is exactly the chunk `l..r`.

Check it on the picture: sum of boxes 1–2 is 1 + 4 = 5. Formula: `prefix[3] − prefix[1]` = 8 − 3 = 5. ✓

Now watch the leading zero earn its keep. Sum of boxes 0–2 (a chunk starting at the very front): `prefix[3] − prefix[0]` = 8 − 0 = 8. ✓ No special case. No `if l == 0` branch.

> **The zero marker exists so that "from the very start" is not a special case.**

Off-by-one errors are the number one bug in prefix sums, and the n+1-sized array with a leading zero is the vaccine. Take the convention; don't fight it.

---

## Build the code, sentence by sentence

**Sentence 1: "Plant a marker before the road, then one after every step."**

```python
class NumArray:
    def __init__(self, nums):
        self.prefix = [0]                       # marker before the road
        for x in nums:
            self.prefix.append(self.prefix[-1] + x)   # last marker + one step
```

**Sentence 2: "Any stretch is end marker minus start marker."**

```python
    def sumRange(self, l, r):
        return self.prefix[r+1] - self.prefix[l]
```

That is all of LC 303.

**The trade you just made:** build once for O(n); every question afterward costs O(1). Answering each question by re-adding the chunk would cost O(q·n) for q questions. Markers make it O(n + q).

> **One walk of preparation, so that every later question becomes a subtraction.** That sentence is the pattern's whole identity.

---

## The big move: LC 560, Subarray Sum Equals K

*Count how many contiguous chunks sum to exactly k.* Negatives allowed. This is where prefix sums stop being a warm-up and start winning interviews.

Start from the formula and rearrange it. A chunk `l..r` sums to k when:

```
prefix[r+1] − prefix[l] = k
        ⇕  (rearrange)
prefix[l] = prefix[r+1] − k
```

Read the second line as a story:

> **"I'm standing at marker `prefix[r+1]`. Every *earlier* marker equal to `my marker − k` is the start of a chunk that ends right here and sums to k."**

So walk the road once, carrying a running total — your current marker. At each step, ask one question: *"how many times have I seen the marker `current − k` before?"*

"How many times have I seen X?" — that question picks the data structure by itself: a **counter (dict)**. Not a set — a set only knows yes/no, and several different earlier positions can share the same marker value, each one a separate valid chunk.

```python
def subarraySum(nums, k):
    count = 0
    current = 0                 # my marker so far
    seen = {0: 1}               # the zero marker, planted before the road
    for x in nums:
        current += x                            # step forward
        count += seen.get(current - k, 0)       # ask: how many chunk-starts end here?
        seen[current] = seen.get(current, 0) + 1  # THEN record my marker
    return count
```

Three lines in the body, and each one's *position* is forced:

**1. `current += x` first.** The question is about the marker at the end of this box. Update before asking.

**2. Ask before recording.** When we ask, `seen` must contain only *earlier* markers. Record first and — when `k = 0` — you would find your *own* marker and count a chunk of zero length that doesn't exist. This is **read before you write**, the same rule that ordered the sliding window's clean-then-add and the two-pointer's check-then-touch. Third pattern in a row; it's not a coincidence, it's the rule.

**3. `seen = {0: 1}` before the walk.** The zero marker again. Without it, every chunk that starts at box 0 goes uncounted — its "earlier marker" is the one planted before the road began, and nobody planted it.

---

## Full trace on `nums = [1, 1, 1]`, k = 2

**Start:** `current = 0`, `seen = {0: 1}`, `count = 0`.

**Box 0** (x = 1). Marker → 1. Ask: have I seen `1 − 2 = −1`? No. Record 1. `seen = {0:1, 1:1}`.

**Box 1** (x = 1). Marker → 2. Ask: have I seen `2 − 2 = 0`? **Yes — once. The seeded zero marker.** `count = 1`. That is the chunk `[1, 1]` starting at the very front — the leading zero just earned its keep, live, in the trace. Record 2. `seen = {0:1, 1:1, 2:1}`.

**Box 2** (x = 1). Marker → 3. Ask: have I seen `3 − 2 = 1`? **Yes — once, from box 0.** `count = 2`. That is the chunk `[1, 1]` covering boxes 1–2. Record 3.

Answer: **2**. ✓

One walk. **O(n) time, O(n) space** for the marker counts. And notice what we never built: the full `prefix` array. Just a running number and a counter. The markers went invisible — but the thinking is still "marker minus marker."

---

## Why not sliding window here?

Sharp question — "sum of a chunk" smells exactly like a window. Here is why the window fails on LC 560:

Sliding window's shrink logic needs a promise: *growing makes the sum bigger, shrinking makes it smaller.* That promise holds only when every number is positive. LC 560 allows **negatives** — growing the window can make the sum go *down*, so "too big, shrink" stops meaning anything. The window has no ground to stand on.

Prefix + counter doesn't care about signs at all. Hence the rule of thumb:

> **Subarray sums, all numbers positive → sliding window. Negatives allowed → prefix sums.**

This is worth saying out loud in an interview before you pick — it shows you chose the pattern, rather than pattern-matching on the word "subarray."

---

## How to think your way to this code

Same ritual as every pattern in this series. Three questions before any code.

**Question 1 — What is the promise?**

For the walk in LC 560: *"At the moment I ask, `seen` contains exactly the markers of all earlier positions — and nothing else."* That invariant is what makes the count correct.

**Question 2 — What must I ask each turn?**

*"How many times have I seen `current − k`?"* — a counting question → **dict/Counter**. (Compare across the series: two pointers asked "space or letter?" → no structure. Sliding window asked "is X inside?" → set. Prefix sums ask "how many times X?" → counter. The question always picks the structure.)

**Question 3 — What order keeps the promise?**

Step, ask, record — in that order. Recording before asking poisons `seen` with the current marker and breaks the "earlier positions only" promise. The heartbeat, one more time: **ask → then write.**

---

## Common mistakes (I made half of these)

**1. No leading zero.** `prefix` sized n instead of n+1 → chunks starting at box 0 need a special `if`, and half the formulas grow warts. Plant the zero marker; it deletes the special case.

**2. Forgetting to seed `{0: 1}`.** The hashmap version of the same bug. Symptom: answers that are exactly short by the chunks touching the front of the array.

**3. Recording before asking.** With `k = 0`, you count your own marker — phantom zero-length chunks. Read before you write.

**4. Using a set instead of a counter.** Several earlier positions can hold the *same* marker value; each is a separate chunk. A set collapses them into one. The question was "how many," not "whether."

**5. Off-by-one in the formula.** `prefix[r] − prefix[l]` instead of `prefix[r+1] − prefix[l]`. Re-derive it from the words: end marker is *after* box r, start marker is *before* box l.

**6. Reaching for sliding window with negatives present.** The window's promise (grow → bigger) is broken by negative numbers. Check the signs before you pick the pattern.

---

## The phrase to never forget

> **"Mile markers: any stretch is end marker minus start marker."**

If you remember that phrase, both versions rebuild themselves — the array version directly, and the hashmap version by rearranging the subtraction into a question about earlier markers.

---

## Practice ladder

Do these in order. Each one reuses the last one's muscle and adds one small twist.

1. **LC 1480 — Running Sum of 1d Array.** Literally "build the markers." A warm-up you'll finish before your coffee cools.
2. **LC 303 — Range Sum Query, Immutable.** The pure pattern, as above.
3. **LC 724 — Find Pivot Index.** Left sum vs right sum — both read off one marker and the total.
4. **LC 560 — Subarray Sum Equals K.** The big move. Redo it from the phrase tomorrow.
5. **LC 525 — Contiguous Array.** The trick: treat every 0 as −1, and "equal 0s and 1s" becomes "chunk summing to 0." Same code, one costume change. This problem teaches you to *see* prefix sums in disguise.
6. **LC 974 — Subarray Sums Divisible by K.** Markers stored by remainder mod k. Same skeleton, modular twist.
7. **LC 304 — Range Sum Query 2D.** The boss fight: mile markers on a *grid*. The subtraction becomes four corners — inclusion-exclusion. Draw the rectangle before you code.

When the ladder is done, close everything and re-derive LC 560 from the phrase alone. If you can, the pattern is yours.

---

*Previously in this series: Two Pointers (LC 151), Sliding Window (LC 3), Fast & Slow Pointers (LC 141). Next up: Binary Search — cut the search space in half by asking one question.*
