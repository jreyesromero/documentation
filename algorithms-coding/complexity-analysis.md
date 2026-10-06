# Complexity Analysis (Big-O)

How to measure and communicate the efficiency of your code. Every coding interview expects you to state time and space complexity — and the study guide explicitly lists "measuring algorithm complexity" as a called-out topic.

## What Big-O Measures

Big-O describes how an algorithm's **runtime or memory grows** as the input size (n) grows. It's about the *rate of growth*, not exact timings.

```
We care about the DOMINANT term as n → ∞.

O(2n + 10)      → O(n)      (drop constants and lower terms)
O(n² + n)       → O(n²)     (keep the dominant term)
O(500)          → O(1)      (constant, regardless of input)
```

**Two things we always state:**
- **Time complexity** — how runtime grows
- **Space complexity** — how extra memory grows (not counting the input)

---

## The Complexity Hierarchy (Best to Worst)

| Big-O | Name | Example | n=10 | n=1000 |
|-------|------|---------|------|--------|
| **O(1)** | Constant | Hash lookup, array index | 1 | 1 |
| **O(log n)** | Logarithmic | Binary search | ~3 | ~10 |
| **O(n)** | Linear | Single loop | 10 | 1,000 |
| **O(n log n)** | Linearithmic | Efficient sorting | ~33 | ~10,000 |
| **O(n²)** | Quadratic | Nested loops | 100 | 1,000,000 |
| **O(2ⁿ)** | Exponential | Recursive subsets | 1,024 | astronomical |
| **O(n!)** | Factorial | Permutations | 3.6M | unthinkable |

```
Fast  O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(2ⁿ) < O(n!)  Slow
```

**The mental image:**
```
O(1)        ▁ flat — doesn't grow
O(log n)    ▂ grows very slowly
O(n)        ▃ grows steadily (straight line)
O(n log n)  ▅ grows a bit faster than linear
O(n²)       ▇ grows fast (curve up)
O(2ⁿ)       █ explodes
```

---

## How to Recognize Each Complexity

### O(1) - Constant

No loops over the input; the work is fixed.

```python
def get_first(arr):
    return arr[0]          # one operation, regardless of size

def add(a, b):
    return a + b           # constant
```

**Signals:** Hash map get/set, array index, arithmetic, a fixed number of steps.

### O(log n) - Logarithmic

The input is **halved** each step.

```python
def binary_search(arr, target):
    lo, hi = 0, len(arr) - 1
    while lo <= hi:
        mid = (lo + hi) // 2       # halve the search space each iteration
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return -1
```

**Signals:** Binary search, balanced tree operations, "halve each time," divide-and-conquer where you discard half.

### O(n) - Linear

Visit each element a constant number of times.

```python
def find_max(arr):
    m = arr[0]
    for x in arr:              # one pass through n elements
        if x > m:
            m = x
    return m
```

**Signals:** A single loop over the input, one pass.

### O(n log n) - Linearithmic

Usually a sort, or doing O(log n) work for each of n elements.

```python
sorted_arr = sorted(arr)      # Python's sort is O(n log n)
```

**Signals:** Efficient sorting (merge sort, quicksort, Timsort), "sort then process."

### O(n²) - Quadratic

For each element, you loop over all elements again — nested loops.

```python
def has_pair_brute(arr, target):
    for i in range(len(arr)):          # n iterations
        for j in range(i+1, len(arr)): #   × n iterations
            if arr[i] + arr[j] == target:
                return True
    return False
```

**Signals:** Nested loops over the same input, comparing all pairs. **Often a sign you can optimize** (e.g., with a hash map to get O(n)).

### O(2ⁿ) - Exponential

Each step branches into two; you explore all subsets/combinations.

```python
def fib(n):                   # naive recursive Fibonacci
    if n <= 1:
        return n
    return fib(n-1) + fib(n-2)   # two recursive calls each level
```

**Signals:** Recursion with multiple branches, generating all subsets, naive recursive solutions. **Usually needs memoization/DP to fix.**

---

## Common Patterns → Complexity

| Pattern | Time | Why |
|---------|------|-----|
| Hash map lookup | O(1) | Direct access |
| Single loop | O(n) | One pass |
| Two nested loops | O(n²) | n × n |
| Loop + binary search inside | O(n log n) | n × log n |
| Two pointers (one pass) | O(n) | Each moves ≤ n |
| Sliding window | O(n) | Each element enters/exits once |
| Sorting | O(n log n) | Comparison sort limit |
| BFS / DFS on graph | O(V + E) | Visit vertices + edges |
| Recursion halving | O(log n) | Halve each call |

