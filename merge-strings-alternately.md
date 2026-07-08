## 1. Problem Info
- **Name:** Merge Strings Alternately
- **Link:** https://leetcode.com/problems/merge-strings-alternately/
- **Difficulty:** Easy
- **Date solved:** 2026-07-08
- **Pattern/Technique:** Two-pointer / index iteration, string building

---

## 2. What I Tried First
I compared the lengths of `word1` and `word2` up front, figured out which one
was longer, sliced off its "extra" tail, then looped up to the shorter length
building the merged string with `+`.

```python
class Solution:
    def mergeAlternately(self, word1: str, word2: str) -> str:
        merged = ''
        word1_len = len(word1)
        word2_len = len(word2)
        extra_characters = ''
        range_number = word1_len
        if word1_len > word2_len:
            extra_characters = word1[len(word2):len(word1)]
            range_number = word2_len
        elif word1_len < word2_len:
            extra_characters = word2[len(word1):len(word2)]
            range_number = word1_len
        for i in range(range_number):
            merged = merged + word1[i] + word2[i]
        return merged + extra_characters
```

This passed. But "passed" and "well-written" aren't the same thing — see below.

---

## 3. Mistakes I Made (the most important section)

**Mistake 1 — building a string with `+=` inside a loop.**
I wrote `merged = merged + word1[i] + word2[i]` inside the `for` loop. I didn't
realize this was a problem because the code *worked*. The issue is invisible
until you think about *how* Python strings behave.

- Strings in Python are **immutable** — once created, they can't be changed.
- So `merged + word1[i] + word2[i]` doesn't add two characters onto the
  existing string in place. It creates a **whole new string**, copies every
  character of the old `merged` into it, and *then* adds the two new ones.
- That means iteration 50 doesn't just do "2 characters of work" — it copies
  all 98 characters built so far, plus adds 2 more.
- Add that up across the whole loop and the total work grows like
  `2 + 4 + 6 + ... + N ≈ N²/2` — quadratic, not linear, even though I only
  looped `n` times.

**Why I didn't notice:** with strings capped at length 100 (per the problem's
constraints), N² work is still only ~20,000 character copies — invisible in
milliseconds. This is the trap: **small test cases hide bad time complexity.**
The lesson only shows up if I imagine the same code running on 10-million
character strings, where O(n²) would take way too long while O(n) stays fast.

**The fix:** don't concatenate strings in a loop. Append pieces to a `list`
instead, then join once at the very end:

```python
parts = []
parts.append(char)   # amortized O(1) — no full copy each time
result = ''.join(parts)  # one single O(n) pass at the end
```

`list.append()` doesn't have this problem because Python lists over-allocate
space in advance, so most appends don't trigger a resize/copy. `''.join()`
then does exactly one pass over everything to build the final string.

**Mistake 2 — left debug `print()` statements in the "final" code.**
Small thing, but it's a habit to catch before calling something done: strip
debug output, or make it conditional on a `DEBUG` flag.

**Takeaway to remember for next time:** *whenever I'm accumulating a string
piece-by-piece in a loop, that's my cue to reach for a list + join instead of
`+=`.* This isn't specific to this problem — it applies anywhere I'm building
up a string incrementally.

---

## 4. Final Solution — and *why* it's the better approach

```python
class Solution:
    def mergeAlternately(self, word1: str, word2: str) -> str:
        result = []
        i, j = 0, 0
        n, m = len(word1), len(word2)

        while i < n or j < m:
            if i < n:
                result.append(word1[i])
                i += 1
            if j < m:
                result.append(word2[j])
                j += 1

        return "".join(result)
```

**Walking through why this works well, not just that it works:**

- **Two pointers (`i`, `j`) instead of one shared index.** My original
  solution used a single loop variable and had to precompute "how far can I
  safely go before one string runs out" (the `range_number` logic). The
  two-pointer version sidesteps that entirely — each pointer just stops
  advancing on its own once its string is exhausted (`if i < n`, `if j < m`),
  so there's no upfront math needed to figure out the shorter length or slice
  off a remainder.
- **The loop condition `while i < n or j < m`** naturally keeps going until
  *both* strings are exhausted — this is what handles "append the leftover
  tail" for free, without a separate `extra_characters` variable or slicing
  step. Whichever string still has characters left just keeps getting
  appended after the other one stops contributing.
- **Building with a list + one `join()`** avoids the quadratic copying problem
  from Mistake 1. Every `append` is cheap, and the single `join()` at the end
  is one clean O(n) pass.
- **Net result:** O(n + m) time, O(n + m) space — both optimal, since every
  character in the output must be touched at least once no matter what
  approach you use. There's no way to do better than linear here, and this
  solution doesn't do any unnecessary extra work to get there.

**A more Pythonic alternative, same complexity, worth knowing but not
necessarily "the" way to write it in an interview:**

```python
from itertools import zip_longest

class Solution:
    def mergeAlternately(self, word1: str, word2: str) -> str:
        return "".join(a + b for a, b in zip_longest(word1, word2, fillvalue=""))
```

`zip_longest` pairs up characters from both strings and fills in `""` once
one string runs out, which is a built-in way of getting the same "append the
leftover tail" behavior as the `while` loop above.

---

## 5. Similar Problems
- (add links here as I find more string-interleaving / two-pointer problems)

---

## 6. Review Flag
- [ ] Revisit in 1 week
- [ ] Revisit in 1 month
- [x] Understood the concept — but the "list + join, not string +=" habit is
      the one thing I want to make automatic going forward
