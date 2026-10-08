# 01: Python Fundamentals

Core building blocks: variables, types, operators, and control flow.

## Variables and Data Types

Python is **dynamically typed** — you don't declare types, Python infers them.

```python
# Integers and floats
age = 25              # int
height = 5.9          # float
count = int("42")     # String to int

# Strings
name = "Alice"
message = 'Hello'
multiline = """This
spans multiple
lines"""

# Booleans
is_active = True
is_deleted = False

# None (absence of value)
result = None

# Type checking
print(type(age))      # <class 'int'>
print(isinstance(age, int))  # True
```

## Type Conversion

```python
# Convert between types
x = "123"
y = int(x)            # 123 (int)
z = float(x)          # 123.0 (float)
s = str(42)           # "42" (str)
b = bool(0)           # False
b = bool(1)           # True
b = bool("")          # False (empty string)
b = bool("hello")     # True

# Common gotcha: type conversion can fail
try:
    result = int("abc")  # ValueError!
except ValueError as e:
    print(f"Can't convert: {e}")
```

## Operators

### Arithmetic
```python
10 + 3   # 13
10 - 3   # 7
10 * 3   # 30
10 / 3   # 3.333... (float division)
10 // 3  # 3 (integer division, rounds down)
10 % 3   # 1 (modulo, remainder)
2 ** 3   # 8 (exponentiation)
```

### Comparison (return True/False)
```python
5 == 5        # True (equal)
5 != 3        # True (not equal)
5 > 3         # True (greater than)
5 >= 5        # True (greater or equal)
5 < 10        # True (less than)
5 <= 5        # True (less or equal)

# String comparison (lexicographic/alphabetical)
"apple" < "banana"  # True
"zebra" > "apple"   # True
```

### Logical
```python
True and False   # False (both must be True)
True or False    # True (at least one True)
not True         # False (negation)

# Short-circuit evaluation (important!)
5 > 3 and 10 / 0  # Doesn't crash! The "and" short-circuits
False or 10 / 0   # This DOES crash (second part is evaluated)
```

### Membership
```python
5 in [1, 2, 5, 10]      # True
"x" in "hello"          # False
"e" in "hello"          # True
"hello" in "hello world"  # True (substring)
```

## Control Flow

### if / elif / else
```python
age = 25

if age < 13:
    print("Child")
elif age < 18:
    print("Teenager")
elif age < 65:
    print("Adult")
else:
    print("Senior")

# Ternary operator (one-liner)
status = "Adult" if age >= 18 else "Minor"
```

### for loops
```python
# Iterate over a range
for i in range(5):        # 0, 1, 2, 3, 4
    print(i)

for i in range(2, 10, 2): # Start=2, stop=10 (exclusive), step=2
    print(i)              # 2, 4, 6, 8

# Iterate over a list
fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    print(fruit)

# Get index and value
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")  # 0: apple, 1: banana, etc.

# Iterate over dictionary
person = {"name": "Alice", "age": 25, "city": "NYC"}
for key, value in person.items():
    print(f"{key} = {value}")
```

### while loops
```python
count = 0
while count < 5:
    print(count)
    count += 1  # count = count + 1

# break and continue
while True:
    user_input = input("Enter 'quit' to exit: ")
    if user_input == "quit":
        break          # Exit the loop
    if user_input == "skip":
        continue       # Skip to next iteration
    print(f"You entered: {user_input}")
```

## String Basics

```python
# String methods
s = "hello world"
s.upper()         # "HELLO WORLD"
s.capitalize()    # "Hello world"
s.replace("world", "python")  # "hello python"
s.split(" ")      # ["hello", "world"]
s.strip()         # Remove leading/trailing whitespace
s.startswith("he")  # True
s.endswith("ld")    # True
s.find("world")     # 6 (index of first occurrence)
s.count("l")        # 3 (occurrences)

# String concatenation
first = "John"
last = "Doe"
full_name = first + " " + last      # "John Doe"
full_name = f"{first} {last}"       # f-string (cleaner!)
full_name = "{} {}".format(first, last)  # .format()

# f-strings (Python 3.6+, preferred)
name = "Bob"
age = 30
print(f"{name} is {age} years old")
print(f"In 5 years, {name} will be {age + 5}")
print(f"Hello {'WORLD' if True else 'world'}")
```

## Input and Output

```python
# Print
print("Hello")
print("A", "B", "C", sep=", ", end="!\n")  # "A, B, C!"

# Input
name = input("What's your name? ")
print(f"Hello, {name}!")
```

## Exercises

### Exercise 1: FizzBuzz
Write a program that prints numbers 1 to 100, but:
- Print "Fizz" if divisible by 3
- Print "Buzz" if divisible by 5
- Print "FizzBuzz" if divisible by both
- Otherwise print the number

```python
# Hint: use modulo (%) to check divisibility
```

**Solution:**
```python
for i in range(1, 101):
    output = ""
    if i % 3 == 0:
        output += "Fizz"
    if i % 5 == 0:
        output += "Buzz"
    if output:
        print(output)
    else:
        print(i)
```

### Exercise 2: Palindrome Check
Write a function that checks if a string is a palindrome (reads the same forwards and backwards). Ignore spaces and case.

```python
# Hint: use string methods and slicing
def is_palindrome(s):
    pass

# Test cases:
assert is_palindrome("racecar") == True
assert is_palindrome("A man a plan a canal Panama") == True
assert is_palindrome("hello") == False
```

**Solution:**
```python
def is_palindrome(s):
    # Remove spaces and convert to lowercase
    cleaned = s.replace(" ", "").lower()
    # Check if it equals its reverse
    return cleaned == cleaned[::-1]
```

### Exercise 3: Temperature Converter
Write functions to convert between Celsius, Fahrenheit, and Kelvin.

```python
def celsius_to_fahrenheit(c):
    pass

def fahrenheit_to_celsius(f):
    pass

def celsius_to_kelvin(c):
    pass

# Test cases:
assert celsius_to_fahrenheit(0) == 32
assert celsius_to_fahrenheit(100) == 212
assert fahrenheit_to_celsius(32) == 0
assert celsius_to_kelvin(0) == 273.15
```

**Solution:**
```python
def celsius_to_fahrenheit(c):
    return (c * 9/5) + 32

def fahrenheit_to_celsius(f):
    return (f - 32) * 5/9

def celsius_to_kelvin(c):
    return c + 273.15
```

## Key Takeaways

✅ Python is dynamically typed — no explicit type declarations  
✅ Use `==` for equality, `is` for object identity (mostly don't need `is`)  
✅ String slicing: `s[start:stop:step]` where stop is **exclusive**  
✅ `range(n)` goes from 0 to n-1  
✅ f-strings are cleaner than `.format()` or `+` concatenation  
✅ `enumerate()` gives you both index and value  
✅ Modulo `%` is great for divisibility checks  

## Next Module

[Data Structures](./02-data-structures.md) — Lists, dictionaries, sets, and tuples
