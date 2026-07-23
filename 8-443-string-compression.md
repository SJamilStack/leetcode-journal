# LC 443 — String Compression

**Link:** https://leetcode.com/problems/string-compression/
**Difficulty:** Medium
**Pattern:** Two Pointers (reader/writer)

---

## What I tried first

**Attempt 1 — dict + separate string.**

```python
class Solution:
    def compress(self, chars: List[str]) -> int:
        
        d = dict()
        s = ''
        for c in chars:
            if c not in d:
                d[c] = 0
            d[c] = d[c] + 1

        for key, value in d.items():
            s += key
            if value > 1:
                s += str(value)
        print(s)
        print(len(s))
        print(list(s))
        return len(s)
```

This counted each character using a dict, then built a separate string `s` from the counts and returned its length. It got the compressed *content* right, but `chars` itself was never touched — I was returning a length that didn't match what was actually sitting in `chars`.

**Attempt 2 — two pointers, but counting was wrong and still writing into a new list.**

```python
class Solution:
    def compress(self, chars: List[str]) -> int:
    
        s = []
        left = 0
        right = 1
        while left < len(chars):
            s.append(chars[left])
            while right < len(chars) and chars[right] == chars[left]:
                s.append(chars[left])            
                right += 1
            
            left = right
            right = left + 1
            print(s)
            print(len(s))

        print(s)
```

---

## Mistakes I made

**Mistake 1 — the dict approach loses order.**

Keying by character silently assumes all instances of a letter belong to the same group. That's true for `"aabbccc"`, but breaks on something like `"aabaa"` — the dict would merge both groups of `a` into one, when they should stay separate since a `b` sits between them.

**Mistake 2 — building a separate string `s`, and only returning its length.**

The problem statement describes the algorithm using a variable called `s` — but that's just prose explaining the *logic*, not an instruction to keep a separate string in the code. The very next sentence in the problem says the compressed result should **not** be returned separately, but stored directly in `chars`. My code did the opposite: it built `s` on the side and only handed back `len(s)`, while `chars` itself stayed completely untouched. LeetCode grades this by looking at `chars[:returned_length]` — so my output showed the *original*, uncompressed `chars`, just cut off at the right length.

**Mistake 3 — in attempt 2, the inner loop appended instead of counting.**

```python
while right < len(chars) and chars[right] == chars[left]:
    s.append(chars[left])            
    right += 1
```

Every time a match was found, I appended the letter *again* — so this was just copying `chars` into `s`, not compressing it. I needed to count how many matches there were, and only write the letter once (plus the count, if more than one) after the counting was done.

**Mistake 4 — IndexError from checking contents before checking bounds.**

```python
if chars[left] == chars[right]:
```

`right` could walk past the end of the array on the last group (nothing left to compare against), and the code tried to read `chars[right]` before confirming `right` was still a valid index. Fix: always check the bound first, so Python's `and` short-circuits and never touches a box that doesn't exist:

```python
while right < len(chars) and chars[right] == chars[left]:
```

**Mistake 5 — the real block: not seeing how to write into `chars` itself.**

Even once the counting logic worked with `s`, I was stuck on how to translate "store it in `chars`" into actual code. The unlock was realizing: **`s.append(x)` and `chars[write] = x; write += 1` do the same job** — one grows a new list, the other writes into a box you already have, using a second pointer (`write`) to track how far you've filled in. Every `s.append(...)` in my head became `chars[write] = ...; write += 1` once I saw that mapping directly, one line at a time.

I was also confused about what happens to leftover boxes in `chars` after the compressed content is shorter than the original — e.g. `chars = ['a','3','b','2','b']` after compressing `"aaabb"`, where the last `'b'` is untouched junk. The array itself can't shrink, so that leftover data just stays there. The line in the problem statement — *"the characters in the array beyond the returned length do not matter and should be ignored"* — exists specifically for this. The returned length (`write`) is what tells the grader where the real answer stops; everything after that is ignored, not part of the answer.

---

## Final solution, with explanation

```python
class Solution:
    def compress(self, chars: List[str]) -> int:
        write = 0
        left = 0

        while left < len(chars):
            char = chars[left]
            right = left
            count = 0
            while right < len(chars) and chars[right] == char:
                right += 1
                count += 1

            chars[write] = char
            write += 1

            if count > 1:
                for digit in str(count):
                    chars[write] = digit
                    write += 1

            left = right

        return write
```

**Why it works:**

Two pointers do two different jobs. `left` (with a helper, `right`) reads through `chars` and measures how long the current group of repeated characters is. `write` marks where the next piece of the compressed answer goes — always at or behind wherever `left`/`right` have already read, so writing never overwrites data that hasn't been processed yet.

For each group:
1. `right` scans forward counting matches, starting from `left`.
2. Once the group ends, write the character once into `chars[write]`, and advance `write`.
3. If the group had more than one character, write each digit of the count into `chars`, one box per digit, advancing `write` each time.
4. Jump `left` to wherever `right` stopped, and repeat.

There is no separate string anywhere in this version — every letter and digit that would have gone into a string `s` gets written directly into `chars` instead. That's what "store it in `chars`" means: the array becomes the answer, in place, rather than a new structure being built and copied over.

**Why the return value matters:** the array can't physically shrink, so anything after the compressed content (like a leftover original character) just stays sitting in `chars`, unchanged. Returning `write` tells the grader exactly where the real answer ends — `chars[:write]` is the compressed result; anything past that is explicitly meant to be ignored.

**Complexity:**
- Time: O(n) — `right` (and therefore `left`) visits each character exactly once across the whole run.
- Space: O(1) extra — only a handful of single-value pointers (`write`, `left`, `right`, `count`, `char`), no second array or dict.

**The one-line takeaway:** *"One finger counts a group, one finger writes it down — and the writing finger never needs to know where it is, only where it left off."*
