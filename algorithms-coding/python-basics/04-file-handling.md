# 04: File Handling

Working with files is essential for SRE roles. This module covers reading/writing files, JSON, YAML, and context managers.

## Basic File I/O

### Reading Files

```python
# Simple read
with open("data.txt", "r") as f:
    content = f.read()  # Entire file as string
    print(content)

# Read line by line
with open("data.txt", "r") as f:
    for line in f:
        print(line.strip())  # strip() removes newline

# Read all lines into a list
with open("data.txt", "r") as f:
    lines = f.readlines()  # ["line1\n", "line2\n", ...]
    for line in lines:
        print(line.strip())

# Read a specific number of characters
with open("data.txt", "r") as f:
    first_100 = f.read(100)
```

### Writing Files

```python
# Write (overwrites existing)
with open("output.txt", "w") as f:
    f.write("Hello, World!\n")
    f.write("Second line\n")

# Append (adds to end)
with open("output.txt", "a") as f:
    f.write("Third line\n")

# Write multiple lines
lines = ["Line 1\n", "Line 2\n", "Line 3\n"]
with open("output.txt", "w") as f:
    f.writelines(lines)
```

### File Modes

| Mode | Description |
|------|-------------|
| `r` | Read (default, file must exist) |
| `w` | Write (creates or overwrites) |
| `a` | Append (adds to end) |
| `r+` | Read and write |
| `w+` | Write and read |

## Context Managers: `with` Statement

**Always use `with` when working with files** — it automatically closes the file.

```python
# WITHOUT with (bad) — file might not close
f = open("data.txt", "r")
content = f.read()
# What if an error happens? File stays open!

# WITH with (good) — file closes automatically
with open("data.txt", "r") as f:
    content = f.read()
# File is closed even if an error occurs
```

## Working with JSON

JSON is a standard format for data exchange. Python's `json` module handles it.

```python
import json

# Python dict → JSON string
data = {"name": "Alice", "age": 25, "city": "NYC"}
json_string = json.dumps(data)
print(json_string)  # {"name": "Alice", "age": 25, "city": "NYC"}

# JSON string → Python dict
json_string = '{"name": "Bob", "age": 30}'
data = json.loads(json_string)
print(data["name"])  # Bob

# Read JSON from file
with open("config.json", "r") as f:
    config = json.load(f)  # load, not loads
    print(config["database"]["host"])

# Write JSON to file
data = {"debug": True, "port": 8080}
with open("config.json", "w") as f:
    json.dump(data, f, indent=2)  # indent for readability

# Pretty-print JSON
print(json.dumps(data, indent=2))
```

### Common JSON Errors

```python
import json

# KeyError if key doesn't exist
config = {"host": "localhost"}
# port = config["port"]  # KeyError!
port = config.get("port", 8080)  # Use .get() with default

# JSONDecodeError if invalid JSON
try:
    data = json.loads("{'invalid': 'json'}")  # Single quotes!
except json.JSONDecodeError as e:
    print(f"Invalid JSON: {e}")

# Ensure data is serializable
import datetime
date = datetime.datetime.now()
# json.dumps({"date": date})  # TypeError! datetime not JSON serializable

# Solution: convert to string or use custom encoder
json.dumps({"date": str(date)})
```

## Working with YAML (SRE-relevant)

YAML is common in DevOps (Kubernetes, Ansible, Docker Compose). Install with `pip install pyyaml`.

```python
import yaml

# YAML string → Python dict
yaml_string = """
database:
  host: localhost
  port: 5432
  credentials:
    user: admin
    password: secret
"""
data = yaml.safe_load(yaml_string)
print(data["database"]["host"])  # localhost

# Python dict → YAML string
config = {
    "app": {
        "name": "MyApp",
        "version": "1.0"
    }
}
yaml_string = yaml.dump(config, default_flow_style=False)
print(yaml_string)

# Read YAML from file
with open("config.yaml", "r") as f:
    config = yaml.safe_load(f)

# Write YAML to file
with open("config.yaml", "w") as f:
    yaml.dump(config, f)
```

## Working with CSV

CSV (comma-separated values) is common for data files.

