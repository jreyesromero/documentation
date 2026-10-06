# The Coding Interview Protocol

How to approach a coding problem in an interview. The interviewer cares **more about your thinking than a perfect answer** — walk them through your assumptions, trade-offs, and reasoning. This protocol is how you demonstrate that.

## The Golden Rule

> "We care more about your thinking than a perfect answer. Walk us through your assumptions, trade-offs, alternatives, and why you'd choose one approach."

**Translation:** A silent candidate who writes a perfect solution scores *worse* than one who thinks out loud, explains trade-offs, and asks good questions — even with a minor bug.

---

## The 6-Step Protocol

When you receive a problem, **do NOT start coding immediately.** Follow these steps:

```
1. CLARIFY    → Ask questions, confirm understanding
2. EXAMPLES   → Walk through a sample input by hand
3. APPROACH   → State brute force, then optimize, out loud
4. CODE       → Write it, narrating as you go
5. TEST       → Trace through edge cases
6. ANALYZE    → State time and space complexity
```

---

### Step 1: Clarify (before any code)

Ask questions to confirm you understand and to surface edge cases. This shows maturity and prevents you from solving the wrong problem.

**Good clarifying questions:**
```
"Can the input be empty?"
"Are there duplicates?"
"Is the array sorted?"
"Can values be negative?"
"What should I return if there's no valid answer?"
"How large can the input be?" (hints at required complexity)
"Can I modify the input in place, or should I preserve it?"
```

**Example:**
```
Problem: "Find two numbers that add up to a target."

You: "A few questions first — can I assume exactly one solution exists, 
     or should I handle multiple or none? Can the same element be used 
     twice? And are the numbers sorted?"
```

This 20-second habit signals senior-level thinking.

---

### Step 2: Examples (work it by hand)

Walk through a concrete example **before coding**. It confirms your understanding and often reveals the approach.

```
Problem: Two Sum, nums=[2,7,11,15], target=9

You: "Let me trace it. I need two numbers summing to 9. 2+7=9, so the 
     answer is indices 0 and 1. Let me think about how I'd find that 
     systematically..."
```

Also consider an edge-case example:
```
"What about [3,3] with target 6? Same value twice, different indices — 
 so I need to handle duplicates carefully."
```

---

### Step 3: Approach (brute force → optimize, OUT LOUD)

**This is the most important step.** State the naive solution, then improve it, narrating your reasoning. This is where you show your thinking.

```
You: "The brute force is to check every pair — two nested loops. That's 
     O(n²) time. It works but it's slow for large inputs.

     I can do better with a hash map. As I iterate, for each number I 
     compute the complement — target minus the current number — and 
     check if I've already seen it. That's O(n) time, at the cost of 
     O(n) space for the map.

     I'll go with the hash map approach — trading space for time."
```

**The pattern to verbalize:**
```
"The brute force would be O(n²)..."
"I could optimize with a hash map..."
"That trades space for time..."
"Let me go with [approach] because [reason]."
```

---

### Step 4: Code (narrate as you write)

Now write the code, explaining as you go. Keep narrating — don't go silent.

```
You: "I'll create a dictionary to store numbers I've seen mapped to 
     their index. Then iterate... for each number, compute the 
     complement, check if it's in the map, and if so return both 
     indices..."
```

**Write clean code:**
- Meaningful variable names (`complement`, not `x`)
- Handle the edge cases you clarified
- Don't prematurely optimize readability away

---

### Step 5: Test (trace edge cases)

**Walk through your code** with real inputs, including edge cases. Catch your own bugs before the interviewer does.

**Standard edge cases to always check:**
```
□ Empty input          []
□ Single element       [5]
□ Two elements         [1, 2]
□ Duplicates           [3, 3]
□ Negative numbers     [-1, -2]
□ No valid answer      (what do you return?)
□ The normal case      [2, 7, 11, 15]
```

```
You: "Let me trace [2,7,11,15], target 9:
     i=0, num=2, complement=7, not in map → store {2:0}
     i=1, num=7, complement=2, IS in map → return [0,1]. Correct.
     
     Edge case — empty array: loop doesn't run, returns []. Good.
     Edge case — [3,3] target 6: i=0 stores {3:0}, i=1 complement=3 
     found → [0,1]. Handles duplicates correctly."
```

---

### Step 6: Analyze (state complexity)

Always finish by stating time and space complexity, and justify it.

```
You: "Time is O(n) — I iterate the array once, and each hash map 
     lookup is O(1) average. Space is O(n) — in the worst case the 
     map holds every element before I find the pair."
```

---

## Additional Winning Behaviors

The study guide and interviewers explicitly reward these:

### Ask for Hints (it's encouraged, not penalized)

```
"I'm considering two approaches and I'm not sure which fits better 
 here — could you give me a hint about the expected complexity?"
```

Interviewers are there to **guide**, not just score. Asking a good question when stuck is better than freezing.

### Propose Alternatives

```
"I solved it with a hash map for O(n). An alternative is sorting first 
 and using two pointers — that's O(n log n) time but O(1) space. If 
 memory were constrained, I'd prefer that."
```

### Adapt When Requirements Change

Interviewers often change the problem mid-stream to test flexibility:

```
Interviewer: "Now what if the array is sorted?"
You: "Then I'd switch to two pointers — no extra space needed, O(n) 
     time, taking advantage of the sorted order."
```

### Review Your Code

The study guide specifically mentions checking for **errors and readability** before submitting:

```
"Let me review this quickly — variable names are clear, I handle the 
 empty case, and... yes, this looks correct. One thing I'd add in 
 production is input validation."
```

---

## What NOT to Do

```
❌ Start coding immediately without clarifying
❌ Go silent while thinking (narrate instead)
❌ Jump to the optimal solution without mentioning brute force
❌ Skip edge-case testing
❌ Forget to state complexity
❌ Freeze when stuck (ask for a hint instead)
❌ Get defensive when the interviewer changes requirements
```

---

## The Full Flow (Worked Example)

**Problem:** "Given an array, return true if any value appears twice."

```
CLARIFY:  "Can the array be empty? Are these integers? Should I 
          consider it case-sensitive if they were strings?" 
          → "Empty returns false, they're integers."

EXAMPLES: "[1,2,3,1] → true (1 repeats). [1,2,3] → false. [] → false."

APPROACH: "Brute force: compare every pair, O(n²). Better: use a set. 
          As I iterate, if a number's already in the set, return true. 
          That's O(n) time, O(n) space. Even simpler: compare len(set) 
          to len(array) — if they differ, there's a duplicate."

CODE:     def containsDuplicate(nums):
              seen = set()
              for num in nums:
                  if num in seen:
                      return True
                  seen.add(num)
              return False

TEST:     "[1,2,3,1]: stores 1,2,3, then sees 1 again → True. 
          []: loop skips, returns False. [1]: stores 1, returns False."

ANALYZE:  "O(n) time — single pass. O(n) space — the set. The one-liner 
          len(set(nums)) != len(nums) is O(n) too but less explicit 
          about early-exit."
```

---

## Interview Tips

1. **Never code first** — clarify, example, approach, THEN code
2. **Think out loud constantly** — silence is the enemy
3. **Brute force first** — always mention it before optimizing
4. **Test your own code** — trace edge cases before they ask
5. **State complexity** — every time, with justification
6. **Ask for hints when stuck** — it's encouraged, not a penalty
7. **Adapt gracefully** — when they change the problem, pivot calmly

---

## The One-Sentence Summary

> Clarify, give the brute force, optimize out loud, code cleanly, test edge cases, state Big-O — and never stop narrating your reasoning.
