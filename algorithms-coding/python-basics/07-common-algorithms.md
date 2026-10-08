# 07: Common Algorithms

Essential algorithms for interviews: searching, sorting, and useful patterns.

## Linear Search

Find an element in an unsorted list.

```python
def linear_search(items, target):
    """Find target in list, return index or -1"""
    for i, item in enumerate(items):
        if item == target:
            return i
    return -1

linear_search([1, 5, 3, 7, 2], 7)  # 3
linear_search([1, 5, 3], 10)        # -1

# Time: O(n), Space: O(1)
```

## Binary Search

Find an element in a **sorted** list in O(log n) time.

```python
def binary_search(items, target):
    """Find target in sorted list using binary search"""
    left, right = 0, len(items) - 1
    
    while left <= right:
        mid = (left + right) // 2
        
        if items[mid] == target:
            return mid
        elif items[mid] < target:
            left = mid + 1     # Search right half
        else:
            right = mid - 1    # Search left half
    
    return -1

binary_search([1, 3, 5, 7, 9], 7)  # 3
binary_search([1, 3, 5, 7, 9], 4)  # -1

# Time: O(log n), Space: O(1)
```

## Sorting

Python's `sorted()` is highly optimized (Timsort). For interviews, know the concepts.

```python
# Built-in sorting
numbers = [3, 1, 4, 1, 5]
sorted_nums = sorted(numbers)       # [1, 1, 3, 4, 5] (new list)
numbers.sort()                      # In-place sort

# Sort with key
people = [{"name": "Alice", "age": 30}, {"name": "Bob", "age": 25}]
sorted_by_age = sorted(people, key=lambda p: p["age"])

# Reverse
sorted(numbers, reverse=True)       # [5, 4, 3, 1, 1]

# Time: O(n log n), Space: O(n)
```

## Two-Pointer Technique

Efficiently find pairs or process arrays.

```python
def two_sum_sorted(nums, target):
    """Find two numbers that sum to target (assumes sorted)"""
    left, right = 0, len(nums) - 1
    
    while left < right:
        current_sum = nums[left] + nums[right]
        
        if current_sum == target:
            return [left, right]
        elif current_sum < target:
            left += 1          # Need larger sum
        else:
            right -= 1         # Need smaller sum
    
    return []

two_sum_sorted([2, 7, 11, 15], 9)   # [0, 1]
two_sum_sorted([1, 3, 5, 7], 10)    # [2, 3]

# Time: O(n), Space: O(1)
```

### Valid Palindrome

```python
def is_palindrome(s):
    """Check if string is palindrome (ignore spaces, case)"""
    s = s.lower()
    left, right = 0, len(s) - 1
    
    while left < right:
        # Skip non-alphanumeric
        if not s[left].isalnum():
            left += 1
            continue
        if not s[right].isalnum():
            right -= 1
            continue
        
        if s[left] != s[right]:
            return False
        
        left += 1
        right -= 1
    
    return True

is_palindrome("A man, a plan, a canal: Panama")  # True
is_palindrome("race a car")                       # False
```

### Merge Sorted Arrays

```python
def merge_sorted(arr1, arr2):
    """Merge two sorted arrays into one sorted array"""
    result = []
    i = j = 0
    
    while i < len(arr1) and j < len(arr2):
        if arr1[i] <= arr2[j]:
            result.append(arr1[i])
            i += 1
        else:
            result.append(arr2[j])
            j += 1
    
    # Add remaining
    result.extend(arr1[i:])
    result.extend(arr2[j:])
    
    return result

merge_sorted([1, 3, 5], [2, 4, 6])  # [1, 2, 3, 4, 5, 6]

# Time: O(n + m), Space: O(n + m)
```

## Sliding Window

Find optimal subarray/substring of fixed or variable size.

```python
def max_product_subarray(nums, k):
    """Find max sum of any k consecutive elements"""
    if k > len(nums):
        return None
    
    # Calculate sum of first window
    window_sum = sum(nums[:k])
    max_sum = window_sum
    
    # Slide the window
    for i in range(k, len(nums)):
        window_sum = window_sum - nums[i - k] + nums[i]
        max_sum = max(max_sum, window_sum)
    
    return max_sum

max_product_subarray([1, 3, 2, 6, -1, 4], 3)  # 11 (6 + -1 + 4 = 9, but 3 + 2 + 6 = 11)

# Time: O(n), Space: O(1)
```

