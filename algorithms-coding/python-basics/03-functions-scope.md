# 03: Functions & Scope

Functions organize code into reusable blocks. Understanding scope is critical for debugging.

## Defining Functions

```python
# Basic function
def greet(name):
    return f"Hello, {name}!"

result = greet("Alice")  # "Hello, Alice!"

# Multiple parameters
def add(x, y):
    return x + y

# Default parameters
def power(base, exponent=2):
    return base ** exponent

power(3)       # 9 (uses default exponent=2)
power(3, 3)    # 27

# No return statement (returns None)
def print_info(name):
    print(f"Name: {name}")

# Return multiple values
def get_min_max(numbers):
    return min(numbers), max(numbers)

mn, mx = get_min_max([1, 5, 3])  # mn=1, mx=5
```

## Variable Arguments: *args and **kwargs

```python
# *args: variable number of positional arguments (tuple)
def sum_all(*args):
    return sum(args)

sum_all(1, 2, 3, 4)  # 10
sum_all()             # 0

# **kwargs: variable number of keyword arguments (dict)
def print_config(**kwargs):
    for key, value in kwargs.items():
        print(f"{key} = {value}")

print_config(host="localhost", port=8080, debug=True)
# host = localhost
# port = 8080
# debug = True

# Combining all
def flexible(*args, **kwargs):
    print("Positional:", args)
    print("Keyword:", kwargs)

flexible(1, 2, 3, name="Bob", age=30)
# Positional: (1, 2, 3)
# Keyword: {'name': 'Bob', 'age': 30}

# Unpacking arguments
numbers = [1, 2, 3]
print(sum(numbers))     # Error! sum() expects integers, not a list
print(sum(*numbers))    # sum(1, 2, 3) — unpacks the list

config = {"host": "localhost", "port": 8080}
print_config(**config)  # Unpacks the dict as keyword arguments
```

## Scope: LEGB Rule

Python resolves variable names in this order: **Local → Enclosing → Global → Built-in**

```python
x = "global"  # Global scope

def outer():
    x = "enclosing"  # Enclosing scope
    
    def inner():
        x = "local"  # Local scope
        print(x)  # "local"
    
    inner()
    print(x)  # "enclosing"

print(x)  # "global"

# Accessing global variables
y = 5

def modify_global():
    global y         # Tell Python to use the global y
    y = 10

modify_global()
print(y)  # 10
```

## Common Patterns

### Early Return
```python
def validate_user(user):
    if not user:
        return None
    if "name" not in user:
        return None
    if "email" not in user:
        return None
    return user

# Instead of deeply nested if statements
```

### Default to Mutable (Gotcha Again!)
```python
# WRONG!
def append_item(item, items=[]):
    items.append(item)
    return items

# RIGHT!
def append_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

### Helper Functions
```python
def is_even(n):
    return n % 2 == 0

def filter_evens(numbers):
    result = []
    for num in numbers:
        if is_even(num):  # Reuse helper
            result.append(num)
    return result

# Or with a list comprehension
def filter_evens_v2(numbers):
    return [n for n in numbers if is_even(n)]
```

## Lambda (Anonymous Functions)

Lambdas are small, one-line functions. Use them sparingly.

```python
# Regular function
def double(x):
    return x * 2

# Lambda
double = lambda x: x * 2

# Using with built-in functions
numbers = [1, 2, 3, 4, 5]
doubled = map(lambda x: x * 2, numbers)  # [2, 4, 6, 8, 10]
evens = filter(lambda x: x % 2 == 0, numbers)  # [2, 4]

# Sorting with lambda
people = [
    {"name": "Alice", "age": 30},
    {"name": "Bob", "age": 25},
]
sorted_by_age = sorted(people, key=lambda p: p["age"])
# Bob (25) comes first, then Alice (30)

# Better: use a named function instead for readability
def get_age(person):
    return person["age"]

sorted_by_age = sorted(people, key=get_age)
```

## Recursion

A function calling itself. Base case is essential to avoid infinite recursion.

```python
# Factorial: 5! = 5 * 4 * 3 * 2 * 1 = 120
def factorial(n):
    if n <= 1:           # Base case
        return 1
    return n * factorial(n - 1)  # Recursive case

factorial(5)  # 120

# Fibonacci
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

fibonacci(6)  # 8

# Count down
def countdown(n):
    if n <= 0:
        print("Done!")
        return
    print(n)
    countdown(n - 1)

countdown(3)
# 3
# 2
# 1
# Done!
```

## Docstrings

Document what your function does:

```python
def calculate_average(numbers):
    """
    Calculate the average of a list of numbers.
    
    Args:
        numbers: A list of integers or floats
        
    Returns:
        The average as a float. Returns 0 for empty list.
        
    Raises:
        TypeError: If numbers is not a list
    """
    if not numbers:
        return 0
    return sum(numbers) / len(numbers)

# Access docstring
print(calculate_average.__doc__)
```

## Exercises

### Exercise 1: Recursive Sum
Write a function that sums a list of numbers recursively (without using `sum()`).

```python
def recursive_sum(numbers):
    pass

# Test cases:
assert recursive_sum([1, 2, 3, 4, 5]) == 15
assert recursive_sum([]) == 0
assert recursive_sum([10]) == 10
```

**Solution:**
```python
def recursive_sum(numbers):
    if not numbers:
        return 0
    return numbers[0] + recursive_sum(numbers[1:])
```

### Exercise 2: Filter with Lambda
Write a function that takes a list and a condition function, returns filtered results.

```python
def filter_list(items, condition):
    pass

# Test cases:
evens = filter_list([1, 2, 3, 4, 5], lambda x: x % 2 == 0)
assert evens == [2, 4]

long_words = filter_list(["hi", "hello", "hey"], lambda w: len(w) > 3)
assert long_words == ["hello"]
```

**Solution:**
```python
def filter_list(items, condition):
    return [item for item in items if condition(item)]

# Or using filter():
def filter_list_v2(items, condition):
    return list(filter(condition, items))
```

### Exercise 3: Compose Functions
Write a function that returns a new function combining two operations.

```python
def compose(f, g):
    """Returns a function that applies f, then g"""
    pass

# Test case:
double = lambda x: x * 2
add_one = lambda x: x + 1
h = compose(double, add_one)  # First double, then add_one
assert h(5) == 11  # (5 * 2) + 1 = 11
```

**Solution:**
```python
def compose(f, g):
    def composed(x):
        return g(f(x))
    return composed

# Or more concise with lambda:
def compose_v2(f, g):
    return lambda x: g(f(x))
```

## Key Takeaways

✅ Functions make code reusable and testable  
✅ Use early returns to avoid deeply nested if statements  
✅ **Mutable defaults** — use `None` and create inside function  
✅ `*args` is a tuple, `**kwargs` is a dict  
✅ LEGB: Local → Enclosing → Global → Built-in scope order  
✅ `global` keyword lets you modify global variables  
✅ Recursion needs a base case to avoid infinite loops  
✅ Lambdas are fine for simple functions passed to `map()`/`filter()`/`sorted()`  

## Next Module

[File Handling](./04-file-handling.md) — Reading/writing files, JSON, YAML, and context managers
