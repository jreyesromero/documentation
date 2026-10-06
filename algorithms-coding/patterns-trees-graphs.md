# Pattern: Trees & Graphs (BFS / DFS)

Trees and graphs are traversed with two core techniques: **DFS** (depth-first, go deep, uses recursion/stack) and **BFS** (breadth-first, go level by level, uses a queue). Recognizing which to use is the key skill.

## DFS vs BFS — The Core Choice

| | DFS (Depth-First) | BFS (Breadth-First) |
|---|-------------------|---------------------|
| **Goes** | Deep first, then backtrack | Level by level, wide |
| **Uses** | Recursion or a stack | A queue |
| **Good for** | Path existence, "all the way down," tree depth | Shortest path, level-order, nearest |
| **Order** | Down one branch fully, then the next | All neighbors, then their neighbors |

```
Tree:        1
           /   \
          2     3
         / \
        4   5

DFS order:  1 → 2 → 4 → 5 → 3   (go deep)
BFS order:  1 → 2 → 3 → 4 → 5   (go level by level)
```

**Rule of thumb:**
- "Shortest path" or "level by level" → **BFS**
- "Does a path exist," "explore fully," "tree depth" → **DFS**

---

## Tree Traversal Orders (DFS)

The study guide calls out in-order, pre-order, post-order. All are DFS; they differ in *when* you visit the node relative to its children.

```
        1
       / \
      2   3

Pre-order  (Node, Left, Right):  visit node BEFORE children  → 1, 2, 3
In-order   (Left, Node, Right):  node BETWEEN children       → 2, 1, 3
Post-order (Left, Right, Node):  node AFTER children         → 2, 3, 1
```

```python
def preorder(node):
    if not node: return
    visit(node)              # node first
    preorder(node.left)
    preorder(node.right)

def inorder(node):
    if not node: return
    inorder(node.left)
    visit(node)              # node in the middle
    inorder(node.right)

def postorder(node):
    if not node: return
    postorder(node.left)
    postorder(node.right)
    visit(node)              # node last
```

**Key fact:** In-order traversal of a **Binary Search Tree** visits nodes in **sorted order** — a common interview insight.

---

## Problem 1: Maximum Depth of Binary Tree (#104, Easy)

**Statement:** Return the maximum depth (number of nodes along the longest root-to-leaf path).

```
    3
   / \
  9  20          → depth 3
    /  \
   15   7
```

**Approach (DFS):** The depth of a node is 1 + the max depth of its two subtrees. Natural recursion.

```python
def maxDepth(root):
    if not root:
        return 0
    return 1 + max(maxDepth(root.left), maxDepth(root.right))
```

**Trace:**
```
maxDepth(3) = 1 + max(maxDepth(9), maxDepth(20))
maxDepth(9) = 1 + max(0, 0) = 1
maxDepth(20) = 1 + max(maxDepth(15), maxDepth(7)) = 1 + max(1,1) = 2
maxDepth(3) = 1 + max(1, 2) = 3 ✓
```

**Complexity:** Time O(n) — visit every node once. Space O(h) — recursion stack, where h is the tree height (O(log n) balanced, O(n) worst case).

**Why DFS:** Depth is inherently recursive — a node's depth depends on its children's depths. This is the cleanest DFS example.

---

## Problem 2: Binary Tree Level Order Traversal (#102, Medium)

**Statement:** Return the node values level by level, as a list of lists.

```
    3
   / \
  9  20          → [[3], [9,20], [15,7]]
    /  \
   15   7
```

**Approach (BFS):** Process the tree level by level using a queue. For each level, record all current nodes, then enqueue their children.

```python
from collections import deque

def levelOrder(root):
    if not root:
        return []
    result = []
    queue = deque([root])
    while queue:
        level_size = len(queue)        # how many nodes on this level
        level = []
        for _ in range(level_size):
            node = queue.popleft()
            level.append(node.val)
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
        result.append(level)
    return result
```

**The key trick:** Capture `level_size = len(queue)` *before* the inner loop — that's how many nodes belong to the current level. You process exactly that many, and the children you enqueue form the next level.

**Trace:**
```
queue=[3]        → level [3],    enqueue 9,20
queue=[9,20]     → level [9,20], enqueue 15,7
queue=[15,7]     → level [15,7], no children
→ [[3],[9,20],[15,7]] ✓
```

**Complexity:** Time O(n), Space O(n) — the queue holds up to one level (up to n/2 nodes).

**Why BFS:** "Level by level" is the textbook signal for BFS with a queue.

---

## Problem 3: Number of Islands (#200, Medium)

