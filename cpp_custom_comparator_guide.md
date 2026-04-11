# C++ Custom Sorting by Comparator — Complete Guide

---

## Table of Contents

1. [What Is a Comparator?](#1-what-is-a-comparator)
2. [The Golden Rule — Strict Weak Ordering](#2-the-golden-rule--strict-weak-ordering)
3. [How `sort()` Uses a Comparator](#3-how-sort-uses-a-comparator)
4. [Building Intuition — Manual Swap First](#4-building-intuition--manual-swap-first)
5. [Custom Comparator for Primitives](#5-custom-comparator-for-primitives)
6. [Custom Comparator for Pairs](#6-custom-comparator-for-pairs)
7. [Passing Comparator to `sort()`](#7-passing-comparator-to-sort)
8. [Multi-level Sorting](#8-multi-level-sorting)
9. [Built-in Comparators](#9-built-in-comparators)
10. [Ways to Write a Comparator](#10-ways-to-write-a-comparator)
11. [Edge Cases & Common Bugs](#11-edge-cases--common-bugs)
12. [Complexity](#12-complexity)

---

## 1. What Is a Comparator?

By default `sort()` orders elements **ascending** — smaller values first.
A **comparator** is a function you provide to tell `sort()` a different rule.

```
sort(arr.begin(), arr.end())           →  uses default  (ascending)
sort(arr.begin(), arr.end(), comp)     →  uses your rule (whatever comp says)
```

The comparator is just a **function that takes two elements and returns a bool**:

```
bool comp(T a, T b) {
    return /* true if a should come BEFORE b */;
}
```

That's the entire contract. Everything else follows from this.

---

## 2. The Golden Rule — Strict Weak Ordering

Your comparator **must** follow one rule — called **strict weak ordering**:

```
Return true   →  a comes BEFORE b
Return false  →  a does NOT come before b (b comes before or they are equal)
```

The word **strict** means: if `a == b`, you must return **false**.

```
✅  return a < b;       correct — strict, ascending
✅  return a > b;       correct — strict, descending
✅  return a.x < b.x;  correct — sort by field x

❌  return a <= b;      WRONG — returns true when a == b → undefined behaviour
❌  return true;        WRONG — always swaps → undefined behaviour
❌  return a >= b;      WRONG — same issue
```

Breaking this rule causes **undefined behaviour** — the sort may produce garbage, infinite loops, or crash.

---

## 3. How `sort()` Uses a Comparator

Internally, `sort()` compares pairs of elements and decides their order.
Every time it needs to compare `a` and `b`, it calls `comp(a, b)`:

```
comp(a, b) == true   →  keep a before b  (no swap needed)
comp(a, b) == false  →  put b before a   (swap)
```

```
Example: sort [3, 1, 4, 2] with comp = (a < b)  [ascending]

Compare 3 and 1:  comp(3,1) = (3 < 1) = false  →  swap  →  [1, 3, 4, 2]
Compare 3 and 4:  comp(3,4) = (3 < 4) = true   →  keep  →  [1, 3, 4, 2]
Compare 4 and 2:  comp(4,2) = (4 < 2) = false  →  swap  →  [1, 3, 2, 4]
...and so on until sorted:   [1, 2, 3, 4]
```

Same logic, different comparator — descending `(a > b)`:

```
Compare 3 and 1:  comp(3,1) = (3 > 1) = true   →  keep  →  3 stays before 1
Compare 1 and 4:  comp(1,4) = (1 > 4) = false  →  swap  →  4 moves forward
...final result:  [4, 3, 2, 1]
```

---

## 4. Building Intuition — Manual Swap First

Before using `sort()` with a comparator, it helps to see the raw idea:
a function that says **"should I swap these two elements?"**

```
bool should_i_swap(int a, int b) {
    return a > b;
}
```

This is used inside a manual bubble-style sort:

```
for i in 0..n:
    for j in i+1..n:
        if should_i_swap(arr[i], arr[j]):
            swap(arr[i], arr[j])
```

The mental model:

```
arr[i] = 5,  arr[j] = 2

should_i_swap(5, 2) = (5 > 2) = true  →  swap

After swap:  arr[i] = 2,  arr[j] = 5

Result: smaller value moves to front  →  ascending order
```

```
arr[i] = 2,  arr[j] = 5

should_i_swap(2, 5) = (2 > 5) = false  →  no swap

2 stays before 5  →  ascending order preserved
```

This is exactly what `sort()` does internally — just much faster (O(n log n) vs O(n²)).
The comparator you pass to `sort()` is the **same idea** as `should_i_swap`, but with a slight rename:

```
should_i_swap(a, b) = true   →  swap a and b
comp(a, b)          = true   →  a stays BEFORE b  (opposite framing, same logic)

Relationship:
  should_i_swap(a, b)  ≡  comp(b, a)   [they're mirrors of each other]
```

---

## 5. Custom Comparator for Primitives

### Ascending (default behaviour)

```
comp(a, b):  return a < b

[5, 2, 8, 1]  →  [1, 2, 5, 8]
```

### Descending

```
comp(a, b):  return a > b

[5, 2, 8, 1]  →  [8, 5, 2, 1]
```

The only change is flipping `<` to `>`. That single flip reverses the entire sort order.

---

## 6. Custom Comparator for Pairs

When elements are `pair<int, int>`, you need to define sorting across **two fields**.

The common requirement:

```
Sort by first element ascending.
If first elements are equal → sort by second element descending.
```

### The Decision Flow

```
Given pair a = (a.first, a.second)
      pair b = (b.first, b.second)

Step 1: Are first elements different?
          │
          ├── YES → use first element to decide order
          │           return a.first < b.first   (ascending by first)
          │
          └── NO  → first elements are equal, use second element
                      return a.second > b.second  (descending by second)
```

### Visualized

```
Input pairs:  (3,9)  (1,5)  (3,2)  (1,8)

Step 1 — sort by first ascending:
  1 < 3  →  (1,_) before (3,_)
  pairs with first=1:  (1,5) and (1,8)
  pairs with first=3:  (3,9) and (3,2)

Step 2 — within each group, sort by second descending:
  first=1 group:  (1,8) before (1,5)   [8 > 5]
  first=3 group:  (3,9) before (3,2)   [9 > 2]

Final order:  (1,8)  (1,5)  (3,9)  (3,2)
```

---

## 7. Passing Comparator to `sort()`

The comparator is passed as the **third argument** to `sort()`:

```
bool comp(pair<int,int> a, pair<int,int> b) {
    if (a.first != b.first) {
        return a.first < b.first;
    }
    return a.second > b.second;
}

sort(arr.begin(), arr.end(), comp);
                              ↑
                       function name only — no () here
```

Notice: you pass the **function name**, not a call. `comp` not `comp()`.

```
sort(arr.begin(), arr.end(), comp);    ✅  correct
sort(arr.begin(), arr.end(), comp());  ❌  wrong — calls comp() which returns a bool, not a function
```

---

## 8. Multi-level Sorting

The pair comparator pattern generalizes to any number of fields.
The rule is always the same: **check fields one by one; the first field that differs decides the order**.

```
Level 1:  primary sort key
Level 2:  tiebreaker (used only when level 1 is equal)
Level 3:  second tiebreaker (used only when levels 1 and 2 are equal)
...
```

```
bool comp(pair<int,int> a, pair<int,int> b) {

    // Level 1: first element, ascending
    if (a.first != b.first)
        return a.first < b.first;

    // Level 2: second element, descending (tiebreaker)
    return a.second > b.second;
}
```

For three fields (struct or tuple):

```
bool comp(T a, T b) {

    // Level 1
    if (a.x != b.x) return a.x < b.x;

    // Level 2
    if (a.y != b.y) return a.y > b.y;

    // Level 3
    return a.z < b.z;
}
```

---

## 9. Built-in Comparators

STL ships with ready-made comparator objects — no need to write your own for simple cases.

### `greater<T>()` — Descending Order

```
sort(arr.begin(), arr.end(), greater<int>());
→  sorts ints descending

sort(arr.begin(), arr.end(), greater<pair<int,int>>());
→  sorts pairs descending
```

### How `greater<pair<int,int>>()` Compares Pairs

`greater<T>` simply flips the default `<` comparison to `>`.
For pairs, the default `<` is lexicographic (first element first, second breaks ties).
So `greater<pair<int,int>>` is also lexicographic but **reversed**:

```
Default less<pair>:     (1,5) < (2,3) < (3,9)   →  ascending by first, then second
greater<pair>:          (3,9) > (2,3) > (1,5)   →  descending by first, then second
```

```
Input:  (1,5)  (3,9)  (2,3)  (3,2)

sort with greater<pair<int,int>>():

First sort by first element descending:
  (3,_) before (2,_) before (1,_)

Within first=3 group, sort by second descending:
  (3,9) before (3,2)

Output:  (3,9)  (3,2)  (2,3)  (1,5)
```

### `less<T>()` — Ascending Order (default, rarely written explicitly)

```
sort(arr.begin(), arr.end(), less<int>());
→  same as sort(arr.begin(), arr.end())
```

---

## 10. Ways to Write a Comparator

There are three ways to pass a comparator to `sort()`. All are equivalent.

### Option 1 — Named Function

```
bool comp(pair<int,int> a, pair<int,int> b) {
    if (a.first != b.first)
        return a.first < b.first;
    return a.second > b.second;
}

sort(arr.begin(), arr.end(), comp);
```

Best for: complex logic you want to name and reuse.

---

### Option 2 — Lambda (inline, no separate function)

```
sort(arr.begin(), arr.end(), [](pair<int,int> a, pair<int,int> b) {
    if (a.first != b.first)
        return a.first < b.first;
    return a.second > b.second;
});
```

Best for: one-off comparators used in a single place.

---

### Option 3 — Built-in (`greater`, `less`)

```
sort(arr.begin(), arr.end(), greater<int>());
sort(arr.begin(), arr.end(), greater<pair<int,int>>());
```

Best for: simple ascending/descending without custom logic.

---

### Comparison

```
┌──────────────────┬─────────────────┬────────────────────────────┐
│  Style           │  Where defined  │  Best for                  │
├──────────────────┼─────────────────┼────────────────────────────┤
│  Named function  │  Outside main   │  Complex/reusable logic    │
│  Lambda          │  Inside sort()  │  Simple, one-time use      │
│  Built-in        │  STL            │  Ascending/descending only │
└──────────────────┴─────────────────┴────────────────────────────┘
```

---

## 11. Edge Cases & Common Bugs

---

### Bug 1 — Using `<=` instead of `<`

```
❌  return a <= b;    // returns true when a == b → UB
✅  return a < b;
```

When `a == b`, `a <= b` is true — meaning `sort` thinks `a` should come before `b`.
But it would also think `b` should come before `a`. Contradiction → undefined behaviour.

---

### Bug 2 — Passing `comp()` instead of `comp`

```
❌  sort(arr.begin(), arr.end(), comp());   // calls comp with no args → compile error
✅  sort(arr.begin(), arr.end(), comp);     // passes the function itself
```

---

### Bug 3 — Wrong field used in tiebreaker

```
// Intention: ascending first, descending second
❌
if (a.first != b.first)
    return a.first < b.first;
return a.second < b.second;   // wrong: this is ascending second, not descending

✅
if (a.first != b.first)
    return a.first < b.first;
return a.second > b.second;   // correct: descending second
```

---

### Bug 4 — Missing the inequality check

```
❌
bool comp(pair<int,int> a, pair<int,int> b) {
    if (a.first > b.first) return true;   // handles a.first > b.first
    return a.second < b.second;           // runs even when a.first < b.first → wrong
}

✅
bool comp(pair<int,int> a, pair<int,int> b) {
    if (a.first != b.first)               // gates the tiebreaker correctly
        return a.first < b.first;
    return a.second > b.second;
}
```

Without the `!=` guard, when `a.first < b.first` the tiebreaker still runs and may contradict the primary sort.

---

### Bug 5 — `should_i_swap` vs `comp` framing confusion

```
should_i_swap(a, b) = true  →  swap a and b (b comes first)
comp(a, b)          = true  →  a stays before b (no swap)

They are MIRRORS:
  should_i_swap(a, b)  ≡  comp(b, a)

Don't mix the two framings — pick one and stay consistent.
```

---

## 12. Complexity

Custom comparators do not change the asymptotic complexity of `sort()`.

| What | TC | SC |
|------|----|----|
| `sort()` with comparator | O(n log n) | O(log n) |
| Each comparator call | O(1) for primitives/pairs | O(1) |
| Each comparator call (strings) | O(L) — string comparison | O(1) |

Total comparisons made by `sort()`: O(n log n).
Each comparison calls your `comp` once.
So if `comp` is O(1), total time is O(n log n).
If `comp` is O(L) (e.g., comparing strings of length L), total time is O(n · L · log n).

---

> **One-line memory aid:**
> `comp(a, b) = true` means **a comes before b** — return true when a should be on the left. Flip `<` to `>` for one field to reverse that field's order. Never return true when a equals b.
