---
title: "FP-Style Python"
date: 2019-11-24T16:47:31-05:00
draft: true
---

Structuring your codebase around data structures
[The Life of a File](https://www.youtube.com/watch?v=XpDsk374LDE)

- immutable data types
- pure functions
- union types
- ergonomic import / namespacing
- type annotations + mypy static checking


```python
import lib.Type as Type

t1 = Type.Type(*args, **kwargs)
t2 = Type.function1(t1)
```

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Type:
  param1: str
  param2: int
  param3: List[pd.DataFrame]

def function1(t: Type) -> Type:
  pass
```
