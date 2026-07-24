# LC 283 — Move Zeroes

**Link:** https://leetcode.com/problems/move-zeroes/
**Difficulty:** Easy
**Pattern:** Two Pointers (reader/writer)

---

## What I tried first

Nothing I could write down yet — I got stuck before writing any code. My instinct was to make a copy of the array, build the clean version in the copy, and put it back. I couldn't picture how to do this using the original array only, in place, without a second array to fall back on.

The block wasn't the syntax. It was not seeing *which group of elements needed to move in order, and which group didn't matter at all.*

---

## Mistakes I made

**Mistake 1 — reaching for a copy before checking whether one was needed.**

Same root cause as earlier problems in this journal (LC 443): in-place problems use the *same* array as both source and destination. A copy isn't just extra space — it's solving a different, easier problem than the one being asked. The real question to ask first is always: *"do I actually need a second structure, or can I use two pointers on the array I already have?"*

**Mistake 2 — not splitting the array into "order matters" vs "filler" before thinking about the loop.**

The unlock was this question, asked on purpose: **which group needs to keep its original relative order, and which group is just filler — all interchangeable, doesn't matter which one lands where?**

- The non-zero numbers (`1, 3, 12`) — their order relative to each other matters. `1` has to stay before `3`, which has to stay before `12`.
- The zeroes — every zero is identical. Nobody can tell one zero from another, so it doesn't matter which slot a specific zero ends up in.

Once that split is visible, the algorithm is mechanical: scan and place the "order matters" group one at a time with a write pointer. For the "filler" group, don't think about them individually at all — just count how many slots are left over, and fill every one of them with the known filler value.

**Mistake 3 — worrying about what happens to the leftover boxes.**

Before seeing the answer, I wasn't sure what to do with whatever was left in the array after the non-zero numbers were placed. The fix: I didn't need to figure out what was *already* sitting there — I already knew, with certainty, what was supposed to be there instead (zero). So instead of trying to preserve or inspect the leftover boxes, the second loop just overwrites all of them, no questions asked.

---

## Final solution, with explanation

```python
class Solution:
    def moveZeroes(self, nums: List[int]) -> None:
        write = 0
        for read in range(len(nums)):
            if nums[read] != 0:
                nums[write] = nums[read]
                write += 1

        for i in range(write, len(nums)):
            nums[i] = 0
```

**Why it works:**

Two pointers, one job each. `read` scans through every element once. `write` marks how many boxes of the final answer have been filled so far — it only ever moves forward by exactly one, and only immediately after something real gets written into `nums[write]`.

The loop only writes when `nums[read] != 0` — that's the entire "should I keep this?" question. Every non-zero number gets copied to `nums[write]`, in the same order it was found, and `write` advances by one.

By the time the first loop finishes, `write` equals exactly the count of non-zero numbers. Everything from index `write` onward is leftover — and since the only thing that was ever skipped is a zero, the second loop can just fill all of it with `0`, with no need to check what's currently sitting there.

**Full trace on `[0,1,0,3,12]`:**

- `read=0`: `nums[0]=0` — skip.
- `read=1`: `nums[1]=1` — keep. `nums[0]=1`. `write=1`.
- `read=2`: `nums[2]=0` — skip.
- `read=3`: `nums[3]=3` — keep. `nums[1]=3`. `write=2`.
- `read=4`: `nums[4]=12` — keep. `nums[2]=12`. `write=3`.

After the first loop: `[1,3,12,3,12]` — front is correct, tail is stale leftover data.

Second loop fills index 3 and 4 with `0`: `[1,3,12,0,0]`. ✓

**Complexity:**
- Time: O(n) — every index is visited once by `read`, and once more in the backfill loop; two O(n) passes add up to O(n), not O(n²).
- Space: O(1) — no copy, no second data structure. Just two integer pointers.

**The one-line takeaway:** *"Scan and keep the group whose order matters. Whatever's left, I already know what goes there — I don't need to inspect it, just overwrite it."*