---

## Space Complexity

How much **extra** memory you use (beyond the input).

```python
# O(1) space — a few variables, no growth with input
def sum_list(arr):
    total = 0                 # one variable
    for x in arr:
        total += x
    return total

# O(n) space — a data structure that grows with input
def unique(arr):
    seen = set()              # grows up to n elements
    for x in arr:
        seen.add(x)
    return seen
```

**Common space costs:**
- Hash map / set storing n items → O(n)
- Recursion depth of n → O(n) (call stack)
- A fixed number of variables → O(1)
- A 2D grid of size n×n → O(n²)

**Interview note:** Recursion uses stack space. A recursive solution with depth n is O(n) space even if it looks like it uses no extra memory.

---

## The Classic Trade-Off: Time vs Space

Most optimizations trade one for the other. Verbalizing this is gold:

```
Two Sum:
  Brute force:  O(n²) time, O(1) space   (check every pair)
  Hash map:     O(n) time,  O(n) space   (store seen numbers)

  "I trade O(n) space for a huge time improvement — usually worth it."
```

```
Sorted array search:
  Hash set:     O(n) time,  O(n) space
  Two pointers: O(n) time,  O(1) space   (if already sorted)

  "If the array's sorted, two pointers gives the same time with no 
   extra space."
```

---

## Numbers Every Programmer Should Know

The study guide explicitly lists this (latency numbers). You don't need exact figures, but know the **relative orders of magnitude** — it shows systems awareness.

| Operation | Approximate Latency | Relative |
|-----------|--------------------|----------| 
| L1 cache reference | ~1 ns | 1× |
| L2 cache reference | ~4 ns | 4× |
| Main memory (RAM) reference | ~100 ns | 100× |
| SSD random read | ~16,000 ns (16 µs) | 16,000× |
| Read 1 MB sequentially from memory | ~3,000 ns (3 µs) | — |
| Read 1 MB from SSD | ~49,000 ns (49 µs) | — |
| Round trip within same datacenter | ~500,000 ns (0.5 ms) | — |
| Disk (HDD) seek | ~2,000,000 ns (2 ms) | — |
| Read 1 MB from disk | ~825,000 ns (0.8 ms) | — |
| Network round trip CA ↔ Netherlands | ~150,000,000 ns (150 ms) | — |

**The key takeaways to internalize:**
```
Memory is ~100× slower than L1 cache
SSD is ~100× slower than memory
Disk seek is ~100× slower than SSD
Cross-continent network is the slowest by far (~150ms)
```

**Why it matters for an SRE:** When something is slow, this intuition tells you where to look — a cross-datacenter call (0.5ms) or disk seek (2ms) dwarfs in-memory work (ns). Latency problems usually live at I/O and network boundaries, not in CPU-bound code.

---

## How to State Complexity in an Interview

```
"The time complexity is O(n) because I make a single pass through the 
 array, and each hash map operation is O(1) on average.

 The space complexity is O(n) because in the worst case the hash map 
 stores every element before I find the answer."
```

**Always:**
1. State the Big-O
2. **Justify it** (why that growth rate)
3. Mention average vs worst case if relevant (hash maps are O(1) *average*)

---

## Interview Tips

1. **Always state both time AND space** — interviewers expect both
2. **Justify, don't just declare** — "O(n) because single pass"
3. **Drop constants** — O(2n) is O(n); O(n/2) is O(n)
4. **Nested loops = O(n²)** — and usually a hint to optimize
5. **Halving = O(log n)** — binary search, divide and conquer
6. **Recursion costs stack space** — depth n = O(n) space
7. **Know the latency orders of magnitude** — memory < SSD < disk < network

---

## Common Pitfalls

1. **Forgetting space complexity** — always state it too
2. **Not dropping constants** — O(3n) should be stated as O(n)
3. **Ignoring recursion stack space** — it's not free
4. **Confusing average and worst case** — hash maps are O(1) average, O(n) worst
5. **Saying "fast" instead of a Big-O** — be precise
6. **Missing that nested loops over DIFFERENT inputs are O(n×m)**, not O(n²)
