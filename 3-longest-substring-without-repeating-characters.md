---
title: "Sliding Window, Explained So Simply You Can Never Forget It"
published: false
description: "The sliding window pattern taught as a story: grow right while clean, shrink left when dirty. With box-by-box traces, code derived sentence by sentence, and a practice ladder."
tags: algorithms, python, interview, leetcode
---

*This is part of a series where I learn DS & Algo patterns as plain-English stories first, and only then turn the stories into code. The goal is muscle memory — being able to rebuild the code from the story, any time, from scratch.*

---

## The story in one sentence

> **A stretchy rubber band lies over the array. Grow the right end while the band is clean. When it gets dirty, shrink the left end until it is clean again.**

That is the entire pattern. Everything below just unpacks these words.

---

## What problem are we solving?

We will learn the pattern on one concrete problem: **LeetCode 3 — Longest Substring Without Repeating Characters.**

In plain words: given a string, find the longest chunk with no repeated letter.

Example: in `"abcba"`, the answer is `"abc"` — length 3.

Why this problem? Because it forces every part of the pattern to show up: growing, shrinking, tracking what is inside, and measuring the best answer.

---

## The setup: boxes and two fingers

A string is just characters in numbered boxes:

```
s =  "abcba"
      01234
```

The **window** is a chunk of boxes. We mark it with two numbers:

- `left` — the box where the window starts.
- `right` — the box where the window ends.

These are our two **pointers**. A pointer is nothing magical. It is just a number that says which box we are looking at.

Both start at box 0. So the window starts tiny — one box wide.

We need one more tool: a **set**. Think of it as a *guest list*. It holds the letters currently inside the window. Its only job is to answer one question fast: *"is this letter already inside?"*

---

## The rules of the game

There are only two rules.

**Rule 1 — Grow.** Move `right` one box at a time. Each new letter tries to enter the window.

**Rule 2 — Shrink.** If the new letter is already on the guest list, the window is *dirty*. Kick letters out from the left — move `left` rightward — until the copy is gone. Then the new letter may enter.

Why kick from the left? Because the duplicate is somewhere inside the window. The window must stay one solid, contiguous chunk. You cannot pluck a letter from the middle of a chunk. The only legal move is to slide the left wall in.

And here is the promise the window always keeps:

> **At the end of every turn, the window has no repeats.**

This promise has a name: the **loop invariant**. It means: the top of the loop assumes something is true, and before we loop back, we must make it true again. Every clean loop you have ever written works this way — sliding window just makes it visible.

---

## Build the code, sentence by sentence

Do not write code first. Write the story as sentences. Then let each sentence force the code into place.

**Sentence 1: "Move the right end across the whole string."**

That sentence *is* the outer loop:

```python
for right in range(len(s)):
```

**Sentence 2: "If the new letter is already inside, shrink from the left until it's not."**

"Already inside" → ask the guest list: `s[right] in seen`.

"Shrink until it's not" → a `while` loop. Here is a rule worth tattooing somewhere: **a `while` condition describes when to KEEP GOING, not when to stop.** The story says "stop when the copy is gone." Flip it: "keep going while the copy is still inside."

```python
    while s[right] in seen:
        seen.remove(s[left])
        left += 1
```

Read the shrink body slowly. Two small acts, always in this order:

1. Kick out the letter standing at the left wall: `seen.remove(s[left])`.
2. Move the wall one box right: `left += 1`.

**Sentence 3: "Now the letter may enter."**

```python
    seen.add(s[right])
```

**Sentence 4: "The window is clean — is it our best so far?"**

```python
    best = max(best, right - left + 1)
```

`right - left + 1` is the number of boxes in the window. The `+1` is there because both ends are included. (Same reason a fence from post 2 to post 5 has 4 posts, not 3.)

Put together:

```python
def lengthOfLongestSubstring(s: str) -> int:
    seen = set()
    left = 0
    best = 0
    for right in range(len(s)):
        while s[right] in seen:     # dirty? shrink
            seen.remove(s[left])
            left += 1
        seen.add(s[right])          # now it may enter
        best = max(best, right - left + 1)
    return best
```

Notice *where* each line sits, and why it could sit nowhere else:

