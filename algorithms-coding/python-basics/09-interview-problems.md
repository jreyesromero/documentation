# 09: Interview Problems

15+ medium difficulty problems to practice. Solve these **without looking at solutions first**.

## Quick Start

For each problem:
1. Read and understand (2 min)
2. Plan approach, discuss trade-offs (3 min)
3. Write code, handle edge cases (15 min)
4. Test with examples (5 min)

**Target:** Complete each problem in 20-25 minutes.

---

## Problem 1: Two Sum

Given an array of integers and a target, return indices of the two numbers that add up to target. Assume each input has exactly one solution.

```python
def two_sum(nums, target):
    """
    Example:
    nums = [2, 7, 11, 15], target = 9
    Returns: [0, 1]  (2 + 7 = 9)
    """
    pass

# Test cases:
assert two_sum([2, 7, 11, 15], 9) == [0, 1]
assert two_sum([3, 2, 4], 6) == [1, 2]
assert two_sum([3, 3], 6) == [0, 1]
```

**Solution:**
```python
def two_sum(nums, target):
    seen = {}
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
    return []
# Time: O(n), Space: O(n)
```

---

## Problem 2: Valid Parentheses

Given a string with parentheses, determine if they're balanced.

```python
def is_valid(s):
    """
    Example:
    s = "([])" → True
    s = "([)]" → False
    """
    pass

# Test cases:
assert is_valid("()") == True
assert is_valid("()[]{}") == True
assert is_valid("([)]") == False
assert is_valid("") == True
```

**Solution:**
```python
def is_valid(s):
    stack = []
    pairs = {"(": ")", "[": "]", "{": "}"}
    
    for char in s:
        if char in pairs:
            stack.append(char)
        else:
            if not stack or pairs[stack.pop()] != char:
                return False
    
    return len(stack) == 0
# Time: O(n), Space: O(n)
```

---

## Problem 3: Merge Two Sorted Lists

Merge two sorted arrays into one sorted array.

```python
def merge(list1, list2):
    """
    Example:
    list1 = [1, 3, 5], list2 = [2, 4, 6]
    Returns: [1, 2, 3, 4, 5, 6]
    """
    pass

# Test cases:
assert merge([1, 3, 5], [2, 4, 6]) == [1, 2, 3, 4, 5, 6]
assert merge([], [1, 2]) == [1, 2]
assert merge([1], [1]) == [1, 1]
```

**Solution:**
```python
def merge(list1, list2):
    result = []
    i = j = 0
    
    while i < len(list1) and j < len(list2):
        if list1[i] <= list2[j]:
            result.append(list1[i])
            i += 1
        else:
            result.append(list2[j])
            j += 1
    
    result.extend(list1[i:])
    result.extend(list2[j:])
    return result
# Time: O(n + m), Space: O(n + m)
```

---

## Problem 4: Contains Duplicate

Determine if a list contains any duplicates.

```python
def contains_duplicate(nums):
    """
    Example:
    nums = [1, 2, 3, 1] → True
    nums = [1, 2, 3, 4] → False
    """
    pass

# Test cases:
assert contains_duplicate([1, 2, 3, 1]) == True
assert contains_duplicate([1, 2, 3, 4]) == False
assert contains_duplicate([]) == False
```

**Solution:**
```python
def contains_duplicate(nums):
    return len(nums) != len(set(nums))

# Or more explicit:
def contains_duplicate_v2(nums):
    seen = set()
    for num in nums:
        if num in seen:
            return True
        seen.add(num)
    return False
# Time: O(n), Space: O(n)
```

---

## Problem 5: Best Time to Buy and Sell Stock

Given prices, find max profit from buying and selling once. Must buy before selling.

```python
def max_profit(prices):
    """
    Example:
    prices = [7, 1, 5, 3, 6, 4]
    Best: buy at 1, sell at 6 → profit = 5
    """
    pass

# Test cases:
assert max_profit([7, 1, 5, 3, 6, 4]) == 5
assert max_profit([7, 6, 4, 3, 1]) == 0
assert max_profit([2, 4, 1, 7, 5, 11]) == 10  # buy 1, sell 11
```

**Solution:**
```python
def max_profit(prices):
    if not prices or len(prices) < 2:
        return 0
    
    min_price = prices[0]
    max_profit = 0
    
    for price in prices[1:]:
        max_profit = max(max_profit, price - min_price)
        min_price = min(min_price, price)
    
    return max_profit
# Time: O(n), Space: O(1)
```

---

## Problem 6: Group Anagrams

Group words that are anagrams of each other.

```python
def group_anagrams(words):
    """
    Example:
    words = ["eat", "tea", "ate", "tan", "nat"]
    Returns: [["eat", "tea", "ate"], ["tan", "nat"]]
    """
    pass

# Test cases:
result = group_anagrams(["eat", "tea", "ate"])
assert len(result) == 1
assert set(map(tuple, result)) == {("eat", "tea", "ate")}
```