```python
import csv

# Read CSV
with open("data.csv", "r") as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)  # ['name', 'age', 'city']

# Read as dictionaries (with headers)
with open("data.csv", "r") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row["name"], row["age"])  # Access by column name

# Write CSV
rows = [
    ["name", "age", "city"],
    ["Alice", 25, "NYC"],
    ["Bob", 30, "LA"],
]
with open("output.csv", "w") as f:
    writer = csv.writer(f)
    writer.writerows(rows)

# Write as dictionaries
data = [
    {"name": "Alice", "age": 25},
    {"name": "Bob", "age": 30},
]
with open("output.csv", "w") as f:
    writer = csv.DictWriter(f, fieldnames=["name", "age"])
    writer.writeheader()
    writer.writerows(data)
```

## File Path Operations

Use `pathlib` (modern) or `os.path` (older).

```python
from pathlib import Path

# Create path objects
p = Path("data.txt")
p = Path("folder/data.txt")
p = Path.home() / "documents" / "file.txt"  # Cleaner joining

# Path operations
p.exists()           # Check if exists
p.is_file()          # Is it a file?
p.is_dir()           # Is it a directory?
p.parent             # Parent directory
p.name               # Just the filename
p.stem               # Filename without extension
p.suffix             # File extension (.txt)
p.read_text()        # Read entire file
p.write_text("data") # Write to file

# List files
folder = Path("data")
for file in folder.glob("*.txt"):  # All .txt files
    print(file)

for file in folder.rglob("*.py"):  # Recursive
    print(file)
```

## Exercises

### Exercise 1: Parse Log File
Write a function that reads a log file and returns a list of error messages.

```python
def extract_errors(log_file):
    """Extract lines containing 'ERROR' from log file"""
    pass

# Example log file:
"""
2026-01-10 10:00:00 INFO Starting application
2026-01-10 10:00:05 ERROR Connection timeout
2026-01-10 10:00:10 INFO Request processed
2026-01-10 10:00:15 ERROR Database unavailable
"""

# Result: ["ERROR Connection timeout", "ERROR Database unavailable"]
```

**Solution:**
```python
def extract_errors(log_file):
    errors = []
    with open(log_file, "r") as f:
        for line in f:
            if "ERROR" in line:
                errors.append(line.strip())
    return errors

# Or more concise:
def extract_errors_v2(log_file):
    with open(log_file, "r") as f:
        return [line.strip() for line in f if "ERROR" in line]
```

### Exercise 2: Config File Reader
Write a function that reads a JSON config file and returns a specific value with a default.

```python
import json

def get_config_value(config_file, *keys, default=None):
    """Get value from nested dict using key path"""
    pass

# Example usage:
# Config: {"database": {"host": "localhost", "port": 5432}}
# get_config_value("config.json", "database", "host") → "localhost"
# get_config_value("config.json", "database", "user") → None (default)
# get_config_value("config.json", "database", "user", default="root") → "root"
```

**Solution:**
```python
import json

def get_config_value(config_file, *keys, default=None):
    with open(config_file, "r") as f:
        config = json.load(f)
    
    value = config
    for key in keys:
        if isinstance(value, dict):
            value = value.get(key)
        else:
            return default
    
    return value if value is not None else default
```

### Exercise 3: Write JSON from User Input
Write a function that collects user information and saves it as JSON.

```python
import json

def save_user_profile(filename):
    """Collect user info and save to JSON file"""
    pass

# Prompts:
# What is your name? Alice
# What is your age? 25
# What is your city? NYC
# Saves to JSON file
```

**Solution:**
```python
import json

def save_user_profile(filename):
    name = input("What is your name? ")
    age = int(input("What is your age? "))
    city = input("What is your city? ")
    
    profile = {
        "name": name,
        "age": age,
        "city": city
    }
    
    with open(filename, "w") as f:
        json.dump(profile, f, indent=2)
    
    print(f"Profile saved to {filename}")
```

## Key Takeaways

✅ **Always use `with` statement** — files close automatically  
✅ `open()` modes: `r` (read), `w` (write), `a` (append)  
✅ Use `json.load()` / `json.dump()` for files, `json.loads()` / `json.dumps()` for strings  
✅ YAML is common in DevOps/SRE (`yaml.safe_load()` is safer than `yaml.load()`)  
✅ `pathlib.Path` is cleaner than `os.path` for modern Python  
✅ `.get()` on dicts to avoid KeyError  
✅ `.strip()` removes newlines from file lines  

## Next Module

[Error Handling](./05-error-handling.md) — Try/except, custom exceptions, and logging
