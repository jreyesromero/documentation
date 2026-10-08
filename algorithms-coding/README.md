# Algorithms & Coding Interview Prep

Welcome to the coding interview guide for SRE, DevOps, and Platform Engineers. This section covers the **interview protocol** (how to communicate), **complexity analysis** (Big-O), and the **core LeetCode patterns** with annotated Python solutions.

> **How to use this section:** These docs make your practice *efficient* — they are a reference and scaffold, not a substitute for actually writing code. Read the pattern, study the annotated solution, then **solve the problems yourself** on LeetCode/NeetCode to build real muscle memory.

## Quick Links

**New to Python?** Start here → [Python Basics](./python-basics/README.md) — 8-12 hours of foundational Python tailored for interviews, with exercises and SRE-specific patterns.

## Table of Contents

### Meta-Skills (read these first)

- [Interview Protocol](./interview-protocol.md) - How to Approach Any Problem
  - The 6-step method: clarify → examples → approach → code → test → analyze
  - Think out loud, brute force first, ask for hints
  - What to do when requirements change

- [Complexity Analysis](./complexity-analysis.md) - Big-O
  - The complexity hierarchy (O(1) → O(n!))
  - How to recognize each complexity
  - Time vs space trade-offs
  - "Numbers Every Programmer Should Know" (latency table)

### The Core Patterns

- [Arrays & HashMap](./patterns-arrays-hashmap.md) - O(1) Lookups
  - Two Sum, Contains Duplicate, Valid Anagram, Group Anagrams, Top K Frequent

- [Two Pointers](./patterns-two-pointers.md) - Converging & Slow/Fast
  - Valid Palindrome, Two Sum II (sorted), 3Sum

- [Sliding Window](./patterns-sliding-window.md) - Contiguous Subarrays
  - Best Time to Buy/Sell Stock, Longest Substring Without Repeating, Minimum Window Substring

- [Stack](./patterns-stack.md) - LIFO & Matching
  - Valid Parentheses, Min Stack, Evaluate Reverse Polish Notation

- [Trees & Graphs](./patterns-trees-graphs.md) - BFS / DFS
  - Maximum Depth, Level Order Traversal, Number of Islands, Connected Components

- [Binary Search & Linked Lists](./patterns-binary-search-linked-list.md) - Halving & Pointers
  - Binary Search, Search in Rotated Sorted Array, Reverse List, Merge Two Lists, Cycle Detection

---

## Pattern Recognition Cheat Sheet

The hardest part is knowing *which* pattern a problem calls for. This table maps signals to patterns:

| Signal in the Problem | Pattern |
|-----------------------|---------|
| "Find a pair/count/seen before" | [Arrays & HashMap](./patterns-arrays-hashmap.md) |
| "Sorted array, find pair/triplet" | [Two Pointers](./patterns-two-pointers.md) |
| "Check palindrome" | [Two Pointers](./patterns-two-pointers.md) |
| "Longest/shortest CONTIGUOUS subarray/substring" | [Sliding Window](./patterns-sliding-window.md) |
| "Matching brackets / nesting / expression" | [Stack](./patterns-stack.md) |
| "Shortest path / level by level" | [BFS](./patterns-trees-graphs.md) |
| "Explore fully / tree depth / path exists" | [DFS](./patterns-trees-graphs.md) |
| "Count islands / components" | [DFS/BFS](./patterns-trees-graphs.md) |
| "Sorted + O(log n) expected" | [Binary Search](./patterns-binary-search-linked-list.md) |
| "Reverse/merge/cycle in a list" | [Linked Lists](./patterns-binary-search-linked-list.md) |

---

## The Complete Problem List (Verified)

All problems with verified LeetCode numbers and difficulty, grouped by priority.

### Level 1 — Essential (master these first)
| # | Problem | Difficulty | Pattern |
|---|---------|-----------|---------|
| 1 | Two Sum | Easy | Arrays/HashMap |
| 217 | Contains Duplicate | Easy | Arrays/HashMap |
| 242 | Valid Anagram | Easy | Arrays/HashMap |
| 49 | Group Anagrams | Medium | Arrays/HashMap |
| 347 | Top K Frequent Elements | Medium | Arrays/HashMap |

