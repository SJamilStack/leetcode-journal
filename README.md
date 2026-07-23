# DSA Journal

A running log of coding problems I'm solving — not just the solutions, but
the mistakes and "aha" moments along the way. The goal is to actually get
better, not just collect green checkmarks.

## Structure

Each problem gets its own file, named:

```
{number}-{problem-slug}.md
```

e.g. `1512-merge-strings-alternately.md`

Every entry follows the same [template](./template.md):

1. Problem info (name, link, difficulty, pattern)
2. What I tried first
3. Mistakes I made — the most important part
4. Final solution, with an explanation of *why* it works
5. Similar problems
6. Review flag (revisit later?)

## Why

Solving a problem once doesn't mean the lesson sticks. Writing down what went
wrong and why the good solution is good is what makes it stick.

# LeetCode 75 — Grouped by Pattern (Series Taxonomy)

*The official LeetCode 75 groups problems by data structure/topic. This regroups them by the underlying **pattern** — matching the story-first taxonomy from this series, so you know which article/battle card to reach for.*

## Two Pointers — *"Walk backwards, skip spaces, grab whole words."*

- **LC 5** — [Reverse Vowels of a String](https://leetcode.com/problems/reverse-vowels-of-a-string/) — Converging pointers, swap-on-match
- **LC 6** — [Reverse Words in a String](https://leetcode.com/problems/reverse-words-in-a-string/) — This is literally LC 151 — the article's problem
- **LC 9** — [String Compression](https://leetcode.com/problems/string-compression/) — Reader/writer pointers, in-place
- **LC 10** — [Move Zeroes](https://leetcode.com/problems/move-zeroes/) — Reader/writer pointers
- **LC 11** — [Is Subsequence](https://leetcode.com/problems/is-subsequence/) — Two pointers, different speeds through two strings
- **LC 12** — [Container With Most Water](https://leetcode.com/problems/container-with-most-water/) — Converging pointers with a real greedy decision
- **LC 13** — [Max Number of K-Sum Pairs](https://leetcode.com/problems/max-number-of-k-sum-pairs/) — Sort + converging pointers
- **LC 32** — [Maximum Twin Sum of a Linked List](https://leetcode.com/problems/maximum-twin-sum-of-a-linked-list/) — Two pointers on a list (often paired with fast/slow to find the middle)

**Also two pointers, hiding under "Array/String" in the official list:**

- **LC 1** — [Merge Strings Alternately](https://leetcode.com/problems/merge-strings-alternately/) — Two pointers, one per string, alternating writes

## Sliding Window — *"Grow right while clean. Shrink left when dirty."*

- **LC 14** — [Maximum Average Subarray I](https://leetcode.com/problems/maximum-average-subarray-i/) — Fixed-size window — the gentle version from the article's ladder
- **LC 15** — [Maximum Number of Vowels in a Substring of Given Length](https://leetcode.com/problems/maximum-number-of-vowels-in-a-substring-of-given-length/) — Fixed-size window, one-in-one-out counting
- **LC 16** — [Max Consecutive Ones III](https://leetcode.com/problems/max-consecutive-ones-iii/) — Variable window, "dirty" = too many zeros flipped
- **LC 17** — [Longest Subarray of 1's After Deleting One Element](https://leetcode.com/problems/longest-subarray-of-1s-after-deleting-one-element/) — Variable window, same skeleton as 16 with a twist

## Prefix Sums — *"Mile markers: any stretch is end marker minus start marker."*

- **LC 7** — [Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/) — Prefix *and* suffix markers — multiply instead of subtract
- **LC 18** — [Find the Highest Altitude](https://leetcode.com/problems/find-the-highest-altitude/) — Build the markers, take the min — the LC 1480 warm-up flavor
- **LC 19** — [Find Pivot Index](https://leetcode.com/problems/find-pivot-index/) — This is LC 724 from the article's ladder, verbatim

## Binary Search — *"Paint it NO-NO-YES-YES, then hunt the first YES."*

- **LC 53** — [Guess Number Higher or Lower](https://leetcode.com/problems/guess-number-higher-or-lower/) — The pure boundary template
- **LC 54** — [Successful Pairs of Spells and Potions](https://leetcode.com/problems/successful-pairs-of-spells-and-potions/) — Binary search per query, on a sorted array
- **LC 55** — [Find Peak Element](https://leetcode.com/problems/find-peak-element/) — Monotonic *slope* check instead of a value check — good pattern-flexing problem
- **LC 56** — [Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/) — Binary search on the answer — literally the article's example

## Merge Intervals — *"Sort by start; overlap the last or start fresh."*

- **LC 72** — [Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/) — This is LC 435 from the article's ladder — greedy, sort by *end*
- **LC 73** — [Minimum Number of Arrows to Burst Balloons](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/) — Same greedy-interval family as 72

## Fast & Slow Pointers — *"If the track loops, the fast runner laps the slow one."*

- **LC 29** — [Delete the Middle Node of a Linked List](https://leetcode.com/problems/delete-the-middle-node-of-a-linked-list/) — The "find the middle" bonus move from the article, plus a delete

*(Article already covers this pattern — LC 141/142/876/202 — this is just the one 75-list problem that uses it.)*

## Monotonic Stack — *"Keep the pile sorted; whoever a newcomer evicts just found their answer."*

- **LC 74** — [Daily Temperatures](https://leetcode.com/problems/daily-temperatures/) — The canonical monotonic stack problem
- **LC 75** — [Online Stock Span](https://leetcode.com/problems/online-stock-span/) — Same idea, streaming/online flavor
- **LC 25** — [Asteroid Collision](https://leetcode.com/problems/asteroid-collision/) — Stack simulation — "who survives the collision" is the same eviction idea

## Stacks (general, not monotonic) — LIFO / undo / nesting

- **LC 24** — [Removing Stars From a String](https://leetcode.com/problems/removing-stars-from-a-string/) — Stack as "undo the last thing"
- **LC 26** — [Decode String](https://leetcode.com/problems/decode-string/) — Stack for nested structure (brackets)

## Queue (BFS's building block)

- **LC 27** — [Number of Recent Calls](https://leetcode.com/problems/number-of-recent-calls/) — Pure "sliding window over a queue" — arrivals fall off the front
- **LC 28** — [Dota2 Senate](https://leetcode.com/problems/dota2-senate/) — Two queues, elimination simulation

## Linked List Reversal / Manipulation

- **LC 30** — [Odd Even Linked List](https://leetcode.com/problems/odd-even-linked-list/) — Two "tails," relinking — no reversal but same pointer-surgery family
- **LC 31** — [Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/) — The canonical three-pointer reversal (prev/curr/next)

## BFS (Trees + Graphs) — *"Ripples in a pond."*

- **LC 39** — [Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/) — Level-order, take the last node per level
- **LC 40** — [Maximum Level Sum of a Binary Tree](https://leetcode.com/problems/maximum-level-sum-of-a-binary-tree/) — Level-order, sum per level
- **LC 47** — [Nearest Exit from Entrance in Maze](https://leetcode.com/problems/nearest-exit-from-entrance-in-maze/) — Grid BFS, shortest path
- **LC 48** — [Rotting Oranges](https://leetcode.com/problems/rotting-oranges/) — Multi-source BFS — several starting points at once

## DFS (Trees + Graphs) — *"Explore the maze; dead end → walk back to the last fork."*

- **LC 33** — [Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/) — The simplest recursive DFS
- **LC 34** — [Leaf-Similar Trees](https://leetcode.com/problems/leaf-similar-trees/) — DFS collecting a sequence, then compare
- **LC 35** — [Count Good Nodes in Binary Tree](https://leetcode.com/problems/count-good-nodes-in-binary-tree/) — DFS carrying state (running max) down the path
- **LC 36** — [Path Sum III](https://leetcode.com/problems/path-sum-iii/) — DFS + prefix sum combined — a cross-pattern problem
- **LC 37** — [Longest ZigZag Path in a Binary Tree](https://leetcode.com/problems/longest-zigzag-path-in-a-binary-tree/) — DFS carrying direction + length state
- **LC 38** — [Lowest Common Ancestor of a Binary Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/) — Classic DFS-returns-info-upward pattern
- **LC 41** — [Search in a Binary Search Tree](https://leetcode.com/problems/search-in-a-binary-search-tree/) — DFS + BST property (binary search on a tree)
- **LC 42** — [Delete Node in a BST](https://leetcode.com/problems/delete-node-in-a-bst/) — BST structural surgery
- **LC 43** — [Keys and Rooms](https://leetcode.com/problems/keys-and-rooms/) — Graph DFS, reachability
- **LC 44** — [Number of Provinces](https://leetcode.com/problems/number-of-provinces/) — Connected components — DFS or Union-Find
- **LC 45** — [Reorder Routes to Make All Paths Lead to the City Zero](https://leetcode.com/problems/reorder-routes-to-make-all-paths-lead-to-the-city-zero/) — Graph DFS with edge direction tracking
- **LC 46** — [Evaluate Division](https://leetcode.com/problems/evaluate-division/) — Graph built from equations, DFS/weighted path

## Backtracking — generate every valid configuration

- **LC 57** — [Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/) — Classic combinatorial backtracking
- **LC 58** — [Combination Sum III](https://leetcode.com/problems/combination-sum-iii/) — Backtracking with a pruning bound

## Heaps / Top-K — *"A bouncer holds the k best; newcomers must beat the worst member."*

- **LC 49** — [Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/) — The canonical top-k-heap problem
- **LC 50** — [Smallest Number in Infinite Set](https://leetcode.com/problems/smallest-number-in-infinite-set/) — Min-heap + set, "smallest available"
- **LC 51** — [Maximum Subsequence Score](https://leetcode.com/problems/maximum-subsequence-score/) — Sort + min-heap, greedy with a heap for bookkeeping
- **LC 52** — [Total Cost to Hire K Workers](https://leetcode.com/problems/total-cost-to-hire-k-workers/) — Two-heap simulation

## Hash Map / Hash Set — *"Pick by the question your loop asks."*

*(Not really a separate "pattern" in the story sense — this is the data-structure reference file's territory. Listed here for completeness.)*

- **LC 2** — [Greatest Common Divisor of Strings](https://leetcode.com/problems/greatest-common-divisor-of-strings/) — Math/string, not really hashmap — official list mis-bucket
- **LC 3** — [Kids With the Greatest Number of Candies](https://leetcode.com/problems/kids-with-the-greatest-number-of-candies/) — Track max, then scan
- **LC 4** — [Can Place Flowers](https://leetcode.com/problems/can-place-flowers/) — Greedy array scan
- **LC 8** — [Increasing Triplet Subsequence](https://leetcode.com/problems/increasing-triplet-subsequence/) — Greedy, track two running minimums
- **LC 20** — [Find the Difference of Two Arrays](https://leetcode.com/problems/find-the-difference-of-two-arrays/) — Set difference — "is X inside?" question, textbook set use
- **LC 21** — [Unique Number of Occurrences](https://leetcode.com/problems/unique-number-of-occurrences/) — Counter, then check counts are unique
- **LC 22** — [Determine if Two Strings Are Close](https://leetcode.com/problems/determine-if-two-strings-are-close/) — Counter comparison
- **LC 23** — [Equal Row and Column Pairs](https://leetcode.com/problems/equal-row-and-column-pairs/) — Hashmap of tuples/rows

## Dynamic Programming — *"Answer small versions, write them down, build up."*

- **LC 59** — [N-th Tribonacci Number](https://leetcode.com/problems/n-th-tribonacci-number/) — 1D DP, the gentlest possible start
- **LC 60** — [Min Cost Climbing Stairs](https://leetcode.com/problems/min-cost-climbing-stairs/) — 1D DP
- **LC 61** — [House Robber](https://leetcode.com/problems/house-robber/) — 1D DP, "take or skip" — the archetypal DP decision
- **LC 62** — [Domino and Tromino Tiling](https://leetcode.com/problems/domino-and-tromino-tiling/) — 1D DP, trickier recurrence
- **LC 63** — [Unique Paths](https://leetcode.com/problems/unique-paths/) — 2D grid DP
- **LC 64** — [Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/) — 2D string DP — foundational for a whole family (edit distance, etc.)
- **LC 65** — [Best Time to Buy and Sell Stock with Transaction Fee](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/) — DP with explicit states (holding / not holding)
- **LC 66** — [Edit Distance](https://leetcode.com/problems/edit-distance/) — 2D string DP — the classic hard-mode LCS cousin

## Bit Manipulation

- **LC 67** — [Counting Bits](https://leetcode.com/problems/counting-bits/) — DP + bit tricks
- **LC 68** — [Single Number](https://leetcode.com/problems/single-number/) — XOR trick
- **LC 69** — [Minimum Flips to Make a OR b Equal to c](https://leetcode.com/problems/minimum-flips-to-make-a-or-b-equal-to-c/) — Bit-by-bit reasoning

## Trie

- **LC 70** — [Implement Trie (Prefix Tree)](https://leetcode.com/problems/implement-trie-prefix-tree/) — Build the structure itself
- **LC 71** — [Search Suggestions System](https://leetcode.com/problems/search-suggestions-system/) — Trie + sorting/binary search combined

## Uncategorized / pure simulation (no single dominant pattern)

These are "read carefully and simulate" problems — good for reading comprehension practice, but not a pattern drill:

- (none left uncovered above — every problem in the 75 lands in a bucket)

## Suggested order to tackle the 75, following this series

If you're working through the study plan *alongside* the series (recommended), here's the mapping to "do this problem after this article":

1. **After Two Pointers article:** 1, 5, 6, 9, 10, 11, 12, 13, 32
2. **After Sliding Window article:** 14, 15, 16, 17
3. **After Fast & Slow article:** 29
4. **After Prefix Sums article:** 7, 18, 19
5. **After Binary Search article:** 53, 54, 55, 56
6. **After Merge Intervals article:** 72, 73
7. **Once we cover Monotonic Stack:** 25, 74, 75
8. **Once we cover Stacks/Queues:** 24, 26, 27, 28
9. **Once we cover Linked List reversal:** 30, 31
10. **Once we cover BFS/DFS:** 33–48
11. **Once we cover Backtracking:** 57, 58
12. **Once we cover Heaps:** 49, 50, 51, 52
13. **Hash Map problems** (data-structure reference, no dedicated article needed): 2, 3, 4, 8, 20, 21, 22, 23
14. **Once we cover DP:** 59–66
15. **Bit Manipulation, Trie:** 67–71 — standalone topics, minor patterns, tackle whenever

*This file will be updated as new pattern articles are published in the series. Check off rows as you clear them — this doubles as your LeetCode 75 progress tracker.*
