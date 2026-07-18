---
title: "Fast & Slow Pointers, Explained So Simply You Can Never Forget It"
published: false
description: "The tortoise-and-hare pattern taught as a story: if the track loops, the fast runner laps the slow one. With room-by-room traces, code derived sentence by sentence, the gap proof, and a practice ladder."
tags: algorithms, python, interview, leetcode
---

*This is part of a series where I learn DS & Algo patterns as plain-English stories first, and only then turn the stories into code. The goal is muscle memory — being able to rebuild the code from the story, any time, from scratch. Previously: Two Pointers and Sliding Window.*

---

## The story in one sentence

> **Two runners start together on a track. Slow takes 1 step per turn. Fast takes 2. If the track loops, Fast comes around from behind and catches Slow. If the track ends, Fast falls off the edge first.**

That is the entire pattern — often called *tortoise and hare*, or Floyd's cycle detection. Everything below unpacks these words.

---

## What problem are we solving?

We will learn the pattern on one concrete problem: **LeetCode 141 — Linked List Cycle.**

In plain words: you are given a chain of nodes. Each node has an arrow to the next one. Somewhere, an arrow might point *backward* to an earlier node — making a loop. Tell me: does this chain loop, or does it end?

---

## Why this is even hard

With an array, you would just check "have I been to this box before?" But a chain gives you no box numbers. You can only follow arrows. And if there is a loop, following arrows never ends — you would walk forever without knowing you are going in circles.

The obvious fix: keep a guest list (a set) of nodes you have visited. Walk the chain; if you ever see a node twice, there is a loop. That works! But it costs O(n) extra memory. And interviewers always follow up:

> *"Can you do it with no extra memory?"*

That follow-up is what this pattern answers.

---

## The setup: rooms and doors

A linked list is a chain of rooms. Each room has one door leading to the next room. In code, `node.next` means "walk through the door."

Two fingers again — but this time they point at *rooms*, not box numbers:

- `slow` — walks one door per turn.
- `fast` — walks two doors per turn.

Both start in the first room (`head`).

Two possible endings, and each one answers the question:

- The runners meet again in the same room → **loop**.
- Fast falls off the end of the chain → **no loop**.

---

## Build the code, sentence by sentence

Do not write code first. Write the story as sentences. Then let each sentence force the code into place.

**Sentence 1: "Both runners start at the beginning."**

```python
slow = head
fast = head
```

**Sentence 2: "Keep running while Fast still has road ahead."**

Fast takes two steps per turn. Two steps are only safe if the current room exists *and* the next room exists. Flip the stop condition into "keep going while":

```python
while fast and fast.next:
```

Note the order. Same rule as every bounds check: **check the room exists before opening its door.** `fast` first, then `fast.next`. Python reads `and` left to right and stops early (short-circuit). Swap the order and you knock on a door in a room that is not there — a `None` crash.

**Sentence 3: "Slow steps once. Fast steps twice."**

```python
    slow = slow.next
    fast = fast.next.next
```

**Sentence 4: "If they are in the same room — Fast lapped Slow. Loop found."**

```python
    if slow is fast:
        return True
```

`is`, not `==`. We are asking "same *room*?", not "rooms that look alike?" That is identity, not equality. Two different nodes can hold the same value; they are still different rooms.

**Sentence 5: "Fast fell off the edge. No loop."**

The `while` condition failing *is* this sentence — no extra check needed:

```python
return False
```

Put together:

```python
def hasCycle(head):
    slow = head
    fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:
            return True
    return False
```

Notice *where* each line sits, and why it could sit nowhere else:

- The meet check comes **after** both runners move. They start in the same room — checking before moving would report a loop on turn zero of every list.
- The `return False` lives **after** the loop, because "Fast ran out of road" is exactly what makes the `while` condition fail. The condition does double duty: it is both the safety check and the no-loop verdict.

---

## Full trace, both endings