### Level 2 — Two Pointers
| # | Problem | Difficulty | Pattern |
|---|---------|-----------|---------|
| 125 | Valid Palindrome | Easy | Two Pointers |
| 167 | Two Sum II (sorted) | Medium | Two Pointers |
| 15 | 3Sum | Medium | Two Pointers |

### Level 3 — Sliding Window
| # | Problem | Difficulty | Pattern |
|---|---------|-----------|---------|
| 121 | Best Time to Buy and Sell Stock | Easy | Sliding Window |
| 3 | Longest Substring Without Repeating | Medium | Sliding Window |
| 76 | Minimum Window Substring | Hard | Sliding Window |

### Level 4 — Stack
| # | Problem | Difficulty | Pattern |
|---|---------|-----------|---------|
| 20 | Valid Parentheses | Easy | Stack |
| 155 | Min Stack | Medium | Stack |
| 150 | Evaluate Reverse Polish Notation | Medium | Stack |

### Level 5 — If You Have Time (Search, Lists, Trees, Graphs)
| # | Problem | Difficulty | Pattern |
|---|---------|-----------|---------|
| 704 | Binary Search | Easy | Binary Search |
| 33 | Search in Rotated Sorted Array | Medium | Binary Search |
| 206 | Reverse Linked List | Easy | Linked Lists |
| 21 | Merge Two Sorted Lists | Easy | Linked Lists |
| 104 | Maximum Depth of Binary Tree | Easy | Trees/DFS |
| 102 | Binary Tree Level Order Traversal | Medium | Trees/BFS |
| 200 | Number of Islands | Medium | Graphs/DFS |
| 323* | Number of Connected Components | Medium | Graphs/DFS |

*#323 is LeetCode Premium — practice the free equivalent **#547 "Number of Provinces"** (same pattern).

---

## Topics Explicitly Called Out

These are specifically mentioned as interview topics — make sure you're comfortable:

- **Big-O analysis** → [Complexity Analysis](./complexity-analysis.md)
- **Tree traversals** (in-order, pre-order, post-order, BFS/DFS) → [Trees & Graphs](./patterns-trees-graphs.md)
- **Stacks / queues / cycles** (cycle detection, Floyd's) → [Stack](./patterns-stack.md) + [Linked Lists](./patterns-binary-search-linked-list.md)
- **Asynchrony** (async/await, concurrency primitives — be ready to discuss conceptually)
- **"Numbers Every Programmer Should Know"** (latency numbers) → [Complexity Analysis](./complexity-analysis.md)

---

## External Practice Resources

- **LeetCode** — the source of all problems above; practice by pattern
- **NeetCode** (neetcode.io) — organizes problems by *exactly* these patterns, with free video explanations
- **Blind 75 / NeetCode 150** — curated lists that are precisely this set of problems
- **HackerRank** — alternative practice platform

---

## Study Plan

```
Week 1: Patterns + easy problems
  □ Read Interview Protocol + Complexity Analysis
  □ Arrays/HashMap pattern — solve all 5 (Two Sum first)
  □ Two Pointers — solve Valid Palindrome, Two Sum II
  □ Stack — solve Valid Parentheses

Week 2: Mediums + fluency
  □ Sliding Window — Longest Substring, Best Time to Buy/Sell
  □ Trees/Graphs — Max Depth, Level Order, Number of Islands
  □ Binary Search — both problems
  □ 3Sum (the hardest of the "essential" set)

Week 3: Practice under pressure
  □ Re-solve each pattern's flagship problem from scratch, out loud
  □ Time yourself (aim: easy in ~10 min, medium in ~20)
  □ Practice the 6-step protocol every time
```

---

## Interview Focus

**The non-negotiables:**
1. **Think out loud** — the interviewer scores your reasoning, not just the answer
2. **Brute force first, then optimize** — always mention the naive approach
3. **State complexity** — time AND space, with justification, every time
4. **Test edge cases** — empty, single element, duplicates
5. **Pattern recognition** — identify the pattern from the problem signals
6. **Ask for hints when stuck** — it's encouraged, not penalized

---

**Note:** Unlike the Linux/Networking/Kubernetes sections (where the doc *is* the knowledge), coding ability comes from *doing*. Use these docs to recognize patterns and study clean solutions — then close the doc and solve the problems yourself. That's what builds the skill the interview tests.

**Key Principle:** Most coding interviews are pattern-matching. There are only ~8 core patterns; once you recognize which one a problem calls for, the solution follows. Master the patterns, not individual problems.