- The shrink `while` comes **before** the `add`. The letter may not enter until the window is clean.
- The `best` update comes **after** the `add`. We only measure clean windows. Measuring before cleaning would count dirty ones.
- `left += 1` comes **after** the `remove`. Swap them and you kick out the wrong letter.

This is the deep trick for "where do I put what": **each line's position is forced by the promise (the invariant) the loop makes.** If a line could break the promise, it must run before the promise is checked or restored.

---

## Full trace on `"abcba"`

Watch every turn. Watch which line fires and why.

```
s =  "abcba"
      01234
```

**Turn 1.** `right = 0`, letter `a`. Guest list empty — clean. Add `a`. Window is boxes 0–0. Size 1. `best = 1`.

**Turn 2.** `right = 1`, letter `b`. Not on the list. Add it. Window 0–1: `"ab"`. `best = 2`.

**Turn 3.** `right = 2`, letter `c`. Clean. Add. Window 0–2: `"abc"`. `best = 3`.

**Turn 4.** `right = 3`, letter `b`. **Dirty** — `b` is already inside. Shrink:

- Kick `s[left]` = `s[0]` = `a`. Move `left` to 1. Is `b` still inside? Yes. Keep shrinking.
- Kick `s[1]` = `b`. Move `left` to 2. Is `b` still inside? No. Clean.

Now the new `b` enters. Window 2–3: `"cb"`. Size 2. `best` stays 3.

Pause on this turn. The shrink kicked `a` too — not just the old `b`. It had to. The window must stay one solid chunk. Innocent letters near the left wall get kicked because they stand between the wall and the duplicate. This is the moment most people find surprising, and it is exactly what "the window stays contiguous" means in practice.

**Turn 5.** `right = 4`, letter `a`. Is `a` inside? No — it was kicked in turn 4. Add it. Window 2–4: `"cba"`. Size 3. `best = 3`.

Done. Answer: **3**.

---

## "It's a while inside a for — isn't that O(n²)?"

This question deserves its own section, because the answer rewires how you read nested loops.

The "nested loops = n²" rule is a *heuristic*, not a law. It holds when the inner loop can redo up to n work for **each** turn of the outer loop. That is not what happens here.

Look at what `left` actually does across the *whole run*: it only ever moves right. It never resets. It never backs up. So no matter how the turns divide it up, `left` can take at most n steps **total** — across all shrinks combined.

Count the budget:

- `right` takes n steps (one per turn).
- `left` takes at most n steps (total, ever).
- Total: about 2n steps. That is **O(n)**.

This style of counting has a name: **amortized analysis**. Amortized just means "averaged over the whole run instead of per turn." One turn might shrink a lot. But every box `left` walks over is a box it never walks again — the expensive turn *spends* budget that cheap turns saved up.

The rule of thumb: **don't count loop levels — count total pointer movement.** If the inner loop's progress is never undone, all the loops share one budget of n steps.

(And here is what a *real* O(n²) version would look like: if, after finding a duplicate, we reset `left` back to some earlier position and re-scanned. The no-backing-up property is what kills the quadratic.)

---

## Two flavors of window

Everything above is the **variable-size window**: it grows and shrinks as needed. There is a simpler sibling.

**Fixed-size window.** The band never changes size — say, always k boxes. It just slides one box per turn: the right end takes one letter in, the left end lets one letter go. Classic use: "max sum of any k consecutive elements" (LeetCode 643).

```python
def maxSum(nums, k):
    window_sum = sum(nums[:k])      # first window
    best = window_sum
    for right in range(k, len(nums)):
        window_sum += nums[right]        # one in
        window_sum -= nums[right - k]    # one out
        best = max(best, window_sum)
    return best
```

The beautiful part: each slide is O(1). We do not re-add k numbers. We adjust by the one that entered and the one that left. That "adjust, don't recompute" move is the entire reason windows are fast.

How to know which flavor a problem wants:

- The problem *gives* you the size ("every window of size k...") → **fixed**.
- The problem asks you to *find* a size ("longest...", "shortest...", "at most k distinct...") → **variable**.

---

## The state can be anything

