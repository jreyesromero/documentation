# 05: Error Handling

Writing robust code that handles errors gracefully is critical for SRE roles.

## Try / Except / Finally

```python
# Basic try/except
try:
    x = int("not a number")  # ValueError
except ValueError:
    print("Could not convert to integer")

# Catch specific exceptions
try:
    items = [1, 2, 3]
    print(items[10])  # IndexError
except IndexError:
    print("Index out of range")
except ValueError:
    print("Invalid value")

# Catch multiple exceptions with one handler
try:
    result = 10 / 0  # ZeroDivisionError
except (ValueError, ZeroDivisionError):
    print("Either a value or division error")

# Catch all exceptions (generally avoid, but good for debugging)
try:
    x = 10 / 0
except Exception as e:
    print(f"An error occurred: {e}")
    print(f"Error type: {type(e).__name__}")

# Access error details
try:
    x = int("invalid")
except ValueError as e:
    print(f"Error message: {e}")

# Finally block (always runs, even if error)
try:
    f = open("data.txt", "r")
    content = f.read()
except FileNotFoundError:
    print("File not found")
finally:
    f.close()  # But use 'with' instead!

# Better: use with statement (no finally needed)
try:
    with open("data.txt", "r") as f:
        content = f.read()
except FileNotFoundError:
    print("File not found")
```

## Common Exceptions

```python
# IndexError: accessing invalid index
items = [1, 2, 3]
# x = items[10]  # IndexError

# KeyError: accessing invalid dict key
config = {"host": "localhost"}
# port = config["port"]  # KeyError
port = config.get("port", 8080)  # Use .get()

# ValueError: invalid value for operation
# x = int("not a number")  # ValueError

# TypeError: wrong type
# result = "hello" + 5  # TypeError

# ZeroDivisionError: division by zero
# x = 10 / 0  # ZeroDivisionError

# FileNotFoundError: file doesn't exist
# f = open("nonexistent.txt")  # FileNotFoundError

# AttributeError: accessing invalid attribute
s = "hello"
# s.invalid_method()  # AttributeError

# Convert string to int with error handling
def safe_int(value, default=0):
    try:
        return int(value)
    except ValueError:
        return default

safe_int("42")      # 42
safe_int("invalid") # 0
```

## Custom Exceptions

Define your own exceptions for application-specific errors.

```python
# Define custom exception
class InsufficientFundsError(Exception):
    pass

class InvalidAccountError(Exception):
    pass

# Use custom exception
def withdraw(account, amount):
    if account is None:
        raise InvalidAccountError("Account does not exist")
    
    if amount > account["balance"]:
        raise InsufficientFundsError(f"Cannot withdraw {amount}, balance is {account['balance']}")
    
    account["balance"] -= amount
    return account["balance"]

# Handle custom exception
account = {"name": "Alice", "balance": 100}
try:
    withdraw(account, 150)
except InsufficientFundsError as e:
    print(f"Withdrawal failed: {e}")

# Exception with custom message
class DatabaseError(Exception):
    def __init__(self, message, error_code):
        super().__init__(message)
        self.error_code = error_code

try:
    raise DatabaseError("Connection failed", 503)
except DatabaseError as e:
    print(f"{e} (code: {e.error_code})")
```

## Logging (SRE Essential)

Use logging instead of `print()` for production code.

```python
import logging

# Configure logging
logging.basicConfig(
    level=logging.DEBUG,  # Show all messages >= DEBUG
    format="%(asctime)s - %(levelname)s - %(message)s"
)

logger = logging.getLogger(__name__)

# Log at different levels
logger.debug("Debug message")    # Detailed, for debugging
logger.info("Info message")      # General informational
logger.warning("Warning message")  # Something unexpected
logger.error("Error message")    # A serious problem
logger.critical("Critical message")  # System may fail

# Logging exceptions
try:
    x = 10 / 0
except ZeroDivisionError:
    logger.exception("An error occurred")  # Includes traceback
    # Better than: logger.error(str(e))
```

### Logging Levels

| Level | When to Use |
|-------|------------|
| DEBUG | Detailed info for developers (variable values, loop iterations) |
| INFO | General informational (app started, request processed) |
| WARNING | Something unexpected but app continues (deprecated feature used) |
| ERROR | A serious problem, but app continues (failed request) |
| CRITICAL | System may fail if not addressed (out of memory) |

### Better Logging Setup

```python
import logging
from logging.handlers import RotatingFileHandler

# More realistic setup
logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

# Console handler
console_handler = logging.StreamHandler()
console_handler.setLevel(logging.INFO)
console_format = logging.Formatter("%(levelname)s - %(message)s")
console_handler.setFormatter(console_format)

# File handler with rotation
file_handler = RotatingFileHandler(
    "app.log",
    maxBytes=1024*1024,  # 1MB
    backupCount=3        # Keep 3 old files
)
file_handler.setLevel(logging.DEBUG)
file_format = logging.Formatter("%(asctime)s - %(name)s - %(levelname)s - %(message)s")
file_handler.setFormatter(file_format)

# Add handlers to logger
logger.addHandler(console_handler)
logger.addHandler(file_handler)

# Now use the logger
logger.info("Application started")
logger.debug("Debug information")
logger.error("An error occurred", exc_info=True)
```