**Solution:**
```python
def group_anagrams(words):
    from collections import defaultdict
    groups = defaultdict(list)
    
    for word in words:
        key = "".join(sorted(word))  # Anagrams have same sorted form
        groups[key].append(word)
    
    return list(groups.values())
# Time: O(n * k log k) where k is avg word length, Space: O(n)
```

---

## Problem 7: Longest Substring Without Repeating Characters

Find the length of the longest substring with no repeating characters.

```python
def length_of_longest_substring(s):
    """
    Example:
    s = "abcabcbb" → 3 ("abc")
    s = "bbbbb" → 1 ("b")
    """
    pass

# Test cases:
assert length_of_longest_substring("abcabcbb") == 3
assert length_of_longest_substring("bbbbb") == 1
assert length_of_longest_substring("pwwkew") == 3
assert length_of_longest_substring("") == 0
```

**Solution:**
```python
def length_of_longest_substring(s):
    char_index = {}
    max_length = 0
    start = 0
    
    for end, char in enumerate(s):
        if char in char_index and char_index[char] >= start:
            start = char_index[char] + 1
        
        char_index[char] = end
        max_length = max(max_length, end - start + 1)
    
    return max_length
# Time: O(n), Space: O(min(n, 26)) for ASCII
```

---

## Problem 8: Reverse a List

Reverse a list in-place without using `reverse()`.

```python
def reverse_list(nums):
    """
    Example:
    nums = [1, 2, 3, 4, 5]
    After: nums = [5, 4, 3, 2, 1]
    """
    pass

# Test cases:
nums = [1, 2, 3, 4, 5]
reverse_list(nums)
assert nums == [5, 4, 3, 2, 1]
```

**Solution:**
```python
def reverse_list(nums):
    left, right = 0, len(nums) - 1
    while left < right:
        nums[left], nums[right] = nums[right], nums[left]
        left += 1
        right -= 1
# Time: O(n), Space: O(1)
```

---

## Problem 9: 3Sum

Find all unique triplets that sum to zero.

```python
def three_sum(nums):
    """
    Example:
    nums = [-1, 0, 1, 2, -1, -4]
    Returns: [[-1, -1, 2], [-1, 0, 1]]
    """
    pass

# Test cases:
result = three_sum([-1, 0, 1, 2, -1, -4])
assert len(result) == 2
assert [-1, -1, 2] in result and [-1, 0, 1] in result
```

**Solution:**
```python
def three_sum(nums):
    nums.sort()
    result = []
    
    for i in range(len(nums) - 2):
        if nums[i] > 0:
            break
        if i > 0 and nums[i] == nums[i-1]:
            continue  # Skip duplicates
        
        left, right = i + 1, len(nums) - 1
        while left < right:
            total = nums[i] + nums[left] + nums[right]
            if total == 0:
                result.append([nums[i], nums[left], nums[right]])
                while left < right and nums[left] == nums[left+1]:
                    left += 1
                while left < right and nums[right] == nums[right-1]:
                    right -= 1
                left += 1
                right -= 1
            elif total < 0:
                left += 1
            else:
                right -= 1
    
    return result
# Time: O(n²), Space: O(1) (excluding result)
```

---

## Problem 10: Top K Frequent Elements

Find the K most frequent elements in a list.

```python
def top_k_frequent(nums, k):
    """
    Example:
    nums = [1,1,1,2,2,3], k = 2
    Returns: [1, 2]
    """
    pass

# Test cases:
assert set(top_k_frequent([1,1,1,2,2,3], 2)) == {1, 2}
assert top_k_frequent([4,1,1,1,2,2,3], 2) == [1, 2]
```

**Solution:**
```python
def top_k_frequent(nums, k):
    from collections import Counter
    count = Counter(nums)
    return [num for num, _ in count.most_common(k)]

# Or without Counter:
def top_k_frequent_v2(nums, k):
    freq = {}
    for num in nums:
        freq[num] = freq.get(num, 0) + 1
    
    return sorted(freq.keys(), key=lambda x: freq[x], reverse=True)[:k]
# Time: O(n log n), Space: O(n)
```

---

## Problem 11: Valid Palindrome II

Check if string is palindrome, allowing deletion of at most one character.

```python
def is_palindrome_with_delete(s):
    """
    Example:
    s = "aba" → True
    s = "abca" → True (delete 'c')
    s = "abc" → False
    """
    pass

# Test cases:
assert is_palindrome_with_delete("aba") == True
assert is_palindrome_with_delete("abca") == True
assert is_palindrome_with_delete("abc") == False
```

**Solution:**
```python
def is_palindrome_with_delete(s):
    def is_palindrome(l, r):
        while l < r:
            if s[l] != s[r]:
                return False
            l += 1
            r -= 1
        return True
    
    left, right = 0, len(s) - 1
    while left < right:
        if s[left] != s[right]:
            # Try deleting left or right
            return is_palindrome(left + 1, right) or is_palindrome(left, right - 1)
        left += 1
        right -= 1
    
    return True
# Time: O(n), Space: O(1)
```

---

## Problem 12: Product of Array Except Self

