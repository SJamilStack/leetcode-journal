# Problem 11: Container With Most Water

## Problem Statement

You are given an array of integers `height` of length `n`. There are `n` vertical lines drawn such that the two endpoints of line `i` are at `(i, 0)` and `(i, height[i])`.

Find two lines that, together with the x-axis, form a container that holds the most water.

Return the maximum amount of water the container can store.

**Note:** You may not slant the container; it must stand straight.

**Constraints:**
- `2 <= n <= 10^5`
- `0 <= height[i] <= 10^4`

---

## Step 1: Say it in plain English

You get a list of numbers. Each number is the height of a wall, standing at a position.

You pick any two walls. Water sits between them. The water level cannot go higher than the shorter wall, because water spills over the shorter side.

Amount of water = height of shorter wall × distance between the two walls.

You want to pick two walls that hold the most water.

---

## Step 2: Look at the shape of input and output

- Input: array of numbers.
- Output: single number (max water).

Array input, comparing pairs. This hints two pointers.

---

## Step 3: What is the problem really asking?

Find the best value among many choices. That means we are searching for a max.

---

## Step 4: Check constraints

Array size can go up to 100,000. So O(n^2) is too slow. We need O(n).

---

## Step 5: Brute force first

Check every pair of walls. For each pair, calculate water, keep the biggest.

```python
def maxArea(height):
    n = len(height)
    best = 0
    for i in range(n):
        for j in range(i+1, n):
            water = min(height[i], height[j]) * (j - i)
            best = max(best, water)
    return best
```

**Time:** O(n^2)
**Space:** O(1)

This checks all pairs, so it's always correct, but slow.

---

## Step 6: Find the repeated waste

We check every pair, even pairs that clearly can't beat what we already found. That's wasted work.

---

## Step 7: The better idea — two pointers

Start with one pointer at the far left, one at the far right. This is the widest possible container.

Calculate water. Save it if it's the best so far.

Now move the pointer standing at the **shorter** wall, one step inward. Keep doing this until the two pointers meet.

**Why move the shorter one, not the taller one?**

The shorter wall decides the water height. Moving the taller wall only shrinks width and can never raise the water level, since the short wall is still short. Moving the shorter wall shrinks width too, but it gives a chance that the new wall is taller, which might raise the water level. So only moving the shorter wall has any chance of a better answer.

```python
def maxArea(height):
    left = 0
    right = len(height) - 1
    best = 0

    while left < right:
        water = min(height[left], height[right]) * (right - left)
        best = max(best, water)

        if height[left] < height[right]:
            left += 1
        else:
            right -= 1

    return best
```

---

## Trace on a small array

Array: `[1, 8, 6, 2, 5]`
Index:  ` 0  1  2  3  4`

Start: `left = 0`, `right = 4`, `best = 0`

**Step 1:**
- `height[left] = 1`, `height[right] = 5`
- width = `4 - 0 = 4`
- water = `min(1, 5) × 4 = 1 × 4 = 4`
- `best = max(0, 4) = 4`
- `height[left] (1) < height[right] (5)` → left is shorter → move `left` forward
- `left = 1`

**Step 2:**
- `height[left] = 8`, `height[right] = 5`
- width = `4 - 1 = 3`
- water = `min(8, 5) × 3 = 5 × 3 = 15`
- `best = max(4, 15) = 15`
- `height[left] (8) >= height[right] (5)` → right is shorter → move `right` backward
- `right = 3`

**Step 3:**
- `height[left] = 8`, `height[right] = 2`
- width = `3 - 1 = 2`
- water = `min(8, 2) × 2 = 2 × 2 = 4`
- `best = max(15, 4) = 15`
- `height[left] (8) >= height[right] (2)` → right is shorter → move `right` backward
- `right = 2`

**Step 4:**
- `left = 1`, `right = 2`, loop continues since `left < right`
- `height[left] = 8`, `height[right] = 6`
- width = `2 - 1 = 1`
- water = `min(8, 6) × 1 = 6 × 1 = 6`
- `best = max(15, 6) = 15`
- `height[left] (8) >= height[right] (6)` → right is shorter → move `right` backward
- `right = 1`

Now `left = 1` and `right = 1`. Loop stops, because `left < right` is false.

**Final answer: 15**

### Table view of the same trace

| Step | left | right | height[left] | height[right] | width | water | best so far | move |
|------|------|-------|---------------|----------------|-------|-------|--------------|------|
| 1 | 0 | 4 | 1 | 5 | 4 | 4 | 4 | move left |
| 2 | 1 | 4 | 8 | 5 | 3 | 15 | 15 | move right |
| 3 | 1 | 3 | 8 | 2 | 2 | 4 | 15 | move right |
| 4 | 1 | 2 | 8 | 6 | 1 | 6 | 15 | move right |
| stop | 1 | 1 | - | - | - | - | 15 | left = right, loop ends |

---

## Time and space

**Time:** O(n). Each pointer moves inward at most n times total, so every wall is visited only once.
**Space:** O(1). Only a few variables are used, no extra array or structure.

---

## Follow-up Q&A

**Q: Since min(8,5) = 5 at step 2, shouldn't we skip ahead and keep checking left/right until we find a wall taller than 5, instead of checking every single step in between?**

A: That sounds smart, but we don't know where the next taller wall is until we check it. There is no way to "peek ahead" without looking. Checking one index is a single cheap step (one comparison, one multiplication), so walking through the shorter walls one at a time costs almost nothing.

What we actually save with two pointers is not "skipping bad walls." It is **never pairing one wall against every other wall.** In brute force, the wall at index 1 would be paired separately with index 2, 3, and 4. In two pointers, each wall is visited only once total, as the window shrinks from both sides.

Think of it like closing a gate from both sides: two people walk toward the middle, one step at a time, and only shake hands once each. That is O(n). Comparing everyone on the left with everyone on the right would be O(n²).
