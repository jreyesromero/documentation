# Pattern: Sliding Window

A window (a contiguous range) expands and contracts over the data to find the best/valid subarray or substring in O(n). Instead of re-examining every subrange (O(n²)), you reuse work as the window slides.

## When to Recognize This Pattern

```
"Longest/shortest SUBARRAY or SUBSTRING that..."     → sliding window
"Contiguous range with a constraint..."              → sliding window
"Maximum sum of k consecutive elements..."           → fixed window
"Smallest window containing..."                      → variable window
```

**Key word: CONTIGUOUS.** Sliding window is for *consecutive* elements. If order doesn't matter or elements can be non-adjacent, it's probably a different pattern.

**The core insight:** As the window slides, you add one element on the right and remove one on the left — O(1) per step — instead of recomputing the whole range.

---

## The Two Types

### Fixed-Size Window

The window is always `k` wide. Slide it across.

```
[1, 3, -1, -3, 5, 3]  k=3
 └─────┘                 window sum = 3
    └──────┘             slide: -1 (left out), +(-3) ... 
```

### Variable-Size Window

The window grows and shrinks based on a condition.

```
Expand right to include more. When a constraint breaks, 
shrink from the left until it's valid again.
```

---

## Problem 1: Best Time to Buy and Sell Stock (#121, Easy)

**Statement:** Given daily prices, find the max profit from one buy and one later sell. If no profit is possible, return 0.

```
Input:  [7,1,5,3,6,4]   → 5     (buy at 1, sell at 6)
Input:  [7,6,4,3,1]     → 0     (prices only fall)
```

**Approach:** Track the minimum price seen so far (the best buy point). At each day, the profit if we sold today is `price - min_so_far`. Keep the max.

```python
def maxProfit(prices):
    min_price = float('inf')
    max_profit = 0
    for price in prices:
        min_price = min(min_price, price)       # best buy point so far
        max_profit = max(max_profit, price - min_price)  # best sell today
    return max_profit
```

**Trace** `[7,1,5,3,6,4]`:
```
7: min=7, profit=0
1: min=1, profit=0
5: min=1, profit=4
3: min=1, profit=4
6: min=1, profit=5   ← best
4: min=1, profit=5
→ 5 ✓
```

**Complexity:** Time O(n), Space O(1).

**Window framing:** The "window" is buy-point → current-day. We expand the right edge (today) and track the best left edge (min price). One pass.

---

## Problem 2: Longest Substring Without Repeating Characters (#3, Medium)

**Statement:** Find the length of the longest substring with no repeating characters.

```
Input:  "abcabcbb"   → 3     ("abc")
Input:  "bbbbb"      → 1     ("b")
Input:  "pwwkew"     → 3     ("wke")
```

**Approach:** A variable window. Expand the right edge. If the new character is already in the window, shrink the left edge past its previous occurrence. Track the max window length.

```python
def lengthOfLongestSubstring(s):
    char_index = {}           # char -> last index seen
    left = 0
    max_len = 0
    for right in range(len(s)):
        char = s[right]
        # if we've seen this char INSIDE the current window, jump left past it
        if char in char_index and char_index[char] >= left:
            left = char_index[char] + 1
        char_index[char] = right
        max_len = max(max_len, right - left + 1)
    return max_len
```

**Trace** `"abcabcbb"`:
```
right=0 'a': window "a",   len 1
right=1 'b': window "ab",  len 2
right=2 'c': window "abc", len 3  ← max
right=3 'a': 'a' seen at 0 ≥ left → left=1, window "bca", len 3
right=4 'b': 'b' seen at 1 ≥ left → left=2, window "cab", len 3
...
→ 3 ✓
```

**Complexity:** Time O(n) — each character enters and leaves the window once. Space O(min(n, charset)) — the map.

**The key detail:** `char_index[char] >= left` — only jump if the repeat is *inside* the current window, not a stale earlier occurrence.

---

## Problem 3: Minimum Window Substring (#76, Hard)

**Statement:** Given strings `s` and `t`, find the smallest substring of `s` that contains all characters of `t` (including duplicates). Return "" if none exists.

```
Input:  s = "ADOBECODEBANC", t = "ABC"
Output: "BANC"
```

**Approach:** Variable window. Expand right until the window contains all of `t`. Then shrink from the left as much as possible while still valid, recording the smallest. This is the hardest of the three — a full expand/contract window.

```python
from collections import Counter

def minWindow(s, t):
    if not t or not s:
        return ""

    need = Counter(t)                  # chars we need, with counts
    missing = len(t)                   # total chars still needed
    left = 0
    best_len = float('inf')
    best_start = 0

    for right, char in enumerate(s):
        if need[char] > 0:             # this char helps fulfill the need
            missing -= 1
        need[char] -= 1                # consume it

        # when the window is valid (nothing missing), shrink from the left
        while missing == 0:
            if right - left + 1 < best_len:
                best_len = right - left + 1
                best_start = left
            need[s[left]] += 1         # give back the left char
            if need[s[left]] > 0:      # if we now need it again, window broke
                missing += 1
            left += 1

    return "" if best_len == float('inf') else s[best_start:best_start + best_len]
```

**The logic:**
```
1. Expand right, consuming characters (need[char] -= 1).
2. When missing == 0, the window has everything — try to shrink from left.
3. Shrink until removing a needed char breaks validity.
4. Track the smallest valid window seen.
```

**Complexity:** Time O(n) — each character is visited at most twice (once by right, once by left). Space O(k) — the character counts.

**Interview note:** This is a Hard problem — don't expect to nail it cold. The valuable thing is explaining the expand-then-contract strategy clearly, even if you need hints on the counting details.

---

## The Pattern Template

```python
# Variable-size sliding window
left = 0
window_state = {}        # or a count, sum, set — whatever you track
best = 0

for right in range(len(arr)):
    # 1. EXPAND: add arr[right] to the window
    add(arr[right], window_state)

    # 2. CONTRACT: while the window is invalid, shrink from the left
    while not valid(window_state):
        remove(arr[left], window_state)
        left += 1

    # 3. RECORD: update the best answer with the current valid window
    best = max(best, right - left + 1)

return best
```

```python
# Fixed-size window (size k)
window_sum = sum(arr[:k])
best = window_sum
for right in range(k, len(arr)):
    window_sum += arr[right] - arr[right - k]   # add new, drop old
    best = max(best, window_sum)
```

---

## Interview Tips

1. **"Contiguous subarray/substring" → sliding window** — the trigger phrase
2. **Expand right, contract left** — the universal structure
3. **Each element enters and leaves once → O(n)** — explain why it's linear, not O(n²)
4. **Track window state efficiently** — a count/sum/map updated in O(1), not recomputed
5. **Check "inside the window"** — in substring problems, only react to repeats within `[left, right]`
6. **Fixed vs variable** — fixed-k problems just slide; variable problems grow/shrink on a condition

---

## Common Pitfalls

1. **Recomputing the window each step** — that's O(n²); update incrementally
2. **Forgetting to check the repeat is inside the window** (Longest Substring: `>= left`)
3. **Not handling duplicates in `t`** (Minimum Window needs counts, not just presence)
4. **Applying it to non-contiguous problems** — sliding window is for consecutive elements only
5. **Off-by-one in window length** — it's `right - left + 1`