**Statement:** Given a grid of `'1'` (land) and `'0'` (water), count the islands. An island is land connected horizontally/vertically.

```
Input:
11110
11010
11000
00000
Output: 1

Input:
11000
11000
00100
00011
Output: 3
```

**Approach (DFS flood fill):** Scan the grid. When you hit un-visited land, that's a new island — DFS out from it, "sinking" all connected land so you don't count it again.

```python
def numIslands(grid):
    if not grid:
        return 0
    rows, cols = len(grid), len(grid[0])
    count = 0

    def dfs(r, c):
        # out of bounds or water → stop
        if r < 0 or r >= rows or c < 0 or c >= cols or grid[r][c] == '0':
            return
        grid[r][c] = '0'               # sink this land (mark visited)
        dfs(r+1, c)                    # explore 4 directions
        dfs(r-1, c)
        dfs(r, c+1)
        dfs(r, c-1)

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '1':      # new unvisited island
                count += 1
                dfs(r, c)              # sink the whole island
    return count
```

**The idea:** Each DFS call floods one entire island, marking all its cells as water. So the number of times we *start* a DFS equals the number of islands.

**Complexity:** Time O(rows × cols) — each cell visited once. Space O(rows × cols) worst case (recursion depth if the grid is all land).

**Note:** This "sinks" the grid (mutates input). If you can't modify the input, use a separate `visited` set.

---

## Problem 4: Number of Connected Components (#323, Medium — Premium)

**Statement:** Given `n` nodes (0 to n-1) and a list of undirected edges, count the connected components.

```
Input:  n=5, edges=[[0,1],[1,2],[3,4]]
Output: 2       (component {0,1,2} and component {3,4})
```

> **Note:** #323 is a **LeetCode Premium** problem. The free, nearly identical problem is **#547 "Number of Provinces"** — same connected-components idea with an adjacency matrix. Practice #547 to learn this pattern for free.

**Approach (DFS on a graph):** Build an adjacency list. For each unvisited node, DFS marks its whole component. Count how many DFS launches you need.

```python
def countComponents(n, edges):
    # build adjacency list
    adj = {i: [] for i in range(n)}
    for a, b in edges:
        adj[a].append(b)
        adj[b].append(a)              # undirected → both directions

    visited = set()

    def dfs(node):
        visited.add(node)
        for neighbor in adj[node]:
            if neighbor not in visited:
                dfs(neighbor)

    count = 0
    for node in range(n):
        if node not in visited:       # new component
            count += 1
            dfs(node)                 # mark the whole component
    return count
```

**Same shape as Number of Islands:** count how many times you *start* a traversal over unvisited nodes. Each start = one component.

**Complexity:** Time O(V + E) — visit each node and edge once. Space O(V + E) — adjacency list + visited set + recursion.

**Alternative:** Union-Find (Disjoint Set) solves this too, in near-O(E) — worth mentioning as an alternative if asked.

---

## The Pattern Templates

```python
# DFS (recursion) — go deep
def dfs(node, visited):
    if base_case(node) or node in visited:
        return
    visited.add(node)
    for neighbor in neighbors(node):
        dfs(neighbor, visited)

# BFS (queue) — level by level / shortest path
from collections import deque
def bfs(start):
    queue = deque([start])
    visited = {start}
    while queue:
        node = queue.popleft()
        for neighbor in neighbors(node):
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)

# "Count components / islands" — count traversal launches
count = 0
for node in all_nodes:
    if node not in visited:
        count += 1
        dfs(node, visited)      # one launch floods one component
```

---

## Interview Tips

1. **"Shortest path / level by level" → BFS (queue)** — the clearest BFS signal
2. **"Explore fully / depth / path exists" → DFS (recursion)** — the DFS signal
3. **In-order traversal of a BST = sorted order** — a favorite fact
4. **Level-order: capture `len(queue)` before the inner loop** — the level-size trick
5. **Islands/components = count traversal launches** — each start floods one group
6. **Graph traversal is O(V + E)** — know this complexity
7. **Mark visited** — forgetting this causes infinite loops on graphs with cycles

---

## Common Pitfalls

1. **Forgetting `visited`** — graphs have cycles; DFS/BFS loop forever without it
2. **Using a list as a queue** — `list.pop(0)` is O(n); use `collections.deque`
3. **Not handling the empty tree/grid** — check for null root / empty grid
4. **Recursion depth on huge inputs** — DFS can hit Python's recursion limit; BFS or iterative DFS avoids it
5. **Confusing when to use BFS vs DFS** — shortest path needs BFS, not DFS
6. **Mutating input when you shouldn't** — Number of Islands sinks the grid; use a visited set if the input must be preserved
