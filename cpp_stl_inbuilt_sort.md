# C++ STL `sort()` — Complete Guide

---

## Table of Contents

1. [What Is `sort()`?](#1-what-is-sort)
2. [How It Works — Introsort](#2-how-it-works--introsort)
3. [The Two Parameters — Pointers / Iterators](#3-the-two-parameters--pointers--iterators)
4. [Sorting Arrays vs Vectors](#4-sorting-arrays-vs-vectors)
5. [Partial Sort — Sorting a Range](#5-partial-sort--sorting-a-range)
6. [Custom Comparator](#6-custom-comparator)
7. [Edge Cases](#7-edge-cases)
8. [Time & Space Complexity](#8-time--space-complexity)
9. [Quick Reference](#9-quick-reference)

---

## 1. What Is `sort()`?

`std::sort` is STL's built-in sorting function.
It lives in `<algorithm>` (included via `<bits/stdc++.h>`).

It sorts elements **in-place** — the original container is modified directly, nothing is returned.
By default it sorts in **ascending order**.

```
Input:   [5, 2, 8, 1, 9, 3]
             ↓  sort()
Output:  [1, 2, 3, 5, 8, 9]   (ascending, in-place)
```

---

## 2. How It Works — Introsort

`std::sort` is **not** a simple quicksort.
It is called **Introsort** — a hybrid of three algorithms, each used in the situation it's best at:

```
                  ┌──────────────────────────────────┐
                  │           Introsort              │
                  │                                  │
                  │   ┌────────────┐                 │
                  │   │ Quick Sort │  main workhorse  │
                  │   └─────┬──────┘                 │
                  │         │ too many recursive      │
                  │         │ calls? (depth > 2·logN) │
                  │         ▼                         │
                  │   ┌────────────┐                 │
                  │   │ Heap Sort  │  saves worst case│
                  │   └─────┬──────┘                 │
                  │         │ sub-array small         │
                  │         │ (size ≤ ~16 elements)   │
                  │         ▼                         │
                  │   ┌──────────────────┐           │
                  │   │ Insertion Sort   │  fast on   │
                  │   │                  │  tiny data │
                  │   └──────────────────┘           │
                  └──────────────────────────────────┘
```

### Why Three Algorithms?

| Algorithm | Best At | Weakness |
|-----------|---------|----------|
| Quick Sort | Large arrays — fast in practice, O(n log n) avg | Can degrade to O(n²) on bad pivot choices |
| Heap Sort | Guaranteed O(n log n) always | Slower in practice due to cache misses |
| Insertion Sort | Tiny arrays (≤ ~16 elements) | O(n²) on large arrays |

Introsort starts with quicksort for speed.
If quicksort's recursion depth exceeds `2 · log(n)` — a sign it's hitting a bad case — it switches to heapsort to guarantee O(n log n).
For small sub-arrays, it uses insertion sort because the overhead of quicksort isn't worth it at that size.

**Result:** O(n log n) guaranteed, fast in practice. Best of all three worlds.

---

## 3. The Two Parameters — Pointers / Iterators

`sort()` takes exactly **two parameters**:

```
sort( start ,  end )
       │          │
       │          └──  pointer/iterator to ONE PAST the last element to sort
       └─────────────  pointer/iterator to the first element to sort
```

This is the **half-open range** convention used throughout STL: `[start, end)`.
- `start` is **included**
- `end` is **excluded** (one past the last)

```
Array:   [ 5 | 2 | 8 | 1 | 9 ]
index:     0   1   2   3   4

sort(arr, arr + 5)
           │        │
           │        └── arr + 5 = one past index 4 (past the end)
           └─────────── arr     = address of index 0

Sorts indices 0, 1, 2, 3, 4  →  entire array
```

---

## 4. Sorting Arrays vs Vectors

### Raw Array

For a raw array `int arr[n]`, the array name itself is a pointer to its first element.

```
sort(arr, arr + n);

arr      →  pointer to arr[0]   (start)
arr + n  →  pointer past arr[n-1]  (end, excluded)
```

```
arr:    [ 5 | 2 | 8 | 1 ]    n = 4
         ↑               ↑
        arr           arr+4  (past the end)

sort(arr, arr+4)  →  sorts all 4 elements
```

### Vector

For a `vector<int> arr(n)`, use iterators `.begin()` and `.end()`:

```
sort(arr.begin(), arr.end());

arr.begin()  →  iterator to first element
arr.end()    →  iterator to one past last element
```

```
vector:  [ 5 | 2 | 8 | 1 ]
           ↑               ↑
        begin()           end()

sort(arr.begin(), arr.end())  →  sorts all elements
```

### Comparison

```
Raw array:   sort(arr, arr + n)
Vector:      sort(arr.begin(), arr.end())

Both do exactly the same thing — sort the entire container.
```

---

## 5. Partial Sort — Sorting a Range

You don't have to sort the entire container.
By changing the start and end pointers/iterators, you sort only a **slice**.

```
Vector:  [ 9 | 4 | 7 | 2 | 5 | 1 ]
index:     0   1   2   3   4   5

sort(arr.begin() + 2, arr.end())
           │                  │
           └── starts at index 2 (value 7)
                              └── ends past index 5

Sorts indices 2, 3, 4, 5 only:

Before:  [ 9 | 4 | 7 | 2 | 5 | 1 ]
                   └───────────────┘  ← this part gets sorted
After:   [ 9 | 4 | 1 | 2 | 5 | 7 ]
          ↑   ↑
       untouched   untouched
```

Elements **before** the start pointer are untouched.
Only the specified range is sorted.

### More Range Examples

```
sort(arr.begin(), arr.begin() + 3)
  →  sorts only first 3 elements (indices 0, 1, 2)

sort(arr.begin() + 1, arr.begin() + 4)
  →  sorts indices 1, 2, 3  (middle slice)

sort(arr.begin(), arr.end())
  →  sorts entire vector

sort(arr.end(), arr.end())
  →  empty range — does nothing (valid, not UB)
```

---

## 6. Custom Comparator

By default, `sort` sorts in ascending order.
You can pass a **third parameter** — a comparator function — to change the order or sorting logic.

### Descending Order

```
sort(arr.begin(), arr.end(), greater<int>());

Before:  [5, 2, 8, 1, 9]
After:   [9, 8, 5, 2, 1]
```

### Custom Lambda

```
// Sort by absolute value
sort(arr.begin(), arr.end(), [](int a, int b) {
    return abs(a) < abs(b);
});

Input:   [-5, 2, -1, 8, -3]
Output:  [-1, 2, -3, -5, 8]
```

### Comparator Rule

The comparator must return `true` if `a` should come **before** `b`.
It must define a **strict weak ordering** — if you break this rule (e.g., return `true` when `a == b`), behaviour is undefined.

```
✅ return a < b;    ascending
✅ return a > b;    descending
✅ return abs(a) < abs(b);   by absolute value

❌ return a <= b;   NOT strict — undefined behaviour
❌ return true;     always — undefined behaviour
```

---

## 7. Edge Cases

---

### Empty Range

```
sort(arr.begin(), arr.end())  on empty vector
→  does nothing, no crash
```

---

### Single Element

```
sort(arr.begin(), arr.end())  with one element
→  does nothing, already sorted
```

---

### All Elements Equal

```
Input:   [3, 3, 3, 3]
Output:  [3, 3, 3, 3]   (unchanged, valid)
```

---

### `arr.begin() + 2` on a vector with fewer than 2 elements

```
vector<int> arr = {5};
sort(arr.begin() + 2, arr.end());   ❌  undefined behaviour

arr.begin() + 2 goes past arr.end() — iterator out of bounds
Always verify the vector has enough elements before offsetting begin().
```

---

### `sort` on `set` or `map` — Not Allowed

```
set<int> st = {3, 1, 2};
sort(st.begin(), st.end());   ❌  compile error

set iterators are bidirectional, not random-access.
sort() requires random-access iterators.
```

Works on: `vector`, `array`, `deque`, raw arrays.
Does NOT work on: `set`, `map`, `list`.

---

### Stability — `sort` Is Not Stable

If two elements compare equal, `sort` does not guarantee their relative order is preserved.

```
// Two students with same marks:
{"Alice", 80}  and  {"Bob", 80}

After sort by marks:
Could be:  Alice, Bob   OR   Bob, Alice  — not defined

Use std::stable_sort() if original relative order must be preserved.
stable_sort TC: O(n log² n) without extra memory, O(n log n) with extra memory.
```

---

## 8. Time & Space Complexity

### Time Complexity

```
┌────────────────────────────────────────────────┐
│              Introsort (std::sort)             │
│                                                │
│  Best Case    →   O(n log n)                  │
│  Average Case →   O(n log n)                  │
│  Worst Case   →   O(n log n)  ← guaranteed    │
│               (heapsort fallback prevents O(n²))│
└────────────────────────────────────────────────┘
```

Partial sort on a range of size k:
```
sort(arr.begin() + i, arr.begin() + i + k)
→  O(k log k)   (only k elements are sorted)
```

### Space Complexity

```
O(log n)  →  recursion stack depth from quicksort
```

`sort` is in-place — no extra array is allocated.
The O(log n) space is just the call stack from recursion.

---

## 9. Quick Reference

```
┌──────────────────────────────────────────────────────────────────┐
│  Use Case                   │  Call                             │
│─────────────────────────────┼───────────────────────────────────│
│  Sort full raw array        │  sort(arr, arr + n)               │
│  Sort full vector           │  sort(v.begin(), v.end())         │
│  Sort descending            │  sort(v.begin(), v.end(),         │
│                             │       greater<int>())             │
│  Sort from index i to end   │  sort(v.begin()+i, v.end())       │
│  Sort first k elements      │  sort(v.begin(), v.begin()+k)     │
│  Sort middle slice [i, j)   │  sort(v.begin()+i, v.begin()+j)  │
│  Custom comparator          │  sort(v.begin(), v.end(), cmp)    │
│  Stable sort                │  stable_sort(v.begin(), v.end())  │
└──────────────────────────────────────────────────────────────────┘
```

### Complexity Summary

| What | TC | SC |
|------|----|----|
| Full sort | O(n log n) | O(log n) |
| Partial sort (range size k) | O(k log k) | O(log k) |
| `stable_sort` | O(n log n) with extra mem | O(n) |

---

> **One-line memory aid:**
> `sort(start, end)` — start is included, end is excluded — and Introsort guarantees O(n log n) always by switching from quicksort → heapsort → insertion sort as needed.
