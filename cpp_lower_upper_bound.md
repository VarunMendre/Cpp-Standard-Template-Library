# C++ STL `lower_bound` & `upper_bound` — Complete Guide

---

## Table of Contents

1. [What Are They?](#1-what-are-they)
2. [The Prerequisite — Array Must Be Sorted](#2-the-prerequisite--array-must-be-sorted)
3. [lower\_bound — Exact Definition](#3-lower_bound--exact-definition)
4. [upper\_bound — Exact Definition](#4-upper_bound--exact-definition)
5. [lower\_bound vs upper\_bound — Visual Comparison](#5-lower_bound-vs-upper_bound--visual-comparison)
6. [For Raw Arrays](#6-for-raw-arrays)
7. [For Vectors](#7-for-vectors)
8. [For Sets and Maps — Member Function vs Algorithm](#8-for-sets-and-maps--member-function-vs-algorithm)
9. [Getting the Index from an Iterator](#9-getting-the-index-from-an-iterator)
10. [Common Patterns](#10-common-patterns)
11. [Edge Cases](#11-edge-cases)
12. [Complexity](#12-complexity)
13. [Quick Reference](#13-quick-reference)

---

## 1. What Are They?

`lower_bound` and `upper_bound` are **binary search functions** from `<algorithm>`.
They don't just tell you if a value exists — they tell you **exactly where it is or where it would be** in a sorted sequence.

```
lower_bound(start, end, x)  →  first position where value >= x
upper_bound(start, end, x)  →  first position where value >  x
```

Both return a **pointer or iterator** — not the value itself, not an index.

---

## 2. The Prerequisite — Array Must Be Sorted

`lower_bound` and `upper_bound` use **binary search** internally.
Binary search only works on sorted data.

```
❌  Unsorted:  [5, 2, 8, 1, 9]  →  results are undefined / wrong
✅  Sorted:    [1, 2, 5, 8, 9]  →  results are correct
```

Always `sort()` before calling these functions — or use a container that's always sorted (`set`, `map`).

---

## 3. `lower_bound` — Exact Definition

> Returns an iterator/pointer to the **first element that is ≥ x** (not less than x).

```
Array:  [1, 3, 5, 5, 7, 9]
                ↑
lower_bound for 5  →  points here (first element >= 5)

Array:  [1, 3, 5, 5, 7, 9]
                      ↑
lower_bound for 6  →  points here (first element >= 6, which is 7)
                      6 doesn't exist — still gives a valid answer
```

Think of it as: **"where would x fit if I inserted it, keeping sorted order?"**
It points to the slot where x belongs, pushing existing elements right.

---

## 4. `upper_bound` — Exact Definition

> Returns an iterator/pointer to the **first element that is > x** (strictly greater).

```
Array:  [1, 3, 5, 5, 7, 9]
                         ↑
upper_bound for 5  →  points here (first element > 5, which is 7)

Array:  [1, 3, 5, 5, 7, 9]
                      ↑
upper_bound for 6  →  points here (first element > 6, which is 7)
```

Think of it as: **"where does x stop? — one past the last occurrence of x"**.

---

## 5. `lower_bound` vs `upper_bound` — Visual Comparison

```
Array:   [ 1 | 3 | 5 | 5 | 7 | 9 ]
index:     0   1   2   3   4   5

Query value: 5

lower_bound →  index 2  (first position where value >= 5)
upper_bound →  index 4  (first position where value >  5)

           lower        upper
              ↓            ↓
[ 1 | 3 | 5 | 5 | 7 | 9 ]
            └────┘
         range of all 5s = [lower_bound, upper_bound)
```

The range `[lower_bound, upper_bound)` captures **all occurrences of x**.
`upper_bound - lower_bound` = count of x in the array.

---

### When x Does Not Exist

```
Array:   [ 1 | 3 | 5 | 5 | 7 | 9 ]
Query:   6

lower_bound(6)  →  index 4  (first element >= 6 is 7)
upper_bound(6)  →  index 4  (first element >  6 is 7)

Both point to the same position — the range is empty.
This means 6 does not exist in the array.
```

```
lower_bound == upper_bound  →  element does NOT exist
lower_bound != upper_bound  →  element EXISTS
```

---

## 6. For Raw Arrays

For a raw array `int arr[n]`, both functions take **raw pointers**.

```
lower_bound(arr, arr + n, x)
upper_bound(arr, arr + n, x)

arr      →  pointer to first element  (start, included)
arr + n  →  pointer past last element (end, excluded)
```

The return type is `int*` — a raw pointer.

### Checking if the element was found

```
int *ptr = lower_bound(arr, arr + n, x);

if (ptr == arr + n) {
    // pointer is at the end — no element >= x exists in the array
    cout << "Not Found";
} else {
    cout << *ptr;   // dereference to get the value
}
```

```
Visual:

arr:       [ 1 | 3 | 5 | 7 | 9 ]
            ↑                    ↑
           arr                arr+5  (end sentinel)

lower_bound for 10:

No element >= 10 exists  →  returns arr+5 (end)
ptr == arr+n  →  Not Found
```

### Getting the index

```
int *ptr = lower_bound(arr, arr + n, x);
int index = ptr - arr;   // pointer arithmetic gives the index
```

---

## 7. For Vectors

For vectors, use iterators `.begin()` and `.end()`.
The return type is `vector<int>::iterator` — use `auto` to keep it clean.

```
auto it1 = lower_bound(arr.begin(), arr.end(), x);
auto it2 = upper_bound(arr.begin(), arr.end(), x);
```

### Checking if found

```
auto it = lower_bound(arr.begin(), arr.end(), x);

if (it == arr.end()) {
    cout << "Not Found";
} else {
    cout << *it;   // dereference to get the value
}
```

### Getting the index

```
auto it = lower_bound(arr.begin(), arr.end(), x);
int index = it - arr.begin();
```

---

## 8. For Sets and Maps — Member Function vs Algorithm

This is the **most important distinction** in this entire topic.

### Two ways to call `lower_bound` on a set

```
// Way 1 — std::lower_bound from <algorithm>
auto it = lower_bound(st.begin(), st.end(), 10);    ❌  O(n) — TLE on large input

// Way 2 — member function on the set itself
auto it = st.lower_bound(10);                       ✅  O(log n) — correct and fast
```

### Why the Difference?

```
std::lower_bound (algorithm version):
  Uses binary search, but needs RANDOM-ACCESS iterators to jump to the middle.
  set iterators are BIDIRECTIONAL — they can only move one step at a time.
  So std::lower_bound falls back to LINEAR SCAN on sets → O(n).

set::lower_bound (member function):
  The set knows its own internal Red-Black Tree structure.
  It traverses the tree directly → O(log n).
```

```
                std::lower_bound on set            set::lower_bound
                ────────────────────────           ─────────────────
                start                              root
                  ↓                                  ↓
                [1]→[2]→[3]→...→[n]           [5]
                  step, step, step...           ↙     ↘
                  (linear scan)              [2]       [8]
                                           ↙   ↘     ↙
                                         [1]  [3]  [6]
                                         (tree traversal — O(log n))
```

**Rule: always use the member function for `set` and `map`.**

```
set<int> st;
st.lower_bound(x);    ✅  O(log n)
st.upper_bound(x);    ✅  O(log n)

map<int,int> mpp;
mpp.lower_bound(x);   ✅  O(log n) — searches by key
mpp.upper_bound(x);   ✅  O(log n) — searches by key
```

---

## 9. Getting the Index from an Iterator

| Container | How to get index |
|-----------|-----------------|
| Raw array | `ptr - arr` |
| Vector | `it - arr.begin()` |
| Set / Map | ❌ Not meaningful — sets have no index concept |

```
vector<int> arr = {1, 3, 5, 7, 9};
auto it = lower_bound(arr.begin(), arr.end(), 5);

int index = it - arr.begin();   →   index = 2
int value = *it;                →   value = 5
```

---

## 10. Common Patterns

---

### Count occurrences of x

```
auto lo = lower_bound(arr.begin(), arr.end(), x);
auto hi = upper_bound(arr.begin(), arr.end(), x);

int count = hi - lo;

Array:  [1, 3, 5, 5, 5, 7]   x = 5
lo → index 2,  hi → index 5
count = 5 - 2 = 3  ✅
```

---

### Check if x exists

```
auto it = lower_bound(arr.begin(), arr.end(), x);

bool exists = (it != arr.end()) && (*it == x);

// lower_bound gives first element >= x
// if that element equals x → x exists
// if it's > x or end() → x doesn't exist
```

---

### Find the largest element ≤ x (floor of x)

```
auto it = upper_bound(arr.begin(), arr.end(), x);
// it points to first element > x
// one step back → last element <= x

if (it == arr.begin()) {
    // no element <= x exists
} else {
    --it;
    cout << *it;   // largest element <= x
}

Array:  [1, 3, 5, 7, 9]   x = 6
upper_bound(6) → points to 7 (index 3)
--it → points to 5 (index 2)
Answer: 5  ✅
```

---

### Find the smallest element ≥ x (ceiling of x)

```
auto it = lower_bound(arr.begin(), arr.end(), x);

if (it == arr.end()) {
    // no element >= x exists
} else {
    cout << *it;   // smallest element >= x
}

Array:  [1, 3, 5, 7, 9]   x = 4
lower_bound(4) → points to 5 (index 2)
Answer: 5  ✅
```

---

## 11. Edge Cases

---

### x is smaller than all elements

```
Array:  [3, 5, 7, 9]   x = 1

lower_bound(1)  →  index 0  (first element, since all elements >= 1)
upper_bound(1)  →  index 0  (first element > 1)

Both point to the beginning.
```

---

### x is larger than all elements

```
Array:  [1, 3, 5, 7]   x = 10

lower_bound(10)  →  end()  (no element >= 10)
upper_bound(10)  →  end()  (no element > 10)

Both point to end — always check before dereferencing!
```

---

### Dereferencing end() — Undefined Behaviour

```
auto it = lower_bound(arr.begin(), arr.end(), x);

cout << *it;            ❌  if it == arr.end(), this is UB

if (it != arr.end()) {
    cout << *it;        ✅  safe
}
```

---

### x exists exactly once vs multiple times

```
Array:  [1, 3, 5, 7, 9]   x = 5  (exists once)

lower_bound → index 2 (the 5)
upper_bound → index 3 (the 7)
count = 3 - 2 = 1  ✅

Array:  [1, 5, 5, 5, 9]   x = 5  (exists three times)

lower_bound → index 1 (first 5)
upper_bound → index 4 (the 9)
count = 4 - 1 = 3  ✅
```

---

### Unsorted array — wrong results

```
Array (unsorted):  [5, 2, 8, 1, 9]

lower_bound(5)  →  could return anything — WRONG

Always sort first:  [1, 2, 5, 8, 9]
lower_bound(5)  →  index 2  ✅
```

---

### Using `std::lower_bound` on a `set` — TLE

```
set<int> st = {1, 3, 5, 7, 9};

auto it = lower_bound(st.begin(), st.end(), 5);   ❌  O(n) — wrong
auto it = st.lower_bound(5);                      ✅  O(log n) — correct
```

---

## 12. Complexity

| Function | Container | TC | Why |
|----------|-----------|-----|-----|
| `lower_bound(arr, arr+n, x)` | Raw array | O(log n) | Random-access → binary search |
| `lower_bound(v.begin(), v.end(), x)` | Vector | O(log n) | Random-access → binary search |
| `lower_bound(st.begin(), st.end(), x)` | Set | O(n) ❌ | Bidirectional iterator → linear scan |
| `st.lower_bound(x)` | Set | O(log n) ✅ | Uses tree structure directly |
| `mpp.lower_bound(x)` | Map | O(log n) ✅ | Uses tree structure directly |

Space complexity: **O(1)** for all — no extra allocation.

---

## 13. Quick Reference

```
┌─────────────────────────────────────────────────────────────────────┐
│  What you want               │  How to do it                       │
│──────────────────────────────┼──────────────────────────────────── │
│  First element >= x          │  lower_bound(begin, end, x)         │
│  First element >  x          │  upper_bound(begin, end, x)         │
│  Check if x exists           │  it != end && *it == x              │
│  Count occurrences of x      │  upper_bound - lower_bound          │
│  Largest element <= x        │  --upper_bound(begin, end, x)       │
│  Smallest element >= x       │  lower_bound(begin, end, x)         │
│  Index of lower_bound        │  it - arr.begin()  (vector/array)   │
│  On set/map                  │  st.lower_bound(x)  NOT std::       │
└─────────────────────────────────────────────────────────────────────┘
```

### Summary of Return Values

```
lower_bound  →  first position where value >= x
upper_bound  →  first position where value >  x

If result == end()  →  element not found / all elements smaller
Always check != end() before dereferencing
```

---

> **One-line memory aid:**
> `lower_bound` = first **≥ x** · `upper_bound` = first **> x** · for sets/maps always use the **member function**, not `std::lower_bound`
