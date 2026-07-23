# LC 345 — Reverse Vowels of a String

**Link:** https://leetcode.com/problems/reverse-vowels-of-a-string/
**Difficulty:** Easy
**Pattern:** Two Pointers (converging)

---

## What I tried first

I did it in two passes.

**Pass 1:** walk the whole string once. Every time I found a vowel, I saved its index in a list.

**Pass 2:** use two pointers on that list of indices — one at the front, one at the back. Swap the letters at those positions, then move both pointers inward. Keep going until they meet.

```python
class Solution:
    def reverseVowels(self, s: str) -> str:
        vowels = 'aeiou'
        indices = []
        for i, c in enumerate(s):
            print(i,c)
            if c.lower() in vowels:
                indices.append(i)

        left = 0
        right = len(indices) -1
        s = list(s)
        while left < right:
            s[indices[left]], s[indices[right]] = s[indices[right]], s[indices[left]]
            left += 1
            right -= 1
        
        return ''.join(s)
```

It worked. But it used extra space to store all the vowel indices, and it took two full passes to do a job that only needed one. There's also a leftover `print(i, c)` debug line in there — a small thing, but worth flagging, since it's the kind of line that gets caught in a real code review.

**Complexity of this attempt:**

- **Time: O(n).** Pass 1 walks the string once to find vowels — O(n). Pass 2 walks the indices list once to swap — at most O(n) if every character is a vowel. Two passes, but both are O(n), so the total is still O(n) overall (two separate O(n) walks add up to O(n), not O(n²)).
- **Space: O(n).** This is the real cost. The `indices` list can hold up to n positions if every character in the string is a vowel. That's extra memory the final solution doesn't need. (The `list(s)` conversion is required either way, since Python strings are immutable — that part isn't the issue.)

So this version is the same time complexity as the final one, but it pays for a whole second data structure it didn't need. That's the real lesson from this mistake — not that it was *slow*, but that it was *wasteful*.

---

## Mistakes I made

**Mistake 1 — collecting indices instead of scanning the string directly.**

I reached for a list before I reached for two pointers on the string itself. The string already has a front and a back. I didn't need to build a separate list of positions — I could walk in from both ends of the string directly, and just skip over consonants as I went.

**Mistake 2 — checking vowels with `.lower()` and a string.**

My first check was:

```python
vowels = 'aeiou'
if c.lower() in vowels:
```

Two small problems here. First, `.lower()` runs on every single character, every time — extra work I don't need. Second, checking `c in 'aeiou'` walks the string one letter at a time to see if it matches. A set does this in one step, no walking. Fix:

```python
vowels = set('aeiouAEIOU')
if c in vowels:
```

Small savings here since there are only 5 vowels — but it's the right habit. Any time I'm asking "is this in here?" over and over inside a loop, a set is the right tool.

**Mistake 3 — I froze on the `else` block.**

When I tried the direct two-pointer version, I got stuck writing the branches. My worry was: *"How do I know, in the `else` block, that both letters are actually vowels?"*

Here's what fixed it. At any moment, there are only **4 possible situations**:

1. left = vowel, right = vowel
2. left = vowel, right = consonant
3. left = consonant, right = vowel
4. left = consonant, right = consonant

I only want to swap in situation 1. In the other three, someone isn't ready, so I move a pointer and check again.

Writing it as `if / elif / else` handles this automatically:

```python
if s[left] not in vowels:      # catches situation 3 and 4
    left += 1
elif s[right] not in vowels:   # catches situation 2 (and the rest of 4)
    right -= 1
else:                          # only situation 1 is left
    # swap
```

The trick: by the time the code reaches `else`, it already passed the first check (so left IS a vowel) and the second check (so right IS a vowel). I don't need to re-check anything in the `else` block — the lines above it already proved both are vowels, just by process of elimination.

---

## Final solution, with explanation

```python
class Solution:
    def reverseVowels(self, s: str) -> str:
        vowels = set('aeiouAEIOU')
        s = list(s)
        left, right = 0, len(s) - 1

        while left < right:
            if s[left] not in vowels:
                left += 1
            elif s[right] not in vowels:
                right -= 1
            else:
                s[left], s[right] = s[right], s[left]
                left += 1
                right -= 1

        return ''.join(s)
```

**Why it works:**

Two pointers start at opposite ends of the string and walk toward each other. Each pointer only cares about one question: *am I standing on a vowel?*

- If `left` is not on a vowel, it's not ready to swap — move it one step in.
- If `right` is not on a vowel (and we only get here if `left` already IS a vowel), move it one step in.
- If we reach the `else`, both checks above already failed — meaning both `left` and `right` are sitting on vowels. Now, and only now, swap them. Then move both pointers in.

This is the same pattern as Valid Palindrome — two pointers converging from both ends, skipping over anything that isn't relevant. The only difference here is that instead of skipping *and comparing*, we're skipping *and swapping*.

**Why the loop stops:** every single branch moves `left` forward or `right` backward. The gap between them shrinks by at least one on every turn, so the loop always ends — never both pointers stuck in place, never moving apart.

**Complexity:**
- Time: O(n) — one pass, each pointer visits each position at most once.
- Space: O(n) for converting the string to a list (Python strings are immutable, so this step is unavoidable, and both solutions pay it) — but O(1) *extra* space beyond that, since no index list is needed.

**Side by side:**

| | First attempt | Final solution |
|---|---|---|
| Time | O(n) | O(n) |
| Extra space (beyond the required list conversion) | O(n) — the indices list | O(1) — just two pointers |
| Passes over the string | 2 | 1 |

Same time complexity either way — the real win of the final solution is the space, and doing it in a single pass instead of two.

**The one-line takeaway:** *"Walk two fingers inward. Skip non-vowels. Swap what's left."*
