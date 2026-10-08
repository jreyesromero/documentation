# Python Basics for SRE Interview Prep

This module provides **foundational Python knowledge** tailored for SRE/DevOps technical interviews with 30-minute live coding sessions at medium difficulty. The goal is to build fluency and confidence in Python before tackling complex algorithms.

## How to Use This Guide

1. **Read sequentially** — each module builds on the previous
2. **Type every example** — don't just read, execute code
3. **Do the exercises** — solve them without looking at solutions first
4. **Time yourself** — practice working under interview time pressure

**Estimated time:** 8-12 hours total (can be done in 1-2 weeks with daily practice)

## Module Overview

| Module | Topic | Time | Focus |
|--------|-------|------|-------|
| 01 | Fundamentals | 1.5h | Variables, types, control flow |
| 02 | Data Structures | 2h | Lists, dicts, sets, tuples |
| 03 | Functions & Scope | 1.5h | Functions, *args/**kwargs, scope |
| 04 | File Handling | 1.5h | I/O, JSON, YAML, context managers |
| 05 | Error Handling | 1h | Try/except, custom exceptions, logging |
| 06 | Strings & Regex | 1.5h | String methods, f-strings, regex |
| 07 | Common Algorithms | 2h | Searching, sorting, two-pointer technique |
| 08 | SRE Patterns | 1.5h | APIs, CLI args, environment, timestamps |
| 09 | Interview Problems | 2h | 15-20 medium difficulty practice problems |

## What This IS

✅ **Python fundamentals** — syntax, data structures, best practices  
✅ **SRE-focused examples** — logs, configs, APIs, command-line tools  
✅ **Interview-ready** — problem-solving patterns and time management  
✅ **Hands-on exercises** — with solutions and explanations  

## What This ISN'T

❌ Advanced Python (decorators, metaclasses, async/await — not needed for this interview level)  
❌ Web frameworks (Django, Flask)  
❌ Data science (pandas, numpy)  
❌ A substitute for actually coding — you learn by doing

## Prerequisites

- Python 3.8+ installed
- A code editor (VS Code, PyCharm, etc.)
- 15-30 minutes per session
- Willingness to struggle and debug

## Quick Setup

```bash
# Check Python version
python3 --version

# Create a practice directory
mkdir -p ~/practice-python
cd ~/practice-python

# Create a virtual environment (optional but recommended)
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

## Interview Tips Upfront

### Time Management (30 minutes)
```
5 min:  Understand the problem, ask clarifying questions
5 min:  Think through approach, discuss trade-offs
15 min: Write code (clean, readable, error-handled)
3 min:  Test with examples and edge cases
2 min:  Discuss complexity, potential optimizations
```

### Communication Checklist
- **Clarify** — "So the input is a list of integers, and I need to find...?"
- **Think out loud** — "I'm going to use a dictionary here because O(1) lookup..."
- **Name things well** — `seen_values` not `x` or `temp`
- **Handle errors** — Check for `None`, empty lists, edge cases
- **Explain complexity** — "This is O(n) time, O(n) space because..."

### Common Gotchas in Python
1. **Mutable default arguments** — `def foo(items=[]):` → bad! Use `None` instead
2. **Off-by-one errors** — Remember lists are 0-indexed
3. **List slicing doesn't include the end** — `[0:3]` is indices 0, 1, 2 (not 3)
4. **Modifying a list while iterating** — Use list comprehensions or iterate backward
5. **Shallow vs deep copy** — `copy()` is shallow, use `copy.deepcopy()`

## Study Plan

### Week 1: Foundations
- [ ] Read 01-Fundamentals
- [ ] Read 02-Data Structures
- [ ] Do exercises in both
- [ ] Review gotchas

### Week 2: Intermediate
- [ ] Read 03-Functions & Scope
- [ ] Read 04-File Handling
- [ ] Read 05-Error Handling
- [ ] Do all exercises

### Week 3: Applied Skills
- [ ] Read 06-Strings & Regex
- [ ] Read 07-Common Algorithms
- [ ] Read 08-SRE Patterns
- [ ] Solve at least 5 interview problems

### Week 4: Practice Under Pressure
- [ ] Re-solve problems from memory (no looking!)
- [ ] Time yourself — aim for easy in 10 min, medium in 20 min
- [ ] Practice out loud (pretend interviewer is listening)
- [ ] Review solutions and learn alternative approaches

## Running Your Code

```bash
# In interactive mode (REPL)
python3
>>> x = 5
>>> print(x * 2)

# Run a file
python3 my_script.py

# With command-line arguments
python3 my_script.py arg1 arg2

# Debug with print statements
python3 -u my_script.py  # Unbuffered output
```

## Resources

- **Python Docs** — https://docs.python.org/3/
- **Real Python** — https://realpython.com (tutorials)
- **LeetCode** — Practice the medium-difficulty problems once you finish this
- **This folder** — All modules have exercises with solutions

---

**Remember:** The goal isn't to become a Python expert. It's to be confident, fluent, and able to solve problems clearly in an interview setting. Focus on **thinking clearly** and **communicating well** — the code follows.

Good luck! 🚀