Create array where each element is product of all other elements (no division).

```python
def product_except_self(nums):
    """
    Example:
    nums = [1, 2, 3, 4]
    Returns: [24, 12, 8, 6]
    (1st: 2*3*4, 2nd: 1*3*4, etc.)
    """
    pass

# Test cases:
assert product_except_self([1,2,3,4]) == [24,12,8,6]
assert product_except_self([-1,1,0,-3,3]) == [0,0,9,0,0]
```

**Solution:**
```python
def product_except_self(nums):
    n = len(nums)
    result = [1] * n
    
    # Left pass: result[i] = product of all elements to the left
    left = 1
    for i in range(n):
        result[i] = left
        left *= nums[i]
    
    # Right pass: multiply by product of all elements to the right
    right = 1
    for i in range(n-1, -1, -1):
        result[i] *= right
        right *= nums[i]
    
    return result
# Time: O(n), Space: O(1) (excluding result)
```

---

## Problem 13: LRU Cache

Implement a Least Recently Used (LRU) cache.

```python
class LRUCache:
    def __init__(self, capacity):
        pass
    
    def get(self, key):
        """Return value if exists, -1 otherwise"""
        pass
    
    def put(self, key, value):
        """Set value, evict LRU item if at capacity"""
        pass

# Test case:
cache = LRUCache(2)
cache.put(1, 1)
cache.put(2, 2)
assert cache.get(1) == 1
cache.put(3, 3)  # Evicts key 2
assert cache.get(2) == -1
```

**Solution:**
```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity):
        self.capacity = capacity
        self.cache = OrderedDict()
    
    def get(self, key):
        if key not in self.cache:
            return -1
        # Move to end (most recently used)
        self.cache.move_to_end(key)
        return self.cache[key]
    
    def put(self, key, value):
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        
        if len(self.cache) > self.capacity:
            # Remove oldest (first item)
            self.cache.popitem(last=False)
# Time: O(1) for both get and put
```

---

## Problem 14: Trapping Rain Water

Given elevation map, calculate trapped rainwater.

```python
def trap(heights):
    """
    Example:
    heights = [0,1,0,2,1,0,1,3,2,1,2,1]
    Returns: 6 (water trapped between peaks)
    """
    pass

# Test cases:
assert trap([0,1,0,2,1,0,1,3,2,1,2,1]) == 6
assert trap([4,2,0,3,2,5]) == 9
```

**Solution:**
```python
def trap(heights):
    if not heights:
        return 0
    
    left, right = 0, len(heights) - 1
    left_max, right_max = 0, 0
    water = 0
    
    while left < right:
        if heights[left] < heights[right]:
            if heights[left] >= left_max:
                left_max = heights[left]
            else:
                water += left_max - heights[left]
            left += 1
        else:
            if heights[right] >= right_max:
                right_max = heights[right]
            else:
                water += right_max - heights[right]
            right -= 1
    
    return water
# Time: O(n), Space: O(1)
```

---

## Problem 15: Minimum Window Substring

Find minimum window substring that contains all chars from target.

```python
def min_window(s, t):
    """
    Example:
    s = "ADOBECODEBANC", t = "ABC"
    Returns: "BANC"
    """
    pass

# Test case:
assert min_window("ADOBECODEBANC", "ABC") == "BANC"
assert min_window("a", "a") == "a"
assert min_window("a", "aa") == ""
```

**Solution:**
```python
def min_window(s, t):
    if not s or not t:
        return ""
    
    dict_t = {}
    for char in t:
        dict_t[char] = dict_t.get(char, 0) + 1
    
    required = len(dict_t)
    formed = 0
    window_counts = {}
    
    l, r = 0, 0
    ans = float("inf"), None, None
    
    while r < len(s):
        char = s[r]
        window_counts[char] = window_counts.get(char, 0) + 1
        
        if char in dict_t and window_counts[char] == dict_t[char]:
            formed += 1
        
        while formed == required and l <= r:
            if r - l + 1 < ans[0]:
                ans = (r - l + 1, l, r)
            
            char = s[l]
            window_counts[char] -= 1
            if char in dict_t and window_counts[char] < dict_t[char]:
                formed -= 1
            l += 1
        
        r += 1
    
    return "" if ans[0] == float("inf") else s[ans[1]:ans[2]+1]
# Time: O(n + m), Space: O(n + m)
```

---

## Study Tips

1. **Do these in order** — problems progress in difficulty
2. **Don't peek at solutions** — struggle is where learning happens
3. **Time yourself** — aim for 20 mins per problem
4. **Discuss complexity** — state time and space for each solution
5. **Test edge cases** — empty input, single element, duplicates
6. **Review patterns** — each problem teaches a pattern you'll see again

---

## Interview Day Checklist

- [ ] Understand the problem (ask clarifying questions)
- [ ] Work through an example
- [ ] Discuss approach (brute force → optimized)
- [ ] State complexity before coding
- [ ] Write clean, readable code
- [ ] Test with edge cases
- [ ] Optimize if time permits

Good luck! 🚀