## Frequency Counter (Hash Map Pattern)

```python
def has_duplicates(items):
    """Check if list has any duplicates"""
    seen = set()
    for item in items:
        if item in seen:
            return True
        seen.add(item)
    return False

has_duplicates([1, 2, 3, 2])  # True
has_duplicates([1, 2, 3, 4])  # False

# Or simpler:
def has_duplicates_v2(items):
    return len(items) != len(set(items))
```

### Find Most Frequent Element

```python
def most_frequent(items):
    """Return element that appears most often"""
    freq = {}
    for item in items:
        freq[item] = freq.get(item, 0) + 1
    
    return max(freq, key=freq.get)

most_frequent([1, 1, 1, 2, 2, 3])  # 1

# Time: O(n), Space: O(n)
```

## Exercises

### Exercise 1: Search in Rotated Array
Write a function that searches a rotated sorted array.

```python
def search_rotated(nums, target):
    """Find target in rotated sorted array"""
    pass

# Test cases:
assert search_rotated([4, 5, 6, 7, 0, 1, 2], 0) == 4
assert search_rotated([4, 5, 6, 7, 0, 1, 2], 3) == -1
assert search_rotated([1], 1) == 0
```

**Hint:** Binary search, but determine which half is sorted first.

**Solution:**
```python
def search_rotated(nums, target):
    left, right = 0, len(nums) - 1
    
    while left <= right:
        mid = (left + right) // 2
        
        if nums[mid] == target:
            return mid
        
        # Determine which half is sorted
        if nums[left] <= nums[mid]:
            # Left half is sorted
            if nums[left] <= target < nums[mid]:
                right = mid - 1  # Target in left
            else:
                left = mid + 1   # Target in right
        else:
            # Right half is sorted
            if nums[mid] < target <= nums[right]:
                left = mid + 1   # Target in right
            else:
                right = mid - 1  # Target in left
    
    return -1
```

### Exercise 2: Container With Most Water
Given heights of containers, find the maximum area.

```python
def max_area(heights):
    """Find max area between two lines"""
    pass

# Test case:
assert max_area([1, 8, 6, 2, 5, 4, 8, 3, 7]) == 49  # Between 8 and 7
```

**Solution:**
```python
def max_area(heights):
    left, right = 0, len(heights) - 1
    max_area = 0
    
    while left < right:
        # Width is the distance between pointers
        width = right - left
        # Height is the minimum of the two
        height = min(heights[left], heights[right])
        area = width * height
        
        max_area = max(max_area, area)
        
        # Move the pointer pointing to shorter line
        if heights[left] < heights[right]:
            left += 1
        else:
            right -= 1
    
    return max_area
```

### Exercise 3: Remove Duplicates
Write a function that removes duplicates from a sorted array **in-place**.

```python
def remove_duplicates(nums):
    """Remove duplicates and return new length"""
    pass

# Test case:
nums = [0, 0, 1, 1, 1, 2, 2, 3, 3, 4]
length = remove_duplicates(nums)
assert length == 5
assert nums[:length] == [0, 1, 2, 3, 4]
```

**Solution:**
```python
def remove_duplicates(nums):
    if not nums:
        return 0
    
    write_pos = 1  # Position to write next unique element
    
    for read_pos in range(1, len(nums)):
        if nums[read_pos] != nums[read_pos - 1]:
            nums[write_pos] = nums[read_pos]
            write_pos += 1
    
    return write_pos

# Time: O(n), Space: O(1)
```

## Key Takeaways

✅ Linear search: O(n), use for small/unsorted lists  
✅ Binary search: O(log n), requires sorted input  
✅ Two-pointer technique: efficient for sorted arrays  
✅ Sliding window: optimal for subarray/substring problems  
✅ Hash maps (dicts): fast frequency counting  
✅ Python's `sorted()` is O(n log n) and highly optimized  
✅ Always consider time and space complexity  
✅ In-place algorithms modify the input (space-efficient)  

## Next Module

[SRE Patterns](./08-sre-patterns.md) — APIs, command-line tools, environment variables, timestamps
