# Pattern: Arrays & HashMap

The most fundamental pattern. A hash map (dict/set in Python) gives O(1) lookups, which turns many O(n²) brute-force solutions into O(n). If you see "find a pair," "count occurrences," or "have I seen this before," think hash map.

## When to Recognize This Pattern

```
"Find two elements that..."           → hash map of complements
"Count how many times..."             → hash map of counts
"Has this appeared before?"           → set membership
"Group things that share..."          → hash map keyed by the shared property
"Find the top K most frequent..."     → hash map of counts + heap/bucket
```

**The core insight:** A hash map trades O(n) space for O(1) lookups, eliminating a nested loop.

---

## Problem 1: Two Sum (#1, Easy)

**Statement:** Given an array `nums` and a `target`, return the indices of the two numbers that add up to `target`. Exactly one solution exists; you can't use the same element twice.

```
Input:  nums = [2,7,11,15], target = 9
Output: [0,1]           (nums[0] + nums[1] = 2 + 7 = 9)
```

**Approach:** For each number, the number we *need* is `target - num` (the complement). Store each number we've seen in a hash map; if the complement is already there, we found the pair.

```python
def twoSum(nums, target):
    seen = {}                       # value -> index
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
    return []
```

**Trace** `[2,7,11,15]`, target 9:
```
i=0, num=2:  complement=7, not in seen → seen={2:0}
i=1, num=7:  complement=2, IS in seen  → return [0, 1] ✓
```

**Complexity:** Time O(n) — single pass. Space O(n) — the hash map.

**Why not brute force?** Checking every pair is O(n²). The hash map removes the inner loop by remembering what we've seen.

---

## Problem 2: Contains Duplicate (#217, Easy)

**Statement:** Return `true` if any value appears at least twice.

```
Input:  [1,2,3,1]   → true
Input:  [1,2,3,4]   → false
```

**Approach:** A set of seen values. If we see one already in the set, it's a duplicate.

```python
def containsDuplicate(nums):
    seen = set()
    for num in nums:
        if num in seen:
            return True
        seen.add(num)
    return False
```

**One-liner alternative:**
```python
def containsDuplicate(nums):
    return len(set(nums)) != len(nums)   # if the set is smaller, there were dupes
```

**Complexity:** Time O(n), Space O(n).

**Interview note:** The explicit loop is better to present because it *early-exits* on the first duplicate, and shows your reasoning. Mention the one-liner as an alternative.

---

## Problem 3: Valid Anagram (#242, Easy)

**Statement:** Given two strings `s` and `t`, return `true` if `t` is an anagram of `s` (same characters, same counts).

```
Input:  s = "anagram", t = "nagaram"   → true
Input:  s = "rat",     t = "car"       → false
```

**Approach:** Count characters in both. If the counts match, it's an anagram.

```python
from collections import Counter

def isAnagram(s, t):
    if len(s) != len(t):              # quick reject
        return False
    return Counter(s) == Counter(t)
```

**Manual version (shows the counting logic):**
```python
def isAnagram(s, t):
    if len(s) != len(t):
        return False
    counts = {}
    for c in s:
        counts[c] = counts.get(c, 0) + 1
    for c in t:
        if c not in counts:
            return False
        counts[c] -= 1
        if counts[c] == 0:
            del counts[c]
    return len(counts) == 0
```

**Complexity:** Time O(n), Space O(1) if the alphabet is fixed (26 letters), else O(n).

---

## Problem 4: Group Anagrams (#49, Medium)

**Statement:** Given an array of strings, group the anagrams together.

```
Input:  ["eat","tea","tan","ate","nat","bat"]
Output: [["eat","tea","ate"], ["tan","nat"], ["bat"]]
```

**Approach:** Anagrams share the same sorted letters. Use the **sorted string as a hash-map key**; all anagrams map to the same bucket.

```python
from collections import defaultdict

def groupAnagrams(strs):
    groups = defaultdict(list)
    for s in strs:
        key = ''.join(sorted(s))      # "eat" -> "aet", "tea" -> "aet"
        groups[key].append(s)
    return list(groups.values())
```

**Trace:**
```
"eat" → key "aet" → groups["aet"] = ["eat"]
"tea" → key "aet" → groups["aet"] = ["eat","tea"]
"tan" → key "ant" → groups["ant"] = ["tan"]
...
```

**Complexity:** Time O(n · k log k) — n strings, each of length k sorted in k log k. Space O(n · k).

**Optimization to mention:** Instead of sorting (k log k), you can key by a character-count tuple (26 counts), making it O(n · k). Worth noting as an alternative.

---

## Problem 5: Top K Frequent Elements (#347, Medium)

**Statement:** Given an array `nums` and integer `k`, return the `k` most frequent elements.

```
Input:  nums = [1,1,1,2,2,3], k = 2
Output: [1,2]           (1 appears 3×, 2 appears 2×)
```

**Approach:** Count frequencies with a hash map, then get the top K. **Bucket sort** by frequency gives O(n) — buckets indexed by count.

```python
from collections import Counter

def topKFrequent(nums, k):
    counts = Counter(nums)                    # value -> frequency
    # buckets[freq] = list of values with that frequency
    buckets = [[] for _ in range(len(nums) + 1)]
    for num, freq in counts.items():
        buckets[freq].append(num)

    result = []
    for freq in range(len(buckets) - 1, 0, -1):   # high freq → low
        for num in buckets[freq]:
            result.append(num)
            if len(result) == k:
                return result
    return result
```

**Why bucket sort?** Frequencies range from 1 to n, so we can index directly by frequency — no sorting needed. That's O(n) vs the O(n log n) of sorting by count.

**Simpler alternative (mention the trade-off):**
```python
def topKFrequent(nums, k):
    counts = Counter(nums)
    return [num for num, _ in counts.most_common(k)]   # O(n log k)
```

**Complexity:** Bucket approach — Time O(n), Space O(n). The `most_common` approach is O(n log k).

---

## The Pattern Template

```python
# Counting / seen-before
seen = set()               # or {} for key→value, Counter for counts
for item in collection:
    if item in seen:       # O(1) lookup replaces a nested loop
        # do something
    seen.add(item)

# Complement / pairing
seen = {}
for i, x in enumerate(arr):
    need = target - x      # what would complete the pair?
    if need in seen:
        return [seen[need], i]
    seen[x] = i

# Grouping by shared property
groups = defaultdict(list)
for item in collection:
    key = shared_property(item)   # sorted string, count tuple, etc.
    groups[key].append(item)
```

---

## Interview Tips

1. **Hash map turns O(n²) into O(n)** — the headline benefit; always mention the brute force first
2. **Complement trick** — for "find a pair summing to X," store seen values and look for `target - x`
3. **Set for membership, dict for key→value, Counter for counts**
4. **Group by a canonical key** — sorted string or count tuple for anagrams
5. **Bucket sort for top-K** — when the range of counts is bounded, beat O(n log n)
6. **State average vs worst** — hash map lookups are O(1) *average*

---

## Common Pitfalls

1. **Using the same element twice** — store index *after* checking the complement (Two Sum)
2. **Forgetting the length check** — anagrams of different lengths can't match
3. **Sorting when counting suffices** — count tuples beat sorting for Group Anagrams
4. **Mutable default arguments** — use `defaultdict`, not a shared dict
5. **Not mentioning the space cost** — hash maps are O(n) space; call it out