In LC 3 the state was a set. But the guest list is just one costume. The state is *whatever describes the inside of the window well enough to answer "is it clean?" fast*:

- A **running sum** — for "smallest subarray with sum ≥ target" (LC 209).
- A **counter / frequency map** — for "longest substring with at most k distinct characters", or "does this window contain a permutation of t?" (LC 424, 567, 76).
- A **count of zeros** — for "longest run of 1s if you may flip at most k zeros" (LC 1004).

The skeleton never changes. Only two things change per problem:

1. What "dirty" means (the `while` condition).
2. What bookkeeping enters and leaves with each letter.

That is why this is a *pattern* and not a trick.

---

## Wait — isn't this just two pointers?

Yes — and that is not a gotcha. **Sliding window is a special kind of two pointers.** All windows use two pointers. Not all two-pointer code is a window.

The family tree:

```
two pointers  (the family)
├── opposite ends, walk inward      → palindrome check, container with water
├── reader / writer                 → remove duplicates, move zeroes
├── fence a chunk, then move on     → reverse words in a string
└── live chunk with state           → SLIDING WINDOW
```

The quick test — three questions:

1. **Does the problem ask about a contiguous chunk?** Words like *substring*, *subarray*, "longest...", "shortest..." → smells like window.
2. **Do both pointers move the same direction?** Windows always slide one way. Pointers walking toward each other from both ends → plain two pointers.
3. **Do I keep extra state describing what's between the pointers?** A set, a sum, a counter → window. No state about the middle → two pointers.

One line to remember: **two pointers is about the positions; sliding window is about the stuff between the positions.**

If an interviewer asks "is this two pointers or sliding window?", a strong answer is: *"Sliding window — which is a two-pointer technique where the window between the pointers carries state."* That one sentence signals real understanding, not memorized names.

---

## Common mistakes (I made half of these)

**1. Adding before cleaning.** Putting `seen.add(s[right])` before the shrink `while`. Now the duplicate check finds the letter you *just* added, and the shrink eats the whole window. Order: clean first, then enter.

**2. Shrinking with `if` instead of `while`.** One kick is not always enough — the duplicate may be deep inside. "Shrink *until* clean" is a `while` by definition.

**3. Forgetting the `+1` in the size.** `right - left` counts the gaps between posts, not the posts. Size is `right - left + 1`.

**4. Kicking and moving in the wrong order.** `left += 1` before `seen.remove(s[left])` removes the wrong letter. The wall letter goes first; then the wall moves.

**5. Measuring dirty windows.** Updating `best` inside or before the shrink. Only clean windows count — measure after the invariant is restored.

**6. Resetting `left`.** Any code that moves `left` backward has left the pattern — and usually the O(n) guarantee too.

---

## The phrase to never forget

> **"Grow right while clean. Shrink left when dirty."**

If you remember that phrase, you can rebuild the code from scratch — the loops, the conditions, the order of lines all fall out of it.

---

## Practice ladder

Do these in order. Each one reuses the last one's muscle and adds one small twist.

1. **LC 3 — Longest Substring Without Repeating Characters.** The one above. Redo it from memory tomorrow, from the phrase alone.
2. **LC 643 — Maximum Average Subarray I.** Fixed-size window. Even simpler: the band never changes size. Trains "adjust, don't recompute."
3. **LC 209 — Minimum Size Subarray Sum.** Flips the goal: shrink while the window is *good*, to find the smallest good window. Trains reading the `while` condition carefully.
4. **LC 424 — Longest Repeating Character Replacement.** The guest list becomes a letter counter. "Dirty" becomes a small formula.
5. **LC 567 — Permutation in String.** Fixed window + a counter. Two ideas combined.
6. **LC 1004 — Max Consecutive Ones III.** State is just one number: zeros inside the window.
7. **LC 76 — Minimum Window Substring.** The boss fight. Rated Hard, but it is still only Rule 1 and Rule 2 — with a fancier definition of "clean."

When the ladder is done, close everything and re-derive LC 3 from the phrase. If you can, the pattern is yours.

---

*Previously in this series: Two Pointers, learned through LeetCode 151 (Reverse Words in a String). Next up: Fast & Slow Pointers — two runners on a track.*
