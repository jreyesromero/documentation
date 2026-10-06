# Pattern: Stack

A stack (LIFO — last in, first out) is the right tool whenever you need to match pairs, track the most recent unresolved item, or process things in reverse of how they arrived. In Python, a plain list works as a stack (`append` to push, `pop` to pop).

## When to Recognize This Pattern

```
"Matching brackets / parentheses..."              → stack
"Valid nesting / balanced..."                     → stack
"Evaluate an expression (postfix/RPN)..."         → stack
"Most recent unmatched element..."                → stack
"Undo / backtrack to the last..."                 → stack
"Next greater/smaller element..."                 → monotonic stack
```

**The core insight:** A stack remembers the *most recent* unresolved thing, which is exactly what you need for nested structures and matching.

---

## Stack Operations (Python)

```python
stack = []
stack.append(x)      # push
top = stack.pop()    # pop (removes and returns the last item)
peek = stack[-1]     # look at the top without removing
is_empty = not stack # empty check
```

---

## Problem 1: Valid Parentheses (#20, Easy)

**Statement:** Given a string of `()[]{}`, determine if the brackets are validly opened and closed (correct type and order).

```
Input:  "()"       → true
Input:  "()[]{}"   → true
Input:  "(]"       → false
Input:  "([)]"     → false    (wrong nesting order)
Input:  "{[]}"     → true
```

**Approach:** Push opening brackets. On a closing bracket, the top of the stack *must* be the matching opener. If not (or the stack is empty), it's invalid. At the end, the stack must be empty.

```python
def isValid(s):
    stack = []
    pairs = {')': '(', ']': '[', '}': '{'}   # closer -> opener
    for char in s:
        if char in pairs:                     # it's a closing bracket
            # top must be the matching opener
            if not stack or stack.pop() != pairs[char]:
                return False
        else:                                 # it's an opening bracket
            stack.append(char)
    return len(stack) == 0                     # all matched?
```

**Trace** `"([)]"`:
```
'(' → push, stack=['(']
'[' → push, stack=['(','[']
')' → closer; top is '[', need '(' → mismatch → False ✓
```

**Trace** `"{[]}"`:
```
'{' → push ['{']
'[' → push ['{','[']
']' → top '[' matches → pop ['{']
'}' → top '{' matches → pop []
end: stack empty → True ✓
```

**Complexity:** Time O(n), Space O(n) — worst case all openers.

**Why a stack?** Nesting means the *most recent* opener must close first — that's LIFO, exactly what a stack gives you.

---

## Problem 2: Min Stack (#155, Medium)

**Statement:** Design a stack supporting `push`, `pop`, `top`, and `getMin` — all in **O(1)**.

```
push(-2); push(0); push(-3)
getMin()  → -3
pop()
top()     → 0
getMin()  → -2
```

**Approach:** The challenge is O(1) `getMin`. Keep a *second* stack that tracks the minimum at each level. When you push, also push the current min onto the min-stack.

```python
class MinStack:
    def __init__(self):
        self.stack = []
        self.min_stack = []        # min_stack[i] = min of stack[0..i]

    def push(self, val):
        self.stack.append(val)
        # the new min is either val or the previous min
        current_min = val if not self.min_stack else min(val, self.min_stack[-1])
        self.min_stack.append(current_min)

    def pop(self):
        self.stack.pop()
        self.min_stack.pop()       # keep them in sync

    def top(self):
        return self.stack[-1]

    def getMin(self):
        return self.min_stack[-1]  # O(1) — the min is always on top
```

**The trick:** `min_stack[-1]` always holds the minimum of everything currently in the stack. Because they push/pop together, the min for each state is preserved.

**Trace:**
```
push(-2): stack=[-2],      min_stack=[-2]
push(0):  stack=[-2,0],    min_stack=[-2,-2]   (min(0,-2)=-2)
push(-3): stack=[-2,0,-3], min_stack=[-2,-2,-3]
getMin() → -3
pop():    stack=[-2,0],    min_stack=[-2,-2]
getMin() → -2 ✓
```

**Complexity:** All operations O(1) time. Space O(n) for the extra min-stack.

---

## Problem 3: Evaluate Reverse Polish Notation (#150, Medium)

**Statement:** Evaluate an arithmetic expression in Reverse Polish (postfix) Notation. Operators follow their operands.

```
Input:  ["2","1","+","3","*"]           → 9      ((2+1)*3)
Input:  ["4","13","5","/","+"]          → 6      (4 + (13/5)) = 4 + 2
```

**Approach:** Push numbers. On an operator, pop the top two numbers, apply the operator, push the result. At the end, the single remaining value is the answer.

```python
def evalRPN(tokens):
    stack = []
    operators = {'+', '-', '*', '/'}
    for token in tokens:
        if token in operators:
            b = stack.pop()            # second operand (popped first!)
            a = stack.pop()            # first operand
            if token == '+':
                stack.append(a + b)
            elif token == '-':
                stack.append(a - b)
            elif token == '*':
                stack.append(a * b)
            else:  # division truncates toward zero
                stack.append(int(a / b))
        else:
            stack.append(int(token))
    return stack[0]
```

**Trace** `["2","1","+","3","*"]`:
```
'2' → push [2]
'1' → push [2,1]
'+' → pop 1, pop 2 → 2+1=3 → push [3]
'3' → push [3,3]
'*' → pop 3, pop 3 → 3*3=9 → push [9]
→ 9 ✓
```

**Order matters:** The first pop is the *right* operand (`b`), the second is the *left* (`a`). This matters for `-` and `/`: it's `a - b`, not `b - a`.

**Complexity:** Time O(n), Space O(n).

---

## The Pattern Template

```python
# Matching / validation
stack = []
for item in sequence:
    if is_opener(item):
        stack.append(item)
    elif is_closer(item):
        if not stack or not matches(stack.pop(), item):
            return False
return len(stack) == 0

# Expression evaluation
stack = []
for token in tokens:
    if is_operand(token):
        stack.append(value(token))
    else:  # operator
        b = stack.pop()
        a = stack.pop()
        stack.append(apply(a, b, token))
return stack[0]

# Auxiliary stack (track min/max alongside)
main, aux = [], []
def push(x):
    main.append(x)
    aux.append(min(x, aux[-1]) if aux else x)
```

---

## Interview Tips

1. **Brackets / nesting / matching → stack** — the most common trigger
2. **LIFO = most recent unresolved item** — the mental model
3. **Validation ends with an empty-stack check** — leftover openers mean invalid
4. **Auxiliary stack for O(1) min/max** — the Min Stack trick
5. **RPN: pop order is b then a** — matters for subtraction and division
6. **Python list = stack** — `append` / `pop` / `[-1]`

---

## Common Pitfalls

1. **Forgetting the final empty check** — `"((("` has matched nothing but isn't valid
2. **Popping an empty stack** — always check `if not stack` before popping on a closer
3. **Reversed operand order in RPN** — first pop is the right operand
4. **Recomputing min by scanning** — that's O(n); use the auxiliary min-stack for O(1)
5. **Integer division direction** — `int(a/b)` truncates toward zero; `//` floors (differs for negatives)