## Defensive Programming

Check inputs before processing.

```python
def calculate_average(numbers):
    # Input validation
    if not isinstance(numbers, list):
        raise TypeError("Expected a list")
    
    if not numbers:
        raise ValueError("List cannot be empty")
    
    # Verify all items are numbers
    for item in numbers:
        if not isinstance(item, (int, float)):
            raise TypeError(f"Expected number, got {type(item).__name__}")
    
    return sum(numbers) / len(numbers)

# Usage
try:
    avg = calculate_average([1, 2, 3])
    print(avg)
except (TypeError, ValueError) as e:
    print(f"Invalid input: {e}")

# Guard clauses (early return pattern)
def process_user(user):
    if not user:
        return None
    
    if "email" not in user:
        raise ValueError("User missing email")
    
    if not user.get("active"):
        return None
    
    # Process valid, active user
    return user["email"]
```

## Retries with Exponential Backoff

Common SRE pattern for handling transient failures.

```python
import time

def retry_with_backoff(func, max_retries=3, base_delay=1):
    for attempt in range(max_retries):
        try:
            return func()
        except Exception as e:
            if attempt == max_retries - 1:
                raise  # Last attempt, raise the exception
            
            delay = base_delay * (2 ** attempt)  # 1, 2, 4, ...
            logger.warning(f"Attempt {attempt + 1} failed, retrying in {delay}s: {e}")
            time.sleep(delay)

# Usage
def unstable_api_call():
    import random
    if random.random() < 0.7:
        raise ConnectionError("Network error")
    return "Success"

try:
    result = retry_with_backoff(unstable_api_call)
    print(result)
except Exception as e:
    print(f"Failed after retries: {e}")
```

## Exercises

### Exercise 1: Safe Config Reader
Write a function that safely reads a config value with multiple fallbacks.

```python
def get_config(config_dict, *keys, default=None):
    """Navigate nested dict safely, return default if not found"""
    pass

# Test cases:
config = {"database": {"host": "localhost", "port": 5432}}
assert get_config(config, "database", "host") == "localhost"
assert get_config(config, "database", "user") == None
assert get_config(config, "database", "user", default="root") == "root"
assert get_config(config, "invalid", "path") == None
```

**Solution:**
```python
def get_config(config_dict, *keys, default=None):
    current = config_dict
    for key in keys:
        if isinstance(current, dict):
            current = current.get(key)
            if current is None:
                return default
        else:
            return default
    return current
```

### Exercise 2: Custom Exception
Write a function that validates input and raises custom exceptions.

```python
class ValidationError(Exception):
    pass

def validate_email(email):
    """Validate email format, raise ValidationError if invalid"""
    pass

# Test cases:
validate_email("user@example.com")  # Valid
# validate_email("invalid-email")  # Raises ValidationError
# validate_email("")  # Raises ValidationError
```

**Solution:**
```python
class ValidationError(Exception):
    pass

def validate_email(email):
    if not email or "@" not in email:
        raise ValidationError(f"Invalid email: {email}")
    if not email.split("@")[1]:
        raise ValidationError(f"Invalid domain in email: {email}")
    return True
```

### Exercise 3: Retry Logic
Write a function that retries another function up to N times.

```python
def retry(func, max_attempts=3):
    """Call func up to max_attempts times until success"""
    pass

# Test case:
attempt_count = 0
def flaky_function():
    global attempt_count
    attempt_count += 1
    if attempt_count < 3:
        raise ConnectionError("Try again")
    return "Success"

result = retry(flaky_function, max_attempts=5)
assert result == "Success"
```

**Solution:**
```python
def retry(func, max_attempts=3):
    for attempt in range(max_attempts):
        try:
            return func()
        except Exception as e:
            if attempt == max_attempts - 1:
                raise
            print(f"Attempt {attempt + 1} failed, retrying...")
```

## Key Takeaways

✅ Use try/except for expected errors  
✅ Catch specific exceptions, not bare `except:`  
✅ Use `logger` instead of `print()` in production code  
✅ Custom exceptions make code clearer  
✅ Validate inputs at system boundaries  
✅ Use early returns (guard clauses) to avoid nesting  
✅ Retry with exponential backoff for transient failures  
✅ `.get()` on dicts to avoid KeyError  
✅ `with` statement for file handling (no finally needed)  

## Next Module

[Strings & Regex](./06-strings-regex.md) — String methods, f-strings, and regular expressions
