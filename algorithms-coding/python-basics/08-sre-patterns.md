# 08: SRE-Specific Patterns

Real-world patterns for SRE work: APIs, CLIs, environment variables, and data handling.

## Making HTTP Requests

The `requests` library is essential for APIs. Install with `pip install requests`.

```python
import requests
import json

# GET request
response = requests.get("https://api.example.com/users")
print(response.status_code)     # 200, 404, 500, etc.
print(response.text)             # Raw response text
data = response.json()           # Parse JSON

# Query parameters
params = {"page": 1, "limit": 10}
response = requests.get("https://api.example.com/users", params=params)
# URL becomes: https://api.example.com/users?page=1&limit=10

# POST request
payload = {"name": "Alice", "email": "alice@example.com"}
response = requests.post("https://api.example.com/users", json=payload)

# Headers
headers = {"Authorization": "Bearer TOKEN123"}
response = requests.get("https://api.example.com/data", headers=headers)

# Error handling
try:
    response = requests.get("https://api.example.com/data", timeout=5)
    response.raise_for_status()  # Raise exception for bad status codes
except requests.exceptions.Timeout:
    print("Request timed out")
except requests.exceptions.ConnectionError:
    print("Connection failed")
except requests.exceptions.HTTPError as e:
    print(f"HTTP error: {e.response.status_code}")

# Check response before accessing data
if response.status_code == 200:
    data = response.json()
else:
    print(f"Error: {response.status_code}")
```

## Working with Environment Variables

Store secrets and config in environment, not code.

```python
import os

# Read environment variables
api_key = os.getenv("API_KEY")
database_url = os.getenv("DATABASE_URL", "localhost")  # Default value
debug = os.getenv("DEBUG", "False") == "True"

# Check if variable exists
if "API_KEY" not in os.environ:
    raise ValueError("API_KEY not set")

# Set environment variable (in this process only)
os.environ["LOG_LEVEL"] = "DEBUG"

# Access current environment
all_vars = os.environ  # Dict-like object
```

### Example: Health Check Script

```python
import os
import requests

api_endpoint = os.getenv("API_ENDPOINT", "http://localhost:8080")
timeout = int(os.getenv("TIMEOUT", "5"))

try:
    response = requests.get(f"{api_endpoint}/health", timeout=timeout)
    response.raise_for_status()
    print("✓ Service is healthy")
except Exception as e:
    print(f"✗ Service is down: {e}")
    exit(1)
```

## Command-Line Arguments

Parse CLI arguments with `argparse` module.

```python
import argparse

parser = argparse.ArgumentParser(description="Deploy application")
parser.add_argument("--environment", required=True, help="Target environment (dev, staging, prod)")
parser.add_argument("--version", required=True, help="Version to deploy")
parser.add_argument("--dry-run", action="store_true", help="Simulate deployment")
parser.add_argument("--verbose", "-v", action="store_true", help="Verbose output")

args = parser.parse_args()

# Access arguments
print(f"Deploying {args.version} to {args.environment}")
if args.dry_run:
    print("DRY RUN - no changes will be made")
if args.verbose:
    print("Verbose mode enabled")
```

**Usage:**
```bash
python deploy.py --environment prod --version 1.2.0 --dry-run -v
```

## Working with Dates and Times

```python
from datetime import datetime, timedelta
import time

# Current time
now = datetime.now()
print(f"Current time: {now}")

# Parse time string
time_string = "2026-01-10 10:00:05"
dt = datetime.strptime(time_string, "%Y-%m-%d %H:%M:%S")

# Format time to string
formatted = dt.strftime("%Y-%m-%d %H:%M:%S")
print(formatted)  # "2026-01-10 10:00:05"

# Calculate time differences
future = now + timedelta(hours=1, minutes=30)
delta = future - now
print(f"Difference: {delta.total_seconds()} seconds")

# Unix timestamp
timestamp = now.timestamp()  # Seconds since 1970-01-01
from_timestamp = datetime.fromtimestamp(timestamp)

# Check if time is in the past
if now > some_deadline:
    print("Deadline has passed")
```

### Example: Log Time Parser

