# 02: Data Structures

The workhorses of Python: lists, dictionaries, sets, and tuples.

## Lists (Mutable, Ordered)

Lists are **ordered** and **mutable** (can be changed). They're the most common data structure.

```python
# Creating lists
numbers = [1, 2, 3, 4, 5]
mixed = [1, "hello", 3.14, True, None]
empty = []
from_range = list(range(5))      # [0, 1, 2, 3, 4]

# Accessing elements
numbers[0]        # 1 (first element)
numbers[-1]       # 5 (last element)
numbers[1:3]      # [2, 3] (slice, stop is exclusive)
numbers[:3]       # [1, 2, 3] (from start to index 2)
numbers[2:]       # [3, 4, 5] (from index 2 to end)
numbers[::2]      # [1, 3, 5] (every 2nd element)
numbers[::-1]     # [5, 4, 3, 2, 1] (reversed)
```

### List Methods (Mutating)

```python
nums = [1, 2, 3]

# Add elements
nums.append(4)           # [1, 2, 3, 4] (add to end)
nums.insert(1, 1.5)      # [1, 1.5, 2, 3, 4] (add at index)
nums.extend([5, 6])      # Add multiple (merge lists)

# Remove elements
nums.pop()               # Removes and returns last element
nums.pop(0)              # Removes and returns element at index 0
nums.remove(2)           # Removes first occurrence of value
nums.clear()             # Remove all elements

# Other operations
nums = [3, 1, 4, 1, 5]
nums.sort()              # [1, 1, 3, 4, 5] (in-place, modifies original)
nums.reverse()           # Reverse in-place
nums.count(1)            # 2 (count occurrences)
nums.index(4)            # 2 (first index of value)
```

### Important: Mutable Default Arguments (Gotcha!)

```python
# WRONG! Don't do this:
def add_item(item, items=[]):
    items.append(item)
    return items

# Each call shares the same list! The default is created once.
print(add_item(1))      # [1]
print(add_item(2))      # [1, 2] — the 1 is still there!
print(add_item(3))      # [1, 2, 3]

# RIGHT: Use None
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items

print(add_item(1))      # [1]
print(add_item(2))      # [2]
print(add_item(3))      # [3]
```

## Dictionaries (Mutable, Unordered Key-Value)

Dictionaries map **keys** to **values**. Keys must be unique and hashable (usually strings or ints).

```python
# Creating dictionaries
person = {"name": "Alice", "age": 25, "city": "NYC"}
empty = {}
from_pairs = dict([("a", 1), ("b", 2)])  # {a: 1, b: 2}

# Accessing values
person["name"]          # "Alice"
person.get("name")      # "Alice" (safer, returns None if not found)
person.get("job", "unknown")  # "unknown" (default value)

# Checking if key exists
"name" in person        # True
"job" in person         # False

# Adding/updating
person["age"] = 26      # Update existing key
person["job"] = "Engineer"  # Add new key
person.update({"city": "SF", "country": "USA"})

# Removing
del person["job"]       # Delete a key
person.pop("age")       # Remove and return value
person.pop("unknown", None)  # Return None if not found
person.clear()          # Remove all

# Iterating
for key in person:
    print(key, person[key])

for key, value in person.items():
    print(f"{key}: {value}")

for key in person.keys():
    print(key)

for value in person.values():
    print(value)
```

### Dictionary Comprehensions

```python
# Create a dict from a list
nums = [1, 2, 3, 4, 5]
squares = {x: x**2 for x in nums}
# {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}

# With condition
even_squares = {x: x**2 for x in nums if x % 2 == 0}
# {2: 4, 4: 16}

# From two lists
keys = ["a", "b", "c"]
values = [1, 2, 3]
d = dict(zip(keys, values))  # {"a": 1, "b": 2, "c": 3}
```

## Sets (Mutable, Unordered, Unique)

Sets store **unique** values and are unordered. Great for membership testing and removing duplicates.

```python
# Creating sets
numbers = {1, 2, 3, 3, 2, 1}  # {1, 2, 3} (duplicates removed)
empty = set()                  # Not {} (that's an empty dict)
from_list = set([1, 2, 2, 3])  # {1, 2, 3}

# Operations
3 in numbers             # True (fast lookup)
numbers.add(4)           # Add an element
numbers.remove(2)        # Remove, raises error if not found
numbers.discard(5)       # Remove, doesn't raise error
numbers.pop()            # Remove and return an arbitrary element

# Set operations
a = {1, 2, 3}
b = {2, 3, 4}
a.union(b)               # {1, 2, 3, 4}
a.intersection(b)        # {2, 3}
a.difference(b)          # {1}
a.symmetric_difference(b)  # {1, 4}

# Or use operators
a | b   # Union
a & b   # Intersection
a - b   # Difference
a ^ b   # Symmetric difference

# Common use: remove duplicates
numbers = [1, 2, 2, 3, 3, 3, 4]
unique = list(set(numbers))  # [1, 2, 3, 4] (order not guaranteed)
```

