---
title: "Binary Search, Explained So Simply You Can Never Forget It"
published: false
description: "Binary search taught as a story: paint the space NO-NO-YES-YES, then hunt the first YES. The boundary template that kills off-by-one bugs, binary search on the answer, and a practice ladder."
tags: algorithms, python, interview, leetcode
---

*This is part of a series where I learn DS & Algo patterns as plain-English stories first, and only then turn the stories into code. The goal is muscle memory — being able to rebuild the code from the story, any time, from scratch. Previously: Two Pointers, Sliding Window, Fast & Slow Pointers, Prefix Sums.*

---

## The story in one sentence

> **Ask one question in the middle. The answer tells you which half is worthless. Throw that half away. Repeat.**

The childhood game: I'm thinking of a number between 1 and 100. You guess 50. I say "higher." One question just destroyed fifty numbers. You guess 75. "Lower." Twenty-five more gone. Seven questions corner any number out of a hundred.

That halving is the whole pattern — and it is why the cost is **O(log n)**. Each question kills half the remaining space, so a million items need only about 20 questions (log₂ 1,000,000 ≈ 20).

---

## What problem are we solving?

Two, in sequence:

- **LeetCode 704 — Binary Search** — the classic "find the target in a sorted array."
- **LeetCode 278 — First Bad Version** — the version that reveals what the pattern *really* is.

Fair warning: this looks like the easiest pattern in the whole map, and it has the highest off-by-one bug rate of them all. Jon Bentley famously found that most professional programmers could not write a correct binary search. The fix is not more care. The fix is a better mental picture.

---

## The reframe that kills off-by-one bugs

Everyone learns binary search as "find the target." That framing is *why* everyone gets the `+1`s wrong — "found it / go left / go right" gives you three cases to juggle and no principle to decide the offsets.

Here is the better picture. Imagine the search space painted in two colors:

```
[ NO  NO  NO  NO | YES YES YES ]
                 ↑
            the boundary
```

Some question — call it the **check** — answers NO for everything on the left and YES for everything on the right, with one clean switch point. A check that flips one way and never flips back is called **monotonic**. Binary search's real job:

> **Find the boundary — the first YES.**

LC 278 is literally this picture: versions `[good good good | bad bad]`, find the first bad one. And "find target 7 in a sorted array" is this picture too — the check is *"is this value ≥ 7?"*, and the first YES is where 7 lives, if it exists. One picture covers every binary search you will ever write.

---

## Build the code, sentence by sentence

**Sentence 1 — the promise (the loop invariant):**

> "The first YES is always somewhere inside `[lo, hi]`."

Write it down. Every line either uses this promise or protects it. Every offset decision is derived from it.

```python
lo, hi = 0, n - 1
```

**Sentence 2: "While more than one suspect remains, question the middle one."**

"More than one suspect" → the room `[lo, hi]` has at least two slots → `lo < hi`:

```python
while lo < hi:
    mid = (lo + hi) // 2
```

**Sentence 3: "If the middle says YES — it might BE the first yes. Keep it. But everything after it is worthless."**

```python
    if check(mid):
        hi = mid        # mid stays a suspect
```

Stare at `hi = mid` — **not** `mid - 1`. A YES might be *the* first YES. Throw it away and the promise "the answer is in [lo, hi]" is broken, and no later code can repair it. This is the single most common binary search bug, and the promise makes it impossible to commit.

**Sentence 4: "If the middle says NO — it cannot be the answer, and neither can anything before it."**

```python
    else:
        lo = mid + 1    # mid is eliminated
```

Here the `+1` is *required*: a NO is proven innocent. Keeping it wastes a suspect and (worse) can freeze the loop.

**Sentence 5: "One suspect left. That is the boundary."**

When `lo == hi`, the room has one slot, and the promise says the answer is in the room. So:

```python
return lo
```

The full template:

```python
lo, hi = 0, n - 1
while lo < hi:
    mid = (lo + hi) // 2
    if check(mid):
        hi = mid
    else:
        lo = mid + 1
return lo
```

Every offset came from one question — **"could mid still be the answer?"**

- YES → keep it: `hi = mid`.
- NO → eliminate it: `lo = mid + 1`.

You never memorize the offsets. You re-derive them from the promise in two seconds. That is the entire cure for off-by-one bugs.

**One safety fact:** `(lo + hi) // 2` rounds *down*, so when exactly two suspects remain, `mid` is the left one. YES shrinks the room to `[lo, mid]` — strictly smaller. NO moves it to `[mid+1, hi]` — strictly smaller. Either way the room shrinks every turn, so the loop cannot spin forever. (If you ever write the mirrored version — hunting the *last* YES with `lo = mid` — then mid must round *up*: `(lo + hi + 1) // 2`. Otherwise mid can equal lo, `lo = mid` changes nothing, and you get the classic infinite loop. Same principle either way: **every turn must shrink the room.**)

---

## Full trace: LC 278, versions 1–5, first bad is version 4

Check = "is this version bad?" Picture: `[good good good | bad bad]`.

**Start:** `lo = 1, hi = 5`. Promise: the first bad version is somewhere in [1, 5].

**Turn 1.** `mid = 3`. Is version 3 bad? **No.** Version 3 is innocent, and so is everything before it. `lo = 4`. Room: [4, 5]. Promise still holds.

