# Pattern: Binary Search & Linked Lists

Two foundational patterns. **Binary Search** finds things in O(log n) by halving a sorted search space. **Linked Lists** test pointer manipulation — the ability to re-wire `next` references carefully.

---

# Part 1: Binary Search

## When to Recognize This Pattern

```
"The array is SORTED and you need to find/locate..."   → binary search
"Find the boundary / first/last position of..."        → binary search
"Minimize/maximize a value with a monotonic check..."  → binary search on answer
"O(log n) is expected..."                              → binary search
```

**The core insight:** Each comparison eliminates **half** the remaining candidates. That's why it's O(log n) — you can halve n only ~log₂(n) times.

```
n=1,000,000 → ~20 steps to find anything
n=1,000,000,000 → ~30 steps
```

---

## Problem 1: Binary Search (#704, Easy)

**Statement:** Given a sorted array and a target, return its index, or -1 if not present.

```
Input:  nums = [-1,0,3,5,9,12], target = 9   → 4
Input:  nums = [-1,0,3,5,9,12], target = 2   → -1
```

**Approach:** Track `lo` and `hi` bounds. Check the middle. If it's the target, done. If the middle is too small, discard the left half; too big, discard the right half.

```python
def search(nums, target):
    lo, hi = 0, len(nums) - 1
    while lo <= hi:
        mid = (lo + hi) // 2           # middle index
        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            lo = mid + 1               # target is in the right half
        else:
            hi = mid - 1               # target is in the left half
    return -1
```

**Trace** `[-1,0,3,5,9,12]`, target 9:
```
lo=0, hi=5, mid=2 (val 3):  3 < 9 → lo=3
lo=3, hi=5, mid=4 (val 9):  9 == 9 → return 4 ✓
```

**Complexity:** Time O(log n), Space O(1).

**The three must-get-right details:**
```
1. while lo <= hi      (use <=, so a single element is still checked)
2. mid = (lo + hi)//2  (integer division)
3. lo = mid + 1 / hi = mid - 1  (always move past mid, or you loop forever)
```

---

## Problem 2: Search in Rotated Sorted Array (#33, Medium)

**Statement:** A sorted array was rotated at an unknown pivot (e.g., `[0,1,2,4,5,6,7]` → `[4,5,6,7,0,1,2]`). Find the target's index, or -1. Must be O(log n).

```
Input:  nums = [4,5,6,7,0,1,2], target = 0   → 4
Input:  nums = [4,5,6,7,0,1,2], target = 3   → -1
```

**Approach:** Still binary search, but at each step, **one half is always properly sorted.** Figure out which half is sorted, check if the target lies within it, and discard accordingly.

```python
def search(nums, target):
    lo, hi = 0, len(nums) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if nums[mid] == target:
            return mid

        # is the LEFT half sorted?
        if nums[lo] <= nums[mid]:
            # target in the sorted left half?
            if nums[lo] <= target < nums[mid]:
                hi = mid - 1
            else:
                lo = mid + 1
        # otherwise the RIGHT half is sorted
        else:
            if nums[mid] < target <= nums[hi]:
                lo = mid + 1
            else:
                hi = mid - 1
    return -1
```

**The idea:**
```
[4,5,6,7,0,1,2]
 lo    mid    hi
Left half [4,5,6,7] is sorted (nums[lo] <= nums[mid]).
If target is within [4..7), search left; else search right.
One half is ALWAYS sorted — that's what makes O(log n) possible.
```

**Complexity:** Time O(log n), Space O(1).

**Interview note:** This is the classic "binary search with a twist." The key realization to verbalize: *even rotated, one half is always sorted*, so you can still eliminate half each step.

---

## Binary Search Template

```python
lo, hi = 0, len(arr) - 1
while lo <= hi:
    mid = (lo + hi) // 2
    if arr[mid] == target:
        return mid
    elif arr[mid] < target:
        lo = mid + 1
    else:
        hi = mid - 1
return -1
```

---

# Part 2: Linked Lists

## When to Recognize This Pattern

```
"Reverse a linked list..."              → pointer re-wiring
"Merge two lists..."                    → dummy head + splice
"Detect a cycle..."                     → slow/fast pointers (Floyd's)
"Find the middle..."                    → slow/fast pointers
"Remove the Nth node..."                → two pointers, n apart
```