## Tuples (Immutable, Ordered)

Tuples are like lists but **immutable** (can't be changed). Great for fixed collections and dictionary keys.

```python
# Creating tuples
point = (10, 20)
single = (42,)              # Note the comma! (42) is just 42
empty = ()
colors = tuple(["red", "green", "blue"])

# Accessing (same as lists)
point[0]        # 10
point[-1]       # 20
point[0:1]      # (10,)

# Tuples are immutable
point[0] = 5    # TypeError! Can't change a tuple

# But you can create a new tuple
point = (5, 20)

# Unpacking
x, y = point    # x = 10, y = 20
a, *rest = (1, 2, 3, 4)  # a = 1, rest = [2, 3, 4]

# Using tuples as dict keys (because they're immutable)
locations = {
    (0, 0): "origin",
    (10, 20): "point A",
}

# Iterating
for item in point:
    print(item)
```

## Choosing the Right Data Structure

| Task | Use |
|------|-----|
| Ordered collection, may change | **List** |
| Key-value pairs | **Dictionary** |
| Unique values, fast lookup | **Set** |
| Fixed collection, immutable | **Tuple** |
| Dictionary key or set element | **Tuple** (must be hashable) |

## Exercises

### Exercise 1: Frequency Counter
Write a function that counts the frequency of each element in a list and returns a dictionary.

```python
def count_frequencies(items):
    pass

# Test cases:
assert count_frequencies([1, 2, 2, 3, 3, 3]) == {1: 1, 2: 2, 3: 3}
assert count_frequencies(["a", "b", "a"]) == {"a": 2, "b": 1}
assert count_frequencies([]) == {}
```

**Solution:**
```python
def count_frequencies(items):
    freq = {}
    for item in items:
        freq[item] = freq.get(item, 0) + 1
    return freq

# Or using dict.fromkeys() and list.count():
def count_frequencies_v2(items):
    return {item: items.count(item) for item in set(items)}
```

### Exercise 2: Merge Two Sorted Lists
Write a function that merges two sorted lists into one sorted list in O(n) time.

```python
def merge_sorted_lists(list1, list2):
    pass

# Test cases:
assert merge_sorted_lists([1, 3, 5], [2, 4, 6]) == [1, 2, 3, 4, 5, 6]
assert merge_sorted_lists([], [1, 2]) == [1, 2]
assert merge_sorted_lists([1, 2], []) == [1, 2]
```

**Solution:**
```python
def merge_sorted_lists(list1, list2):
    result = []
    i, j = 0, 0
    
    while i < len(list1) and j < len(list2):
        if list1[i] <= list2[j]:
            result.append(list1[i])
            i += 1
        else:
            result.append(list2[j])
            j += 1
    
    # Append remaining elements
    result.extend(list1[i:])
    result.extend(list2[j:])
    return result

# Simpler (but less educational):
def merge_sorted_lists_v2(list1, list2):
    return sorted(list1 + list2)
```

### Exercise 3: Dictionary Operations
Write a function that merges two dictionaries and returns a new one. If a key exists in both, sum the values.

```python
def merge_dicts(d1, d2):
    pass

# Test cases:
assert merge_dicts({"a": 1, "b": 2}, {"b": 3, "c": 4}) == {"a": 1, "b": 5, "c": 4}
assert merge_dicts({}, {"x": 1}) == {"x": 1}
assert merge_dicts({"a": 1}, {"a": 1}) == {"a": 2}
```

**Solution:**
```python
def merge_dicts(d1, d2):
    result = d1.copy()  # Start with copy of d1
    for key, value in d2.items():
        result[key] = result.get(key, 0) + value
    return result

# Or using dict comprehension:
def merge_dicts_v2(d1, d2):
    all_keys = set(d1.keys()) | set(d2.keys())
    return {key: d1.get(key, 0) + d2.get(key, 0) for key in all_keys}
```

## Key Takeaways

✅ Lists are mutable, ordered — use for dynamic collections  
✅ Dicts are key-value pairs — use for lookups  
✅ Sets are unique, unordered — use for membership and removing duplicates  
✅ Tuples are immutable — use as dict keys or for fixed data  
✅ List/dict comprehensions are cleaner than loops  
✅ **Mutable defaults are a gotcha** — always use `None` as default for mutable types  
✅ `.get()` on dicts is safer than `[]` (returns None instead of KeyError)  

## Next Module

[Functions & Scope](./03-functions-scope.md) — Defining functions, parameters, and variable scope