**A chain with a loop:** `A → B → C → D → B` (D's arrow points back to B).

- **Start:** both runners in room A.
- **Turn 1:** slow → B. fast → C. Different rooms.
- **Turn 2:** slow → C. fast walks D, then through D's door back to B. Different rooms — and notice fast is now *behind* slow. It lapped the loop.
- **Turn 3:** slow → D. fast walks C, then D. **Same room. Loop found.** Return `True`.

**A straight chain:** `A → B → C → D → end`.

- **Turn 1:** slow → B, fast → C.
- **Turn 2:** slow → C. fast walks to D, then through D's door to... nothing. `fast` becomes `None`.
- **Top of loop:** is `fast` a room? No. Exit. Return `False`.

---

## "Why must they meet? Couldn't Fast jump *over* Slow?"

This is the question everyone asks, and it deserves a real answer, not hand-waving.

Once both runners are inside the loop, stop thinking about their positions. Think about the **gap** — how many rooms Fast is *behind* Slow, measured around the circle.

Each turn: Slow moves 1, Fast moves 2. So the gap shrinks by exactly **1 room per turn**.

A number that shrinks by exactly 1 each turn must pass through ...3, 2, 1, **0**. It cannot skip from 1 to −1, because it changes by exactly one. Gap 0 means same room. Caught.

That is the whole proof: **relative speed is 1, so jumping over is impossible.**

This also explains why speeds 1 and 2 are the standard pick. What matters is the *relative* speed. Speeds 1 and 2 give relative speed 1 — the catch is guaranteed with no arithmetic. Speed 3 would shrink the gap by 2 per turn, and a gap of 1 would jump to −1: a miss. Why risk it when 1 is bulletproof?

**Cost:** Fast reaches the loop in at most n steps, then closes a gap of at most n rooms in at most n turns. Total **O(n) time**. And the space? Two pointers. **O(1)** — no guest list. That is the entire point of the pattern.

---

## The bonus move: finding the middle

Same two runners, one new observation:

> **When Fast has gone twice as far, and Fast is at the end — Slow is at the middle.**

```python
def middleNode(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    return slow
```

That is LeetCode 876 in five lines. Same skeleton — only the *ending* is read differently. Instead of watching for a meeting, we let the loop finish and look at where Slow stands.

This "one runner at double speed" idea keeps paying: middle of a list, cycle detection, finding a cycle's *start* (LC 142), splitting a list in half for merge sort, even detecting loops in sequences of *numbers* where there is no list at all.

---

## How to think your way to this code

Same ritual as every pattern in this series. Three questions before any code.

**Question 1 — What is the promise?**

> "After every turn, Fast is exactly twice as far from the start as Slow."

That invariant is what makes both endings readable: it is *why* meeting means a lap happened, and *why* Fast at the end means Slow is at the middle.

**Question 2 — What must I ask each turn?**

Two questions only: *"can Fast take two steps safely?"* and *"are they in the same room?"* Neither question needs a container. No membership test, no counting, no ordering. → **No data structure at all.** That is how you *derive* O(1) space instead of memorizing it.

Compare with the set solution: its question is "have I seen this room before?" — a membership test → set → O(n) space. Different question, different structure, different cost. The pattern is not "sets are bad." The pattern is: **change the question you ask each turn, and the data structure (and its cost) changes with it.**

**Question 3 — What order keeps the promise?**

Move both runners, *then* check. They start together; checking before moving reports a false loop instantly. And within the `while` condition: existence before access — `fast` before `fast.next`. Read before you write; check before you touch. Same rule, every pattern.

---

## Common mistakes (I made half of these)

**1. Checking before moving.** `if slow is fast` before the steps → both start at `head` → instant false `True` on every list. Move first, then check.

**2. Wrong or missing safety check.** `while fast.next` alone crashes when `fast` is `None`. `while slow and slow.next` guards the wrong runner — Slow never falls off first. The runner taking two steps is the one that needs two guards: `while fast and fast.next`.

**3. Guards in the wrong order.** `while fast.next and fast` evaluates `fast.next` first — crash when `fast` is `None`. Existence check always goes first; short-circuit only saves you left to right.

**4. Using `==` instead of `is`.** Usually works by accident, but it asks the wrong question. You want "same room," not "same-looking room." Say what you mean: `is`.

**5. Equal speeds.** Both at speed 1 (or both at 2) → the gap never shrinks → they never meet inside a loop → infinite walk. The *difference* in speeds is the whole engine.

**6. Reaching for a set by reflex.** Not wrong — it passes. But it spends O(n) memory to answer a question the two runners answer for free. In interviews, offer the set as the warm-up and the runners as the answer to "can you do better?"

---

## The phrase to never forget

> **"If the track loops, the fast runner laps the slow one."**

If you remember that phrase, you can rebuild the code from scratch — the two speeds, the safety check, the meet test, and both endings all fall out of it.

---

## Practice ladder

Do these in order. Each one reuses the last one's muscle and adds one small twist.

1. **LC 876 — Middle of the Linked List.** The gentlest start. Five lines. Trains the skeleton with the friendlier ending.
2. **LC 141 — Linked List Cycle.** The one above. Redo it from the phrase tomorrow.
3. **LC 142 — Linked List Cycle II.** Find where the loop *starts*. Uses a gorgeous trick: after the runners meet, send one runner back to the start, then walk both at speed 1 — the room where they meet again is the loop's entrance. Try to figure out *why* before reading the proof. (Hint: write down the distances.)
4. **LC 202 — Happy Number.** The mind-bender: there is no linked list at all. The "next room" is a math operation on a number. Same runners, same code shape. This is the problem that proves you understand the pattern, not the syntax.
5. **LC 234 — Palindrome Linked List.** Combines fast & slow (find the middle) with linked-list reversal — a preview of a later pattern in this series.
6. **LC 287 — Find the Duplicate Number.** The boss fight. An *array* is secretly a linked list (`i → nums[i]`), and the duplicate value is a cycle entrance. When this clicks, you will start seeing chains hiding everywhere.

When the ladder is done, close everything and re-derive LC 141 from the phrase alone. If you can, the pattern is yours.

---

*Previously in this series: Two Pointers (LeetCode 151) and Sliding Window (LeetCode 3). Next up: Prefix Sums — a running total that turns "sum of any chunk" into one subtraction.*
