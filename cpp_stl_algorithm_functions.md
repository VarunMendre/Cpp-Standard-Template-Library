# C++ STL Algorithm Functions — Complete Guide

> Covers `min_element`, `max_element`, `accumulate`, `count`, `find`, and `reverse`  
> — with how they work internally, diagrams, edge cases, and complexity.

---

## Table of Contents

1. [The `<algorithm>` Library — How These Functions Work](#1-the-algorithm-library--how-these-functions-work)
2. [The Iterator Range Convention](#2-the-iterator-range-convention)
3. [`min_element` and `max_element`](#3-min_element-and-max_element)
4. [`accumulate`](#4-accumulate)
5. [`count`](#5-count)
6. [`find`](#6-find)
7. [`reverse`](#7-reverse)
8. [Shrinking the Search Space](#8-shrinking-the-search-space)
9. [Mutating vs Non-Mutating Functions](#9-mutating-vs-non-mutating-functions)
10. [Complexity Summary](#10-complexity-summary)
11. [Quick Reference](#11-quick-reference)

---

## 1. The `<algorithm>` Library — How These Functions Work

All these functions live in `<algorithm>` (pulled in by `<bits/stdc++.h>`).

They are **generic functions** — they don't know or care whether you pass an array, vector, string, or deque.
They only need two things:

```
1. A start iterator (where to begin)
2. An end iterator   (where to stop — excluded)
```

Internally, every one of these functions works the same way at the core:

```
start iterator
     │
     ▼
  element 0  ──→  element 1  ──→  element 2  ──→  ...  ──→  element n-1
                                                                   │
                                                              end iterator
                                                            (not visited)
```

They walk from `start` to just before `end`, visiting each element exactly once — **O(n) linear scan**.

This is why:
- They work on **any** container with iterators.
- They always need the array/vector to be in scope — they don't copy data, they read through your original.

---

## 2. The Iterator Range Convention

Every function uses the same **half-open range `[start, end)`**:

```
vector: [ 3 | 7 | 1 | 9 | 4 ]
          ↑                   ↑
       begin()             end()
       (included)          (excluded — points past last element)
```

- `begin()` — iterator to the first element — **included**
- `end()` — iterator to one past the last element — **not included, never dereferenced**

You can also pass a sub-range to only process part of the container:

```
arr.begin() + 1  to  arr.end() - 1
   → skips first and last element, processes the middle
```

---

## 3. `min_element` and `max_element`

### What They Return

Both return an **iterator** (pointer) to the minimum or maximum element — not the value itself.
To get the value, dereference with `*`.

```
auto it  = min_element(arr.begin(), arr.end());  // iterator
auto val = *min_element(arr.begin(), arr.end()); // value  ← dereference
```

### How They Work Internally

Both do a single linear scan, tracking the current best seen so far:

```
min_element — internal logic:

current_min = first element
iterator    = begin

for each element from begin+1 to end:
    if element < current_min:
        current_min = element
        iterator    = current position

return iterator
```

Visualized on `[3, 7, 1, 9, 4]`:

```
Step 0:  current_min = 3  (start with first element)

         [ 3 | 7 | 1 | 9 | 4 ]
           ↑
         current_min = 3

Step 1:  visit 7 → 7 > 3 → no update

         [ 3 | 7 | 1 | 9 | 4 ]
               ↑
         current_min = 3  (no change)

Step 2:  visit 1 → 1 < 3 → UPDATE

         [ 3 | 7 | 1 | 9 | 4 ]
                   ↑
         current_min = 1  ← new minimum

Step 3:  visit 9 → 9 > 1 → no update
Step 4:  visit 4 → 4 > 1 → no update

Return iterator pointing to index 2  →  *it = 1
```

`max_element` works identically but tracks the largest value instead.

### Edge Cases

- **Empty range** — calling on an empty container returns `end()`. Dereferencing it is **undefined behaviour** — always check `!arr.empty()` first.
- **Multiple occurrences of min/max** — returns iterator to the **first** occurrence.
- **Single element** — returns iterator to that element.

```
vector<int> arr = {5, 1, 1, 9, 9};

min_element → points to index 1 (first 1)
max_element → points to index 3 (first 9)
```

| Case | TC |
|------|----|
| Any | O(n) — single linear scan |

**Space:** O(1) — only stores one iterator and one comparison value.

---

## 4. `accumulate`

### What It Does

Computes the **sum of all elements** in a range, starting from an initial value.

```
accumulate(start, end, initial_value)
                        ↑
                  added to the sum first — usually 0
```

### How It Works Internally

```
sum = initial_value

for each element from start to end:
    sum = sum + element

return sum
```

Visualized on `[3, 7, 1, 9, 4]` with initial value `0`:

```
sum = 0

Visit 3 → sum = 0 + 3 = 3
Visit 7 → sum = 3 + 7 = 10
Visit 1 → sum = 10 + 1 = 11
Visit 9 → sum = 11 + 9 = 20
Visit 4 → sum = 20 + 4 = 24

Return 24
```

### The Initial Value Parameter — Why It Matters

The initial value is **not optional** — it must be provided.

```
accumulate(arr.begin(), arr.end(), 0)    →  sum of integers
accumulate(arr.begin(), arr.end(), 0LL)  →  sum as long long (prevents overflow)
accumulate(arr.begin(), arr.end(), 1)    →  product? No — still adds. Use different approach for product.
```

**Overflow trap:**

```
vector<int> arr = {1000000, 1000000, 1000000};

accumulate(arr.begin(), arr.end(), 0)    ❌  0 is an int → result stored as int → overflow
accumulate(arr.begin(), arr.end(), 0LL)  ✅  0LL is long long → result stored as long long
```

The return type of `accumulate` matches the type of the initial value — not the element type.

### Edge Cases

- Empty range → returns the initial value unchanged.
- Initial value `0` with large numbers → use `0LL` to avoid integer overflow.
- Works on strings too: `accumulate(v.begin(), v.end(), string(""))` concatenates strings.

| Case | TC |
|------|----|
| Any | O(n) |

**Space:** O(1)

---

## 5. `count`

### What It Does

Counts how many times a **specific value** appears in a range.

```
count(start, end, value)  →  returns int (number of occurrences)
```

### How It Works Internally

```
counter = 0

for each element from start to end:
    if element == value:
        counter++

return counter
```

Visualized on `[3, 10, 1, 10, 10]`, counting `10`:

```
[ 3 | 10 | 1 | 10 | 10 ]
  ↓    ↓   ↓    ↓    ↓
  3≠10 10=10 1≠10 10=10 10=10
  skip  +1   skip  +1    +1

counter = 3  ✅
```

### `count` vs `count_if`

`count` checks for exact value equality.
`count_if` counts elements that satisfy a **condition**:

```
count(arr.begin(), arr.end(), 10)
→  counts elements equal to 10

count_if(arr.begin(), arr.end(), [](int x) { return x > 5; })
→  counts elements greater than 5
```

### Edge Cases

- Returns `0` if value not found — never returns negative, never throws.
- Works with any type that supports `operator==`.
- For `set` or `map`, the member function `st.count(x)` is O(log n) — prefer that over `std::count` which is O(n).

| Case | TC |
|------|----|
| Any | O(n) |

**Space:** O(1)

---

## 6. `find`

### What It Does

Searches for the **first occurrence** of a value and returns an **iterator** to it.
Returns `end()` if not found.

```
auto it = find(start, end, value)

it != end()  →  found   →  *it gives the value
it == end()  →  not found
```

### How It Works Internally

```
for each element from start to end:
    if element == value:
        return iterator to this element   ← stops immediately on first match

return end   ← only if never matched
```

Visualized on `[3, 7, 10, 9, 4]`, finding `10`:

```
[ 3 | 7 | 10 | 9 | 4 ]
  ↓   ↓    ↓
  3≠10 7≠10 10=10  ← FOUND, stop here

Returns iterator to index 2
*it = 10
```

Visualized finding `99` (not present):

```
[ 3 | 7 | 10 | 9 | 4 ]
  ↓   ↓    ↓   ↓   ↓
  all ≠ 99               ← never matched

Returns end()
it == arr.end()  →  not found
```

### `find` vs `lower_bound`

```
find          →  unsorted OK, O(n), checks exact equality
lower_bound   →  MUST be sorted, O(log n), finds first >= x
```

Use `find` when the container is unsorted or when you need the exact match.
Use `lower_bound` when the container is sorted and you want speed.

### `find` vs `set::find` / `map::find`

```
std::find(st.begin(), st.end(), x)   →  O(n)  ❌  linear scan
st.find(x)                           →  O(log n) ✅  tree traversal
```

Same rule as `lower_bound` — always use the member function for `set` and `map`.

### Edge Cases

- Returns iterator to **first** occurrence — if value exists multiple times, only first is returned.
- Dereferencing `end()` is **undefined behaviour** — always check `it != arr.end()`.
- `find` uses `operator==` for comparison — works on any type with equality defined.

| Case | TC |
|------|----|
| Element at start | O(1) — stops immediately |
| Element at end or not found | O(n) |
| Average | O(n) |

**Space:** O(1)

---

## 7. `reverse`

### What It Does

**Reverses the elements in-place** — modifies the original container directly.
No copy is created. No value is returned.

```
reverse(start, end)

Before:  [ 1 | 2 | 3 | 4 | 5 ]
After:   [ 5 | 4 | 3 | 2 | 1 ]   ← original modified
```

### How It Works Internally

Uses two pointers — one starting from the front, one from the back — swapping and moving inward:

```
left pointer  →  starts at begin
right pointer →  starts at end - 1

while left < right:
    swap(*left, *right)
    left++
    right--
```

Visualized on `[1, 2, 3, 4, 5]`:

```
Step 1:
  left=0, right=4
  [ 1 | 2 | 3 | 4 | 5 ]
    ↑               ↑
  swap(1, 5) → [ 5 | 2 | 3 | 4 | 1 ]

Step 2:
  left=1, right=3
  [ 5 | 2 | 3 | 4 | 1 ]
        ↑       ↑
  swap(2, 4) → [ 5 | 4 | 3 | 2 | 1 ]

Step 3:
  left=2, right=2
  left == right → stop (middle element stays)

Final: [ 5 | 4 | 3 | 2 | 1 ]
```

For even-length arrays, every element is swapped. For odd-length, the middle element stays put.

### Works on Strings Too

```
string s = "hello";
reverse(s.begin(), s.end());
// s is now "olleh"
```

### In-Place — Original Is Mutated

```
vector<int> arr = {1, 2, 3, 4, 5};
reverse(arr.begin(), arr.end());
// arr is now {5, 4, 3, 2, 1}  — original changed, no copy made
```

If you need to keep the original:

```
vector<int> copy = arr;          // make a copy first
reverse(copy.begin(), copy.end()); // reverse the copy
// arr unchanged, copy is reversed
```

### Reversing a Sub-range

```
arr = [1, 2, 3, 4, 5]
reverse(arr.begin() + 1, arr.begin() + 4)
→  reverses indices 1, 2, 3 only

Before:  [ 1 | 2 | 3 | 4 | 5 ]
After:   [ 1 | 4 | 3 | 2 | 5 ]
              └───────────┘
              only this part reversed
```

### Edge Cases

- Empty range or single element — does nothing, no crash.
- Odd-length array — middle element is untouched.
- Always verify you want in-place mutation — there's no undo.

| Case | TC |
|------|----|
| Any | O(n/2) = O(n) — n/2 swaps |

**Space:** O(1) — swaps in place, no extra array.

---

## 8. Shrinking the Search Space

Every function accepts any valid `[start, end)` range — not just `begin()` to `end()`.

```
arr = [ 1 | 2 | 3 | 4 | 5 | 6 | 7 ]
        ↑                           ↑
     begin()                      end()

// Process only indices 1 through 5 (skip first and last):
min_element(arr.begin() + 1, arr.end() - 1)

// Count only in first half:
count(arr.begin(), arr.begin() + n/2, x)

// Reverse only middle section:
reverse(arr.begin() + 2, arr.end() - 2)
```

The range just needs to be valid — `start <= end`. An empty range (start == end) is valid and safe.

---

## 9. Mutating vs Non-Mutating Functions

| Function | Mutates original? | Returns |
|----------|-------------------|---------|
| `min_element` | ❌ No | Iterator to min |
| `max_element` | ❌ No | Iterator to max |
| `accumulate` | ❌ No | Sum value |
| `count` | ❌ No | Integer count |
| `find` | ❌ No | Iterator to element |
| `reverse` | ✅ **Yes** | Nothing (void) |

`reverse` is the only mutating function here — it changes the original container.
All others are **read-only** — they scan and report without touching the data.

---

## 10. Complexity Summary

| Function | Time | Space | Notes |
|----------|------|-------|-------|
| `min_element` | O(n) | O(1) | Single scan, tracks minimum |
| `max_element` | O(n) | O(1) | Single scan, tracks maximum |
| `accumulate` | O(n) | O(1) | Single scan, running sum |
| `count` | O(n) | O(1) | Single scan, running counter |
| `find` | O(n) | O(1) | Stops early on first match |
| `reverse` | O(n) | O(1) | n/2 swaps, two-pointer |

All are **O(n) time, O(1) space** — no hidden allocations, no copies.

---

## 11. Quick Reference

```
┌────────────────┬──────────────────────────────────────────────┬────────┐
│  Function      │  Call                                        │  TC    │
├────────────────┼──────────────────────────────────────────────┼────────┤
│ min_element    │ *min_element(v.begin(), v.end())             │ O(n)   │
│ max_element    │ *max_element(v.begin(), v.end())             │ O(n)   │
│ accumulate     │ accumulate(v.begin(), v.end(), 0)            │ O(n)   │
│ accumulate     │ accumulate(v.begin(), v.end(), 0LL)          │ O(n)   │
│ count          │ count(v.begin(), v.end(), x)                 │ O(n)   │
│ find           │ find(v.begin(), v.end(), x)                  │ O(n)   │
│ reverse        │ reverse(v.begin(), v.end())                  │ O(n)   │
│ reverse copy   │ copy then reverse(copy.begin(), copy.end())  │ O(n)   │
└────────────────┴──────────────────────────────────────────────┴────────┘
```

### Key Rules to Remember

```
1. Always dereference min/max/find result with *it — they return iterators, not values.

2. Always check it != arr.end() before dereferencing find result.

3. accumulate initial value type determines return type — use 0LL for large sums.

4. reverse mutates in-place — make a copy first if you need the original.

5. For set/map: use member functions (st.find, st.count, st.lower_bound)
   NOT std:: versions — member functions are O(log n), std:: versions are O(n).
```

---

> **One-line memory aid:**
> All six functions walk `[begin, end)` once — O(n), O(1) space.  
> `min/max/find` return **iterators** (dereference to get value).  
> `accumulate/count` return **values** directly.  
> `reverse` is the only one that **modifies** the original.
