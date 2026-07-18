---
title: "Two Pointers, Explained So Simply You Can Never Forget It"
published: false
description: "The two-pointer pattern taught as a story: walk backwards, skip spaces, grab whole words. With box-by-box traces, code derived sentence by sentence, the amortized O(n) proof, and a practice ladder."
tags: algorithms, python, interview, leetcode
---

*This is the first post in a series where I learn DS & Algo patterns as plain-English stories first, and only then turn the stories into code. The goal is muscle memory — being able to rebuild the code from the story, any time, from scratch.*

---

## The story in one sentence

> **Walk backwards through the string. Skip the blanks. Every time you land on a letter, fence in the whole word with a second finger and grab it. Walking backwards makes the reversal free.**

That is the entire pattern. Everything below unpacks these words.

---

## What problem are we solving?

We will learn the pattern on one concrete problem: **LeetCode 151 — Reverse Words in a String.**

In plain words: given a string with words and messy spaces, return the words in reverse order, with single spaces, no spaces at the ends.

Example: `"  hi  bye "` becomes `"bye hi"`.

Honest note first: in Python, this problem has a one-liner —

```python
return ' '.join(reversed(s.split()))
```

`split()` with no arguments already eats all the messy spaces. Know this version; interviewers expect it. But they always follow up: *"now without `split()`."* That follow-up is where the pattern lives.

---

## The setup: boxes and one finger

A string is just characters in numbered boxes:

```
s =  "  hi  bye "
      0123456789
```

A **pointer** is nothing magical. It is just a number that says which box you are looking at. Call it `i`.

We start at the **last box**: `i = len(s) - 1`. And here is the key insight of the whole problem:

> **Walking right-to-left means words fall into your list already reversed. No double-reversal needed. The direction of the walk *is* the reversal.**

---

## The rules of the game

Two rules, alternating forever:

**Rule 1 — Skip the air.** If the current box holds a space, move one box left and re-ask. Spaces are nothing; walk past them.

**Rule 2 — Fence and grab.** If the current box holds a letter, you are standing on the **last letter of a word**. Send a second finger, `j`, walking left until it steps *off* the word. Now the word is trapped between the fingers. Grab it, then teleport `i` to where `j` stopped, and go back to Rule 1.

The promise the loop keeps — the **loop invariant**: *at the top of every turn, `i` points at the next unexamined box.* Every line of code exists to keep that promise.

---

## Build the code, sentence by sentence

Do not write code first. Write the story as sentences. Then let each sentence force the code into place.

**Sentence 1: "Start at the last box, and keep going until I fall off the front."**

That sentence *is* the outer loop. "Fall off the front" means `i` goes below 0 — so the loop runs while it hasn't:

```python
words = []
i = len(s) - 1
while i >= 0:
```

**Sentence 2: "If I'm standing on a space, move left and re-ask."**

"Re-ask" means: jump back to the top of the loop. That is exactly what `continue` does:

```python
    if s[i] == ' ':
        i -= 1
        continue
```

Why `continue` and not just falling through? Because everything *below* this point assumes we are standing on a letter. If we are on a space, that code must not run. Rule to keep: **`continue` handles the junk case — tiny cleanup, restart, so everything below can safely assume the good case.**

**Sentence 3: "Send a second finger left until it steps off the word."**

"Send a second finger" from where I stand → `j = i`. Now the condition — and here is a rule worth tattooing somewhere: **a `while` condition describes when to KEEP GOING, not when to stop.** The story says "stop when you hit a space or fall off the front." Flip it: "keep going while it's a letter and I'm still on the string":

```python
    j = i
    while j >= 0 and s[j] != ' ':
        j -= 1
```

Two guards, one for each way a word can end:

- `j >= 0` — "I haven't fallen off the front wall." A word touching the start of the string has no space before it; this guard catches that case.
- `s[j] != ' '` — "I haven't stepped onto a space."

And the **order matters**. Python reads `and` left to right and stops early (short-circuit). With `j >= 0` first, the moment `j` hits −1, the check ends before `s[j]` is ever touched. Flipped, Python would peek at `s[-1]` — which in Python doesn't crash, it *wraps around* to the last character. A letter. The finger keeps walking backwards through the wrong end of the string. Silent garbage, no error. Rule: **the "am I in bounds?" check goes first, so the "what's in the box?" check never runs on a box that doesn't exist.**