```python
from datetime import datetime

def parse_log_timestamp(log_line):
    """Extract timestamp from log line"""
    # Log format: "2026-01-10 10:00:05 ERROR Message"
    time_str = " ".join(log_line.split()[:2])
    try:
        return datetime.strptime(time_str, "%Y-%m-%d %H:%M:%S")
    except ValueError:
        return None

def is_recent(log_line, hours=1):
    """Check if log entry is from the last N hours"""
    log_time = parse_log_timestamp(log_line)
    if not log_time:
        return False
    
    from datetime import timedelta
    cutoff = datetime.now() - timedelta(hours=hours)
    return log_time > cutoff

log = "2026-01-10 10:00:05 ERROR Connection timeout"
print(is_recent(log, hours=24))  # True if within last 24 hours
```

## Configuration Management

```python
import os
import json

class Config:
    """Configuration management"""
    
    def __init__(self, config_file=None):
        self.defaults = {
            "host": "localhost",
            "port": 8080,
            "debug": False,
            "timeout": 30,
        }
        
        # Load from environment
        self.config = self.defaults.copy()
        self.config.update(os.environ)
        
        # Load from file if provided
        if config_file and os.path.exists(config_file):
            with open(config_file) as f:
                file_config = json.load(f)
                self.config.update(file_config)
    
    def get(self, key, default=None):
        return self.config.get(key, default)
    
    def validate(self):
        """Ensure required fields exist"""
        required = ["host", "port"]
        for field in required:
            if field not in self.config:
                raise ValueError(f"Missing required config: {field}")
        return True

# Usage
config = Config("config.json")
config.validate()
host = config.get("host")
port = int(config.get("port", 8080))
```

## Exercises

### Exercise 1: Health Check Aggregator
Write a function that checks multiple services and reports overall health.

```python
def check_health(services):
    """Check health of multiple services, return results"""
    pass

# Test case:
services = {
    "api": "http://localhost:8080/health",
    "db": "http://localhost:5432",
    "cache": "http://localhost:6379",
}

results = check_health(services)
# Returns: {"api": True, "db": False, "cache": True}
```

**Solution:**
```python
import requests

def check_health(services):
    results = {}
    for name, url in services.items():
        try:
            response = requests.get(url, timeout=2)
            results[name] = response.status_code == 200
        except Exception as e:
            results[name] = False
    return results
```

### Exercise 2: Configuration Validator
Write a function that validates a configuration dictionary.

```python
def validate_config(config):
    """Validate required fields and types"""
    pass

# Test case:
config = {
    "host": "localhost",
    "port": 8080,
    "debug": True,
}

validate_config(config)  # Should pass
# Require: host (str), port (int), debug (bool)
```

**Solution:**
```python
def validate_config(config):
    required_fields = {
        "host": str,
        "port": int,
        "debug": bool,
    }
    
    for field, expected_type in required_fields.items():
        if field not in config:
            raise ValueError(f"Missing field: {field}")
        if not isinstance(config[field], expected_type):
            raise TypeError(f"Field {field} should be {expected_type.__name__}")
    
    return True
```

### Exercise 3: Log Aggregator
Write a function that reads a log file and filters by time range.

```python
from datetime import datetime

def filter_logs_by_time(log_file, start_time, end_time):
    """Return logs within time range"""
    pass

# Log format: "2026-01-10 10:00:05 ERROR Message"
# start_time and end_time are datetime objects
```

**Solution:**
```python
from datetime import datetime

def filter_logs_by_time(log_file, start_time, end_time):
    results = []
    with open(log_file, "r") as f:
        for line in f:
            try:
                time_str = " ".join(line.split()[:2])
                log_time = datetime.strptime(time_str, "%Y-%m-%d %H:%M:%S")
                
                if start_time <= log_time <= end_time:
                    results.append(line.strip())
            except (ValueError, IndexError):
                continue
    
    return results
```

## Key Takeaways

✅ Use `requests` library for HTTP calls  
✅ Store secrets in environment variables, not code  
✅ Use `argparse` for CLI tools  
✅ Handle request timeouts and connection errors  
✅ Always check `response.status_code` before using data  
✅ Use `datetime` for time operations  
✅ Parse timestamps with `strptime()`, format with `strftime()`  
✅ Centralize configuration in a Config class  
✅ Validate input early and fail fast  

## Next Module

[Interview Problems](./09-interview-problems.md) — 15+ medium difficulty practice problems
