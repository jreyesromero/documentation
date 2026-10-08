# 06: Strings & Regular Expressions

String manipulation is fundamental. Regex helps with pattern matching.

## String Methods

```python
s = "  Hello, World!  "

# Case conversion
s.upper()           # "  HELLO, WORLD!  "
s.lower()           # "  hello, world!  "
s.capitalize()      # "  hello, world!  " (first char upper)
s.title()           # "  Hello, World!  "

# Whitespace
s.strip()           # "Hello, World!" (remove leading/trailing)
s.lstrip()          # "Hello, World!  " (left only)
s.rstrip()          # "  Hello, World!" (right only)

# Searching
s.find("World")     # 10 (index of first occurrence)
s.rfind("o")        # 13 (rightmost occurrence)
s.count("l")        # 3 (count of substring)
s.startswith("  H") # True
s.endswith("!")     # True

# Replacement
s.replace("World", "Python")  # "  Hello, Python!  "
s.replace("o", "0", 1)        # "  Hell0, World!  " (only first)

# Splitting and joining
sentence = "apple,banana,cherry"
words = sentence.split(",")   # ["apple", "banana", "cherry"]
",".join(words)               # "apple,banana,cherry"

# Is it...?
"123".isdigit()     # True
"abc".isalpha()     # True
"abc123".isalnum()  # True
```

## F-Strings (Preferred in Python 3.6+)

F-strings are the cleanest way to format strings.

```python
name = "Alice"
age = 25

# F-string
print(f"{name} is {age} years old")

# Expressions in f-strings
x = 10
y = 3
print(f"{x} + {y} = {x + y}")

# Formatting numbers
pi = 3.14159
print(f"Pi is {pi:.2f}")        # 3.14
print(f"Percentage: {0.85:.1%}") # 85.0%

# Formatting with padding
print(f"Name: {name:>10}")      # Right-aligned, 10 chars
print(f"Name: {name:<10}")      # Left-aligned
print(f"Name: {name:^10}")      # Center-aligned

# Debug with = (Python 3.8+)
print(f"{name=}")               # "name='Alice'" (variable name and value)
```

## Regular Expressions (Regex)

Regex is powerful for pattern matching. Use `re` module.

```python
import re

# Test if pattern matches
if re.search(r"\d+", "I have 123 apples"):
    print("Found numbers!")

# Find all matches
matches = re.findall(r"\d+", "I have 123 apples and 456 oranges")
print(matches)  # ['123', '456']

# Replace using regex
text = "The phone is 555-1234"
new_text = re.sub(r"\d", "X", text)  # "The phone is XXX-XXXX"

# Match object (more details)
match = re.search(r"(\d+)-(\d+)", "Call 555-1234")
if match:
    print(match.group(0))  # "555-1234" (full match)
    print(match.group(1))  # "555" (first group)
    print(match.group(2))  # "1234" (second group)
```

### Common Regex Patterns

```python
import re

# Email validation (simplified)
email = "user@example.com"
if re.match(r"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$", email):
    print("Valid email")

# Phone number (US format)
phone = "555-1234"
if re.match(r"\d{3}-\d{4}", phone):
    print("Valid phone")

# IP address
ip = "192.168.1.1"
if re.match(r"^(\d{1,3}\.){3}\d{1,3}$", ip):
    print("Valid IP format")

# Extract numbers
text = "The cost is $19.99 and tax is $2.50"
prices = re.findall(r"\$\d+\.\d+", text)
print(prices)  # ['$19.99', '$2.50']

# Extract words
text = "hello-world from_python"
words = re.findall(r"\w+", text)
print(words)  # ['hello', 'world', 'from', 'python']
```

### Regex Metacharacters