When this loop ends, `j` has overshot by one, on purpose — it stands on the space *before* the word, or at −1.

**Sentence 4: "Grab the word between the fingers."**

The word starts one box right of where `j` stopped, and ends at `i`:

```python
    words.append(s[j+1 : i+1])
```

(The `+1` on the end is Python slices excluding the end box. And the `j+1` works whether `j` stopped on a space or fell off the edge at −1 — `s[0 : i+1]` grabs a word touching the front wall perfectly.)

**Sentence 5: "Pick up the walk from where the second finger stopped."**

```python
    i = j
```

This is the **teleport**, and where it goes is forced by the invariant: the top of the loop assumes `i` points at the next unexamined box. Everything right of `j` is consumed — the word we grabbed, the spaces we skipped. The next unknown box is exactly where `j` stands. Rule: **resets restore the loop's opening assumption. Ask what the top of the loop believes, and place the update so it's true when you get there.**

Put together:

```python
def reverseWords(s: str) -> str:
    words = []
    i = len(s) - 1                      # start at the last box
    while i >= 0:                       # until I fall off the front
        if s[i] == ' ':                 # standing on a space?
            i -= 1                      #   step left
            continue                    #   and re-ask
        j = i                           # second finger starts where I stand
        while j >= 0 and s[j] != ' ':   # keep going WHILE still on the word
            j -= 1
        words.append(s[j+1 : i+1])      # grab between the fingers
        i = j                           # teleport: restore the invariant
    return ' '.join(words)
```

The messy spacing of the input never survives, because we only ever grab letters and insert our own clean single spaces at the end.

---

## Full trace on `"  hi  bye "`

```
s =  "  hi  bye "
      0123456789
```

**Turn 1.** Box 9: space → `i = 8`, `continue`. Notice what got skipped: the `j` code, the append, the teleport. That's the `continue` doing its job.

**Turn 2.** Box 8: `'e'` — letter, fall through. `j` starts at 8 and walks: `'e'` letter → 7, `'y'` letter → 6, `'b'` letter → 5, `s[5]` is a space → **stop**. 

```
      "  hi  bye "
       0123456789
            ↑  ↑
            j  i
```

Grab `s[6:9]` → `"bye"`. List: `["bye"]`. Teleport: `i = 5`.

Pause here on the teleport, because this is where the wrong version breaks visibly. If the reset were `i -= 1` instead, `i` would land on box 7 — the `'y'` *inside the word we just took* — and next turn we'd grab `"by"`, then `"b"`. The story says "teleport past the whole word," so the code must too.

**Turns 3–4.** Boxes 5 and 4: spaces → skip, skip. The janitor eating the space clump, one box per turn.

**Turn 5.** Box 3: `'i'` — letter. `j` walks: 3 → 2 (`'h'`) → 1 (space, stop). Grab `s[2:4]` → `"hi"`. List: `["bye", "hi"]`. Teleport: `i = 1`.

**Turns 6–7.** Boxes 1 and 0: spaces → skip, skip. `i` becomes −1.

**Turn 8.** Top of loop: is −1 ≥ 0? No. Exit. "Falling off the front" needed no extra check — the loop condition *is* the check.

Join: `"bye hi"`. Done.

---

## "It's a while inside a while — isn't that O(n²)?"

This question deserves its own section, because the answer rewires how you read nested loops.

The "nested loops = n²" rule is a *heuristic*, not a law. It holds when the inner loop can redo up to n work for **each** turn of the outer loop. That is not what happens here.

Look at what the fingers actually do across the whole run: both `i` and `j` only ever move left. Neither ever backs up. And the teleport (`i = j`) means the boxes `j` walked over are **never visited again by anyone** — `i` doesn't resume from where it was; it jumps past them.

Think of it as one relay walk across the string. Sometimes `i` carries the baton (skipping spaces), sometimes it hands off to `j` (walking through a word), then `j` hands back. The baton only travels leftward, and the total distance covered — by both fingers combined — is the length of the string. Each box gets looked at a small constant number of times.

Count it on our example: 10 boxes, about 12 total looks. Not 10 × 10 = 100.