**The core insight:** Linked-list problems are about carefully re-pointing `next` references without losing track of nodes. Two tools dominate: the **dummy head** (simplifies edge cases) and **slow/fast pointers** (Floyd's cycle detection).

```python
# Node definition (given in problems)
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next
```

---

## Problem 3: Reverse Linked List (#206, Easy)

**Statement:** Reverse a singly linked list.

```
Input:  1 → 2 → 3 → 4 → 5 → None
Output: 5 → 4 → 3 → 2 → 1 → None
```

**Approach:** Walk the list, flipping each node's `next` to point backward. Track `prev` (the reversed part) and `curr` (what's left).

```python
def reverseList(head):
    prev = None
    curr = head
    while curr:
        next_node = curr.next     # save the next node (don't lose it!)
        curr.next = prev          # reverse this node's pointer
        prev = curr               # advance prev
        curr = next_node          # advance curr
    return prev                   # prev is the new head
```

**Trace** `1→2→3`:
```
Start:  prev=None, curr=1
Step 1: save 2; 1.next=None; prev=1, curr=2      → None←1
Step 2: save 3; 2.next=1;    prev=2, curr=3      → None←1←2
Step 3: save None; 3.next=2; prev=3, curr=None   → None←1←2←3
Return prev=3 → 3→2→1→None ✓
```

**Complexity:** Time O(n), Space O(1).

**The critical detail:** Save `curr.next` *before* overwriting it — otherwise you lose the rest of the list. This is the #1 linked-list bug.

---

## Problem 4: Merge Two Sorted Lists (#21, Easy)

**Statement:** Merge two sorted linked lists into one sorted list.

```
Input:  list1 = 1→2→4,  list2 = 1→3→4
Output: 1→1→2→3→4→4
```

**Approach:** Use a **dummy head** to avoid special-casing the first node. Walk both lists, attaching the smaller node each time.

```python
def mergeTwoLists(list1, list2):
    dummy = ListNode()            # placeholder to simplify edge cases
    tail = dummy
    while list1 and list2:
        if list1.val <= list2.val:
            tail.next = list1
            list1 = list1.next
        else:
            tail.next = list2
            list2 = list2.next
        tail = tail.next
    # attach whatever remains (one list may be non-empty)
    tail.next = list1 if list1 else list2
    return dummy.next             # skip the dummy, return the real head
```

**Why the dummy head?** Without it, you'd need special logic to set the first node. The dummy gives `tail` something to attach to from the start; you return `dummy.next` at the end.

**Trace** `1→2→4` and `1→3→4`:
```
compare 1,1 → attach list1's 1; compare 2,1 → attach list2's 1;
compare 2,3 → attach 2; compare 4,3 → attach 3; compare 4,4 → attach 4;
list1 has 4 left → attach remainder
→ 1→1→2→3→4→4 ✓
```

**Complexity:** Time O(n + m), Space O(1) — we re-link existing nodes, no new ones.

---

## Bonus: Floyd's Cycle Detection (slow/fast)

The study guide lists cycle detection explicitly. The technique: two pointers, one moving twice as fast. If there's a cycle, they meet; if `fast` reaches the end, there's no cycle.

```python
def hasCycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next          # 1 step
        fast = fast.next.next     # 2 steps
        if slow == fast:          # they met → cycle
            return True
    return False                  # fast hit the end → no cycle
```

**Why it works:** In a cycle, the fast pointer "laps" the slow one and they collide. Without a cycle, fast reaches `None`. O(n) time, O(1) space — no extra set needed.

---

## Interview Tips

1. **Sorted + O(log n) expected → binary search** — the trigger
2. **Binary search: `lo <= hi`, `mid±1`** — the off-by-one details that cause infinite loops
3. **Rotated array: one half is always sorted** — the key insight for #33
4. **Linked list: save `next` before re-pointing** — the #1 bug to avoid
5. **Dummy head simplifies list construction** — no special case for the first node
6. **Slow/fast pointers for cycle detection** — Floyd's, O(1) space
7. **Draw the pointers** — linked-list problems are much easier with a sketch

---

## Common Pitfalls

**Binary Search:**
1. **`lo < hi` vs `lo <= hi`** — use `<=` or you miss single-element cases
2. **`mid` not moving** — always `lo = mid+1` / `hi = mid-1`, or infinite loop
3. **Integer overflow** — in other languages use `lo + (hi-lo)//2`; Python ints are safe

**Linked Lists:**
4. **Losing the rest of the list** — save `curr.next` before overwriting
5. **Not using a dummy head** — leads to messy first-node special cases
6. **Null pointer errors** — check `fast and fast.next` before `fast.next.next`
