# Interview Playbook: Two Pointers

*One-page battle card. The phrase: **"Walk backwards, skip spaces, grab whole words."***

---

## Recognize it (10 seconds)

- In-place work on a string/array; "without extra space"; "without split()/built-ins"
- Sorted array + pair with a property (sum, difference) → converging pointers
- Palindrome checks; comparing from both ends
- "Remove / move elements in place, keep order" → reader/writer pointers
- Words/tokens separated by junk you must skip

## Your opening lines

> "This is a two-pointer scan. The one-liner is `' '.join(reversed(s.split()))` — but let me do it manually, since that's presumably the point."

Always name the built-in solution first. It shows you know the language; solving without it shows you know the algorithm. Then:

> "I'll walk from the right. Skip spaces. When I land on a letter, that's the *end* of a word — a second pointer finds its start, I grab the slice, and jump past it."

## Narration script while coding

- **The promise:** "My invariant: at the top of every turn, `i` points at the next unexamined position."
- **The junk case:** "Space → step left and `continue`, so everything below can assume I'm on a letter."
- **The inner condition:** "Keep going while in bounds AND on a letter — bounds check first, so I never index off the edge." (`while j >= 0 and s[j] != ' '`)
- **The reset:** "I set `i = j`, not `i -= 1` — the teleport restores the invariant; decrementing would land inside the word I just consumed."

## Complexity talking points

> "It's a while inside a while, but it's O(n): both pointers only move left and never reset, so total movement across the whole run is bounded by n. That's an amortized argument — count pointer movement, not loop levels."

Say this *unprompted*. It's the highest-value sentence in the whole problem.

## Expected follow-ups

- **"Do it in place"** → possible in languages with mutable strings (C++/char array): reverse the whole array, then reverse each word. Python strings are immutable, so O(n) output space is unavoidable — say so.
- **"What about unicode / tabs?"** → the skip test generalizes: replace `== ' '` with `isspace()`.
- **"Why not split?"** → "split() is doing exactly this scan internally; the interview version shows I can write it."

## Pressure traps

1. `i -= 1` instead of `i = j` → re-harvests the same word ("bye", "by", "b").
2. Flipped guard order → `s[-1]` silently wraps around in Python. Bounds first.
3. Missing `continue` → word code runs while standing on a space.
4. Slice off-by-one → it's `s[j+1 : i+1]`; j overshot by one *on purpose*.

## Closing move

> "Passes on messy spacing because I only grab letters and insert my own separators. If you want the in-place variant, I'd do reverse-all then reverse-each-word on a char array."