This style of counting is **amortized analysis** — averaging cost over the whole run instead of per turn. The rule of thumb: **don't count loop levels — count total pointer movement.** If the inner loop's progress is never undone, all the loops share one budget of n steps.

(A real O(n²) version: replace the teleport with `i -= 1`, so `j` re-scans the same word again and again. The teleport is what kills the quadratic.)

---

## How to think your way to this code

Same ritual every time. Three questions before any code.

**Question 1 — What is the promise?**

> "At the top of every turn, `i` points at the next unexamined box."

**Question 2 — What must I ask each turn?**

Just one question: *"is this box a space or a letter?"* No membership test, no counting → **no data structure needed** beyond the output list. (Compare: sliding window's question "is X inside?" demands a set. The question picks the structure — here, the question is so simple it picks nothing.)

**Question 3 — What order keeps the promise?**

Three placement rules fall out, and they transfer to every pointer problem:

1. **`continue` handles the junk case** — cleanup, restart, so the code below assumes the good case.
2. **Write `while` conditions as "keep going while..."** — flip the story's stop condition.
3. **Resets restore the loop's opening assumption** — the teleport goes last, and it goes to `j`, because that is what makes the promise true again.

Next time you are stuck on structure: don't write code. Write the story as numbered English sentences. The loops, conditions, and resets are already hiding in the grammar.

---

## Common mistakes (I made half of these)

**1. Reversing twice.** My first solution reversed the whole string, then un-reversed each word. It passed — but reversing something twice is a smell. It usually means there is a direction of traversal that avoids both reversals. Here: walk backwards, and reversal is free.

**2. `i -= 1` instead of `i = j`.** The reset lands *inside* the word you just grabbed, and you harvest `"by"`, `"b"`... The reset must restore the invariant, not just "move a bit."

**3. Flipped guard order.** `s[j] != ' ' and j >= 0` peeks at `s[-1]`, which wraps around in Python instead of crashing. Silent wrong answers. Bounds check first, always.

**4. Missing the `j >= 0` guard entirely.** Works until a word touches the front of the string — then the walk runs off the edge. The two guards exist because a word can end two ways: at a space, or at the wall.

**5. Forgetting `continue`.** The space case falls through into the word-grabbing code while standing on a space. Garbage grabs.

**6. Off-by-one in the slice.** `s[j:i]` or `s[j+1:i]` instead of `s[j+1:i+1]`. Remember: `j` overshot by one on purpose, and slices exclude the end.

---

## The phrase to never forget

> **"Walk backwards, skip spaces, grab whole words."**

If you remember that phrase, you can rebuild the code from scratch — the loop, the janitor `if`, the fence, and the teleport all fall out of it.

---

## Practice ladder

Do these in order. Each one reuses the last one's muscle and adds one small twist.

**Same pattern almost verbatim:**

1. **LC 58 — Length of Last Word.** A mini version of this exact code: walk from the right, skip trailing spaces, count one word. Five minutes.
2. **LC 186 — Reverse Words in a String II.** In-place on a char array. Forces the word-fencing move with no `split()` crutch.
3. **LC 557 — Reverse Words in a String III.** Reverse each word, keep word order. Same fencing scan, opposite goal.

**The skip-the-junk muscle:**

4. **LC 125 — Valid Palindrome.** Pointers from both ends walking inward, skipping non-letters. The "if junk, move and continue" rhythm on both sides.
5. **LC 844 — Backspace String Compare.** Walk from the right, skipping characters eaten by backspaces.

**The one-way-pointers muscle:**

6. **LC 26 — Remove Duplicates from Sorted Array** and **LC 283 — Move Zeroes.** Reader/writer pointers — a different costume for "each pointer moves one way, never resets."
7. **LC 11 — Container With Most Water.** Converging pointers with a real decision: which one moves, and why.

**Boss fight:**

8. **LC 42 — Trapping Rain Water** (two-pointer solution). Converging pointers plus carried state. When this feels natural, the pattern is fully yours.

When the ladder is done, close everything and re-derive LC 151 from the phrase alone. If you can, the pattern is yours.

---

*Next in this series: Sliding Window — a two-pointer technique where the chunk between the pointers carries state. Grow right while clean, shrink left when dirty.*