| Character | Meaning |
|-----------|---------|
| `.` | Any single character (except newline) |
| `\d` | Digit (0-9) |
| `\w` | Word character (a-z, A-Z, 0-9, _) |
| `\s` | Whitespace (space, tab, newline) |
| `^` | Start of string |
| `$` | End of string |
| `*` | Zero or more |
| `+` | One or more |
| `?` | Zero or one |
| `{n}` | Exactly n times |
| `{n,m}` | Between n and m times |
| `[]` | Character class (any of these) |
| `[^]` | Negated class (not these) |
| `()` | Capture group |
| `\|` | OR (alternation) |

### Useful Regex Examples

```python
import re

# Split on multiple delimiters
text = "apple,banana;orange:grape"
fruits = re.split(r"[,;:]", text)
print(fruits)  # ['apple', 'banana', 'orange', 'grape']

# Case-insensitive matching
pattern = "HELLO"
text = "hello world"
if re.search(pattern, text, re.IGNORECASE):
    print("Found!")

# Multiline matching
text = "line1\nline2\nline3"
lines = re.findall(r"^line\d", text, re.MULTILINE)
print(lines)  # ['line1', 'line2', 'line3']

# Group extraction
log_line = "ERROR: Database connection failed at 2026-01-10 10:00:05"
match = re.search(r"(\w+): (.+) at (\S+)", log_line)
if match:
    level = match.group(1)      # ERROR
    message = match.group(2)    # Database connection failed
    timestamp = match.group(3)  # 2026-01-10
```

## Exercises

### Exercise 1: Email Validation
Write a function that validates email addresses.

```python
import re

def is_valid_email(email):
    """Check if email is valid"""
    pass

# Test cases:
assert is_valid_email("user@example.com") == True
assert is_valid_email("invalid.email") == False
assert is_valid_email("user@domain.co.uk") == True
assert is_valid_email("") == False
```

**Solution:**
```python
import re

def is_valid_email(email):
    pattern = r"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"
    return bool(re.match(pattern, email))
```

### Exercise 2: Extract Info from Log Line
Write a function that parses a log line and extracts timestamp, level, and message.

```python
import re

def parse_log_line(line):
    """Extract timestamp, level, and message from log"""
    pass

# Test case:
line = "2026-01-10 10:00:05 ERROR Database connection failed"
result = parse_log_line(line)
assert result == {
    "timestamp": "2026-01-10 10:00:05",
    "level": "ERROR",
    "message": "Database connection failed"
}
```

**Solution:**
```python
import re

def parse_log_line(line):
    pattern = r"^(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}) (\w+) (.+)$"
    match = re.match(pattern, line)
    if match:
        return {
            "timestamp": match.group(1),
            "level": match.group(2),
            "message": match.group(3),
        }
    return None
```

### Exercise 3: Text Cleanup
Write a function that cleans text: removes extra spaces, converts to lowercase, removes punctuation.

```python
import re

def clean_text(text):
    """Clean text for processing"""
    pass

# Test cases:
assert clean_text("  Hello,  World!  ") == "hello world"
assert clean_text("PYTHON 3.9!!!") == "python 39"
```

**Solution:**
```python
import re

def clean_text(text):
    # Remove punctuation
    text = re.sub(r"[^\w\s]", "", text)
    # Convert to lowercase
    text = text.lower()
    # Remove extra whitespace
    text = re.sub(r"\s+", " ", text).strip()
    return text
```

## Key Takeaways

✅ F-strings are the preferred way to format strings  
✅ `.strip()`, `.split()`, `.replace()` are essential string methods  
✅ Regex (regex) is powerful for pattern matching  
✅ Use raw strings: `r"pattern"` to avoid escaping issues  
✅ Common patterns: email, phone, IP, dates, log lines  
✅ `re.search()` finds first match, `re.findall()` finds all  
✅ `re.sub()` for replacement, `re.split()` for splitting  
✅ Capture groups with `()` and `match.group(n)`  
✅ Use `re.IGNORECASE` and `re.MULTILINE` flags  

## Next Module

[Common Algorithms](./07-common-algorithms.md) — Searching, sorting, and the two-pointer technique