**Turn 2.** `mid = 4`. Is version 4 bad? **Yes.** It might be the first — keep it. `hi = 4`. Room: [4, 4].

**Loop check:** `lo < hi`? 4 < 4 — no. One suspect left.

**Return 4.** ✓

Two questions for five versions. Twenty for a million. That's the halving at work.

---

## The superpower: binary search on the *answer*

The sorted-array picture is training wheels. The real weapon:

> **Any monotonic yes/no question can be binary searched — no array required.**

Take LC 875, *Koko Eating Bananas*: find the minimum eating speed that finishes all piles within h hours. There is no sorted array anywhere in this problem. But ask the check question — *"can she finish at speed s?"* — and watch it paint the picture: if YES at speed 6, then certainly YES at 7, 8, 9 (faster only helps). If NO at speed 3, then certainly NO at 2 and 1. That is `NO NO NO | YES YES YES`, painted over *speeds* instead of array indexes.

So binary search the speed:

```python
def minEatingSpeed(piles, h):
    def can_finish(speed):
        return sum((p + speed - 1) // speed for p in piles) <= h

    lo, hi = 1, max(piles)
    while lo < hi:
        mid = (lo + hi) // 2
        if can_finish(mid):
            hi = mid
        else:
            lo = mid + 1
    return lo
```

Same template, letter for letter. The only new work was writing the check as a helper function.

**Interview recognition tell:** the words **"minimize the maximum"** or **"maximize the minimum"** almost always mean binary search on the answer. See those words, ask "is the check monotonic?", and you've likely cracked the problem before writing a line. This is one of the highest-value tricks in the entire pattern map.

---

## How to think your way to this code

Same ritual as every pattern in this series.

**Question 1 — What is the promise?**

"The first YES is inside [lo, hi]." Everything is derived from it.

**Question 2 — What must I ask each turn?**

One check question about the middle. Before anything else, ask *of the problem*: "what is my check, and is it monotonic — does it flip NO→YES exactly once?" If you can't find a monotonic check, binary search is the wrong pattern, no matter how sorted things look.

**Question 3 — What order/offsets keep the promise?**

Derive, don't memorize: could mid still be the answer? YES → `hi = mid`. NO → `lo = mid + 1`. Then verify the room shrinks in both branches. Two derivations and one shrink-check replace every off-by-one table you've ever squinted at.

---

## Common mistakes (I made half of these)

**1. `hi = mid - 1` after a YES.** Throws away a suspect who might be the answer. Breaks the promise; the bug that produces "sometimes off by one" answers.

**2. `lo = mid` after a NO.** Keeps a proven-innocent suspect. With mid rounding down, `lo = mid` can change nothing → infinite loop.

**3. Three-way branching by reflex.** `if == target / elif < / else` works for exact find, but it doesn't generalize to first/last occurrence, insert position, or answer-space searches. The two-color boundary template handles all of them uniformly.

**4. `while lo <= hi` mixed with `hi = mid`.** The `<=` loop is a different template (exact-match, shrink-past style). Mixing its condition with the boundary template's updates → infinite loop. Pick one template and stay in it.

**5. Forgetting to check "does the YES region even exist?"** If the target is absent, `lo` lands on the first value ≥ target — which may not equal the target. For exact-find problems, verify after the loop: `return lo if s[lo] == target else -1`.

**6. Missing the monotonic check.** Binary searching a space where the check flips YES→NO→YES gives confident wrong answers. The sortedness of an array was never the requirement — the monotonicity of the *check* is.

---

## The phrase to never forget

> **"Paint it NO-NO-YES-YES, then hunt the first YES."**

If you remember that phrase, the template rebuilds itself: the promise, the `lo < hi`, and both offsets fall out of "could mid still be the answer?"

---

## Practice ladder

Do these in order. Each one reuses the last one's muscle and adds one small twist.

1. **LC 704 — Binary Search.** The classic — but do it with the boundary template: check = "is this value ≥ target?", then verify the landing spot.
2. **LC 278 — First Bad Version.** The pattern in its purest form. The one traced above.
3. **LC 35 — Search Insert Position.** The realization: it is *identical* to 704's boundary. The first YES **is** the insert position. No new code.
4. **LC 69 — Sqrt(x).** First taste of searching a number range instead of an array. Check: "is mid² ≤ x?"
5. **LC 153 — Minimum in Rotated Sorted Array.** The check gets cleverer: "is this element < the last element?" paints the two colors across the rotation.
6. **LC 875 — Koko Eating Bananas.** Binary search on the answer. The level-up. The check becomes a whole helper function.
7. **LC 33 — Search in Rotated Sorted Array.** The interview favorite; combines 153's insight with plain search.
8. **LC 410 — Split Array Largest Sum.** The boss fight. "Minimize the largest sum" — pure binary-search-on-answer, and the moment the pattern feels like a superpower.

When the ladder is done, close everything and re-derive LC 278 from the phrase alone. If you can, the pattern is yours.

---

*Previously in this series: Two Pointers (LC 151), Sliding Window (LC 3), Fast & Slow Pointers (LC 141), Prefix Sums (LC 560). Next up: Merge Intervals — sort by start time, then walk asking "does this overlap the last?"*
