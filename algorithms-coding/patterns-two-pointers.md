# Pattern: Two Pointers

Two pointers move through the data (often from both ends, or at different speeds) to solve problems in O(n) time with O(1) space. Especially powerful on **sorted arrays** and strings.

## When to Recognize This Pattern

```
"The array is SORTED and you need to find a pair/triplet..."  → two ends
"Check if a string is a palindrome..."                         → two ends
"Remove duplicates in place..."                                → slow/fast
"Find a pair summing to a target (sorted)..."                  → two ends
```

**The core insight:** On sorted data, you can move pointers *intelligently* — if the sum is too big, move the right pointer left; too small, move the left pointer right. This avoids the nested loop.

**Key advantage over hash map:** O(1) space instead of O(n). If the array is already sorted, two pointers often beats the hash-map approach on space.

---

## The Two Variants

### Variant A: Opposite Ends (converging)

```
[1, 2, 3, 4, 5, 6]
 ↑              ↑
left          right     → move them toward each other
```

Used for: palindromes, sorted-array pair sums, reversing.

### Variant B: Slow / Fast (same direction)

```
[1, 1, 2, 3, 3]
 ↑  ↑
slow fast               → fast scans ahead, slow marks a position
```

Used for: in-place dedup, cycle detection, partitioning.

---

## Problem 1: Valid Palindrome (#125, Easy)

**Statement:** Given a string, return `true` if it's a palindrome, considering only alphanumeric characters and ignoring case.

```
Input:  "A man, a plan, a canal: Panama"   → true
Input:  "race a car"                        → false
```

**Approach:** One pointer at each end. Skip non-alphanumeric characters. Compare (case-insensitive). Move inward.

```python
def isPalindrome(s):
    left, right = 0, len(s) - 1
    while left < right:
        # skip non-alphanumeric from the left
        while left < right and not s[left].isalnum():
            left += 1
        # skip non-alphanumeric from the right
        while left < right and not s[right].isalnum():
            right -= 1
        # compare
        if s[left].lower() != s[right].lower():
            return False
        left += 1
        right -= 1
    return True
```

**Trace** `"A man..."`:
```
left='A', right='a' → match (case-insensitive), move in
...skip spaces and punctuation...
pointers converge without mismatch → True ✓
```

**Complexity:** Time O(n) — each pointer moves at most n. Space O(1) — no extra structure.

---

## Problem 2: Two Sum II - Input Array Is Sorted (#167, Medium)

**Statement:** Given a **sorted** array, find two numbers that add up to `target`. Return their 1-indexed positions.

```
Input:  numbers = [2,7,11,15], target = 9
Output: [1,2]           (2 + 7 = 9, 1-indexed)
```

**Approach:** Because it's sorted, use two ends. If the sum is too big, the right value is too large → move right pointer left. If too small → move left pointer right.

```python
def twoSum(numbers, target):
    left, right = 0, len(numbers) - 1
    while left < right:
        total = numbers[left] + numbers[right]
        if total == target:
            return [left + 1, right + 1]   # 1-indexed
        elif total < target:
            left += 1                      # need a bigger sum
        else:
            right -= 1                     # need a smaller sum
    return []
```

**Trace** `[2,7,11,15]`, target 9:
```
left=2, right=15: sum=17 > 9 → right--
left=2, right=11: sum=13 > 9 → right--
left=2, right=7:  sum=9  = 9 → return [1,2] ✓
```

**Complexity:** Time O(n), Space O(1).

**Why this beats the hash-map Two Sum:** Same O(n) time, but O(1) space instead of O(n) — because the sorted order lets us decide which pointer to move. **This is the classic "what if the array is sorted?" follow-up** to the original Two Sum.

---

## Problem 3: 3Sum (#15, Medium)

**Statement:** Find all unique triplets `[a, b, c]` in the array such that `a + b + c = 0`.

```
Input:  nums = [-1, 0, 1, 2, -1, -4]
Output: [[-1, -1, 2], [-1, 0, 1]]
```

**Approach:** Sort first. Fix one number, then use two pointers on the rest to find pairs that sum to its negative. Skip duplicates to keep triplets unique.

```python
def threeSum(nums):
    nums.sort()                           # enables two pointers + dup skipping
    result = []
    for i in range(len(nums)):
        # skip duplicate values for the fixed element
        if i > 0 and nums[i] == nums[i-1]:
            continue
        # two pointers on the remainder
        left, right = i + 1, len(nums) - 1
        while left < right:
            total = nums[i] + nums[left] + nums[right]
            if total < 0:
                left += 1
            elif total > 0:
                right -= 1
            else:
                result.append([nums[i], nums[left], nums[right]])
                left += 1
                right -= 1
                # skip duplicates for the pair
                while left < right and nums[left] == nums[left-1]:
                    left += 1
                while left < right and nums[right] == nums[right+1]:
                    right -= 1
    return result
```

**The idea:** 3Sum = fix one element + 2Sum (two pointers) on the rest. Sorting is what enables both the two-pointer scan and the duplicate-skipping.

**Complexity:** Time O(n²) — an outer loop (n) × two-pointer scan (n). Space O(1) or O(n) depending on the sort. The sort itself is O(n log n), dominated by the O(n²).

**Interview note:** This is a step up — explain clearly that you're reducing 3Sum to "fix one, then 2Sum." The duplicate-skipping is the tricky part; narrate it.

---

## The Pattern Template

```python
# Variant A: opposite ends (sorted arrays, palindromes)
left, right = 0, len(arr) - 1
while left < right:
    if condition_met(arr[left], arr[right]):
        # found it / record it
        left += 1
        right -= 1
    elif need_bigger():
        left += 1          # move left up to increase
    else:
        right -= 1         # move right down to decrease

# Variant B: slow / fast (in-place work)
slow = 0
for fast in range(len(arr)):
    if keep(arr[fast]):
        arr[slow] = arr[fast]
        slow += 1
# slow now marks the boundary
```

---

## Interview Tips

1. **Sorted array + find pair/triplet → two pointers** — the #1 trigger
2. **Two pointers = O(1) space** — the advantage over hash maps when sorted
3. **"What if sorted?" follow-up** — pivot from hash-map Two Sum to two-pointer Two Sum II
4. **3Sum = fix one + 2Sum** — explain the reduction
5. **Skip duplicates** — the key detail for unique results in 3Sum
6. **Move the right pointer to shrink the sum, left to grow it** — on sorted data

---

## Common Pitfalls

1. **Forgetting to sort** — two pointers on sorted-array problems requires sorting first
2. **Infinite loops** — always move at least one pointer each iteration
3. **Duplicate triplets in 3Sum** — must skip equal adjacent values
4. **Off-by-one on indices** — watch 1-indexed vs 0-indexed (Two Sum II is 1-indexed)
5. **left < right vs left <= right** — for pairs, use `left < right` (two distinct elements)
