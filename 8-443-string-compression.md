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

**Mistake 6 — writing the letter but forgetting to move `write`, then patching it with `write = read`.**

```python
chars[write] = char
if count > 1:
    ...
    write += 1   # only happened inside the count>1 branch
print('chars', chars)
write = read     # tried to "fix" the missing increment this way
```

Writing a character into `chars[write]` should always be immediately followed by `write += 1` — one box filled, one step forward. I only incremented `write` sometimes (inside the digit-writing branch), then patched the gap by setting `write = read` at the end of each group. That's a different quantity entirely: `read` counts how far I've scanned through the *original* array, `write` should count how many boxes of the *compressed* answer I've actually filled. Those only match by coincidence when a group's compressed size equals its original size (size-1 and size-2 groups happened to line up, which is why early tests passed) — they diverge badly for anything bigger. A 12-character group compressed to 3 characters returned length 13 instead of 4, because `write = read` silently overwrote the correct value with the wrong one.

Fix: never assign `write` directly. Only ever `write += 1`, exactly once per character actually written (letter or digit), and let it accumulate naturally.

**Mistake 7 — comparing against a live box instead of a frozen snapshot.**

```python
while read < len(chars) and chars[read] == chars[write]:
```

Once `write` starts falling behind `read` — which happens the moment any group actually compresses — `chars[write]` stops meaning "the character I'm currently grouping" and starts meaning "whatever stale, unrelated data happens to still be sitting in that box." On `"cccb"`, after compressing `ccc` to `c3`, `write` lands on box 2, which still holds a leftover `'c'` from the original array. Comparing the next character (`'b'`) against that stale `'c'` gave a wrong `count`, and the group boundary was missed entirely.

The fix: compare against `char`, a variable captured once at the top of the group and never written to again — a frozen snapshot, not a live box:

```python
char = chars[read]
...
while read < len(chars) and chars[read] == char:
```

`char` can't go stale because nothing overwrites it. `chars[write]` can, because it's the exact box being actively rewritten throughout the run. General lesson: when reading and writing share the same array, never use a box you're overwriting as your comparison reference — snapshot the value into its own variable first.

---

## Final solution, with explanation

```python
class Solution:
    def compress(self, chars: List[str]) -> int:
        write = 0
        read = 0
        while read < len(chars):
            char = chars[read]
            count = 0
            while read < len(chars) and chars[read] == char:
                count += 1
                read += 1

            chars[write] = char
            write += 1
            if count > 1:
                for digit in str(count):
                    chars[write] = digit
                    write += 1

        return write
```

Two small cleanups worth making before calling this submission-ready:

1. The `print(...)` debug lines that showed up in earlier attempts should come out — harmless while testing, but the kind of thing that gets flagged in a real code review.
2. An earlier attempt had a special case for `count > 9` (looping differently for two-digit-or-longer counts). It's not needed — `for digit in str(count)` already handles any number of digits uniformly, whether `count` is `3` or `300`. Looping over `str(number)` is a good general habit for sidestepping "small number vs. big number" special-casing.

**Why it works:**

Two pointers do two different jobs. `read` scans forward, and for each group first captures the character into `char` — a frozen snapshot — then counts how many times that exact character repeats in a row. `write` marks where the next piece of the compressed answer goes — always at or behind `read`, so writing never overwrites data that hasn't been read yet.

For each group:
1. Snapshot `char = chars[read]`, then let the inner loop advance `read` while `chars[read] == char`, counting matches.
2. Once the group ends, write the character once into `chars[write]`, and advance `write` by exactly one.
3. If the group had more than one character, write each digit of the count into `chars`, one box per digit, advancing `write` each time.
4. The outer loop continues from wherever `read` stopped.

There is no separate string anywhere in this version — every letter and digit that would have gone into a string `s` gets written directly into `chars` instead. That's what "store it in `chars`" means: the array becomes the answer, in place, rather than a new structure being built and copied over.

Two details make this version specifically correct, where earlier attempts weren't:

- **`write` only ever moves via `write += 1`, once per character actually written.** No shortcuts, no `write = read` — those two variables track genuinely different things (compressed boxes filled vs. original boxes scanned), and they only coincidentally match for groups of size 1.
- **The group-matching comparison uses `char`, a frozen snapshot, never `chars[write]`.** `chars[write]` is a live box being actively overwritten throughout the run — the moment `write` falls behind `read` (which happens whenever a group actually compresses), that box may hold stale, unrelated leftover data instead of the character actually being grouped.

**Why the return value matters:** the array can't physically shrink, so anything after the compressed content (like a leftover original character) just stays sitting in `chars`, unchanged. Returning `write` tells the grader exactly where the real answer ends — `chars[:write]` is the compressed result; anything past that is explicitly meant to be ignored.

**Complexity:**
- Time: O(n) — `right` (and therefore `left`) visits each character exactly once across the whole run.
- Space: O(1) extra — only a handful of single-value pointers (`write`, `left`, `right`, `count`, `char`), no second array or dict.

**The one-line takeaway:** *"One finger counts a group, one finger writes it down — and the writing finger never needs to know where it is, only where it left off."*
