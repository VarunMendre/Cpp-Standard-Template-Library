# Nesting in STL Maps and Sets

---

## 1. What Is Nesting?

A `map` or `set` is just a template — it doesn't care what type you give it,
as long as that type can be **compared** with `operator<`.

So instead of:

```
map<int, int>          key = int,  value = int
```

You can do:

```
map<pair<int,int>, int>            key = pair
map<int, set<string>>              value = set
map<int, vector<int>>              value = vector
map<set<int>, int>                 key = set
map<pair<string,string>, vector<int>>   both sides nested
```

This is nesting — **using one container inside another**.

---

## 2. How Comparison Works for Nested Types

The outer `map`/`set` must order its keys.
When the key itself is a compound type, STL compares them **lexicographically** — element by element, left to right.

---

### Pair Comparison

```
Comparing  {2, 2}  vs  {2, 3}
            │  │         │  │
            1st 2nd      1st 2nd

Step 1 → compare 1st elements:  2 == 2  (tie → move to next)
Step 2 → compare 2nd elements:  2 < 3   ✅

Result: {2, 2} < {2, 3}   →   true
```

```
Comparing  {1, 2}  vs  {2, 3}

Step 1 → compare 1st elements:  1 < 2   ✅  (decided here, stop)

Result: {1, 2} < {2, 3}   →   true
```

> Think of it like sorting words in a dictionary — letter by letter,
> only moving to the next letter when the current ones are equal.

---

### Set Comparison

A `set` **always stores its elements sorted**, regardless of insertion order.

```
set<int> st1 = {4, 2, 3}   →   stored as  {2, 3, 4}
set<int> st2 = {4, 5, 6}   →   stored as  {4, 5, 6}

Compare element by element (in sorted order):

Position:   0    1    2
   st1:     2    3    4
   st2:     4    5    6
            │
            2 < 4  ✅  (decided at position 0)

Result: st1 < st2   →   true
```

**Critical point:**

```
set<int> a = {3, 1, 2}   →   stored as {1, 2, 3}
set<int> b = {1, 2, 3}   →   stored as {1, 2, 3}

a == b   →   TRUE  (same key in a map!)
```

> Insertion order doesn't matter for sets — only the final sorted contents do.

---

## 3. All Nesting Types — What They Look Like

---

### 3.1 — `pair` as Key

```
┌─────────────────────────────────────────────────┐
│         map<pair<int,int>, int>                 │
│                                                 │
│   Key (pair)     │   Value                      │
│  ─────────────── │ ─────────                    │
│   { 1 , 2 }      │    10                        │
│   { 1 , 5 }      │    30                        │
│   { 2 , 3 }      │    20                        │
│                                                 │
│   Sorted by: 1st element, then 2nd on tie       │
└─────────────────────────────────────────────────┘
```

**Use when:** identity needs two components — grid cell (row, col), person (fname, lname).  
**TC:** O(log n) per operation. Comparison cost = O(1) for int pairs, O(L) for string pairs.

---

### 3.2 — `set` as Key

```
┌─────────────────────────────────────────────────┐
│         map<set<int>, int>                      │
│                                                 │
│   Key (set)       │   Value                     │
│  ──────────────── │ ─────────                   │
│   { 1, 2, 3 }     │    5                        │
│   { 4, 5, 6 }     │    8                        │
│                                                 │
│  {3,1,2} and {1,2,3} → SAME KEY (both = {1,2,3})│
└─────────────────────────────────────────────────┘
```

**Use when:** a group of values is the identity, and order of insertion doesn't matter.  
**TC:** O(k · log n) per operation — each key comparison costs O(k), where k = set size.

---

### 3.3 — `set` as Value

```
┌─────────────────────────────────────────────────┐
│         map<int, set<string>>                   │
│                                                 │
│   Key (marks)  │   Value (student names)        │
│  ──────────── │ ──────────────────────────      │
│      78        │   { "Alice", "Eve" }           │
│      99        │   { "Bob" }                    │
│                                                 │
│   Inner set is always sorted alphabetically     │
│   Duplicates automatically removed              │
└─────────────────────────────────────────────────┘
```

**Use when:** one key owns multiple unique values that need to stay sorted.  
**TC:** O(log n) map + O(log k) set insert.

---

### 3.4 — `vector` as Value

```
┌─────────────────────────────────────────────────┐
│    map<pair<string,string>, vector<int>>        │
│                                                 │
│   Key (name pair)       │  Value (scores)       │
│  ───────────────────── │ ──────────────         │
│   {"Alice", "Smith"}    │  [85, 90, 78]         │
│   {"Bob",   "Jones"}    │  [60, 72]             │
│                                                 │
│   Preserves insertion order. Allows duplicates. │
└─────────────────────────────────────────────────┘
```

**Use when:** order matters, or duplicates must be kept.  
**TC:** O(log n) map + O(1) amortized push_back.

---

### 3.5 — `map` as Value

```
┌──────────────────────────────────────────────────────┐
│         map<string, map<string, int>>                │
│                                                      │
│   Outer key    │   Inner map                         │
│  ─────────────│──────────────────────────────        │
│   "Science"    │   { "Alice"→95, "Bob"→88 }          │
│   "Math"       │   { "Alice"→70, "Eve"→91 }          │
│                                                      │
│   Two-level grouping: dept → student → marks         │
└──────────────────────────────────────────────────────┘
```

**Use when:** you need two-level grouping.  
**TC:** O(log n) outer + O(log m) inner per operation.

---

### Quick Reference Table

```
┌──────────────────────────────────────────────────────────────────┐
│  Pattern                         │ Sorted? │ Dups in value?      │
│──────────────────────────────────┼─────────┼─────────────────────│
│  map<pair<A,B>, V>               │  Yes    │  Depends on V       │
│  map<set<T>, V>                  │  Yes    │  Depends on V       │
│  map<K, set<V>>                  │  Yes    │  No (deduplicates)  │
│  map<K, vector<V>>               │  No     │  Yes                │
│  map<K, map<K2,V>>               │  Yes    │  No (inner map)     │
└──────────────────────────────────────────────────────────────────┘
```

---

## 4. The `operator[]` Default-Construction Rule

When you write `mpp[key].push_back(x)` or `mpp[key].insert(x)` and `key` doesn't exist yet:

```
mpp[key]  →  key not found?
              │
              ▼
         Creates new entry automatically
         Value is default-constructed:
           vector → []         (empty vector)
           set    → {}         (empty set)
           int    → 0
              │
              ▼
         .push_back(x)  or  .insert(x)  called on it
```

You never need to manually initialize the inner container first. The map handles it.

**The trap — accidental insertion:**

```
// Just checking if a key exists?
if (mpp[key].size() > 0)   ❌  creates an empty entry for 'key' if missing!

// Safe way to check:
if (mpp.find(key) != mpp.end())   ✅  no insertion
```

---

## 5. Comparison Cost Grows With Nesting

Every map/set operation walks the tree and compares keys at each node.
When the key is compound, each comparison costs more:

```
Key Type              Comparison cost
─────────────────────────────────────
int                   O(1)
string (length L)     O(L)
pair<int,int>         O(1)
pair<string,string>   O(L)
set<int> (size k)     O(k)
vector<int> (size k)  O(k)

So:
  map<int, V>           →  O(log n) per op
  map<set<int>, V>      →  O(k · log n) per op   ← don't forget this!
```

---

## 6. Problem — Sort Students by Marks

### Statement

Given N students with names and marks, print them sorted:
- **Decreasing order of marks** (highest first)
- **Lexicographical order** for students with the same marks

```
Input:
3
Eve 78
Bob 99
Alice 78

Output:
Bob 99
Alice 78
Eve 78
```

---

### Solution

```cpp
#include <bits/stdc++.h>
using namespace std;

int main()
{
    map<int, set<string>> marks_map;

    int n;
    cin >> n;

    for (int i = 0; i < n; i++)
    {
        int marks;
        string name;
        cin >> name >> marks;

        marks_map[-1 * marks].insert(name);
    }

    auto current_it = --marks_map.end();

    for(auto & marks_students_pair : marks_map)
    {
        auto &students = marks_students_pair.second;
        auto marks = marks_students_pair.first;

        for(auto & student : students) {
            cout << student << " " << -1 * marks << endl;
        }
    }
    return 0;
}
```

---

### Why `map<int, set<string>>`?

The problem needs two levels of sorting at the same time:

```
Level 1 (outer):  sort by marks    → descending
Level 2 (inner):  sort by name     → alphabetical (for same marks)
```

`map<int, set<string>>` handles both in one structure:

```
┌──────────────────────────────────────────────────────────┐
│          map<int, set<string>>                           │
│                                                          │
│   map key (marks)  →  set value (names)                  │
│   ─────────────── ──  ────────────────                   │
│       -99          →  { "Bob" }                          │
│       -78          →  { "Alice", "Eve" }                 │
│                              ↑                           │
│                    set keeps names alphabetical          │
│                    automatically on every insert         │
│                                                          │
│   map iterates keys ascending (-99 → -78)                │
│   = marks descending (99 → 78) ✅                        │
└──────────────────────────────────────────────────────────┘
```

No manual sorting anywhere — the structure does it on every insert.

---

### The Negative Key Trick

`map` always iterates in **ascending** key order.
But we need marks printed **descending**.

```
Normal:    78  <  99    →  map prints 78 first  ❌

Negate:   -99  <  -78   →  map prints -99 first
           ↓                = mark 99 first       ✅
        multiply by -1 when printing to restore original
```

---

### Why `set<string>` as Value, Not `vector<string>`?

```
vector<string>:                    set<string>:
  insert "Eve"   → ["Eve"]           insert "Eve"   → {"Eve"}
  insert "Alice" → ["Eve","Alice"]   insert "Alice" → {"Alice","Eve"}  ← sorted automatically
  ↓                                  ↓
  need to sort after all inserts     always sorted, no extra work
  O(k log k) extra                   O(log k) per insert, done
```

---

### Dry Run

```
Input: Eve 78 / Bob 99 / Alice 78

Step 1 — Insert Eve 78:
  marks_map[-78].insert("Eve")
  map: { -78 → {"Eve"} }

Step 2 — Insert Bob 99:
  marks_map[-99].insert("Bob")
  map: { -99 → {"Bob"}, -78 → {"Eve"} }

Step 3 — Insert Alice 78:
  marks_map[-78].insert("Alice")
  map: { -99 → {"Bob"}, -78 → {"Alice","Eve"} }
                                  ↑
                          set auto-sorted Alice before Eve

Step 4 — Iterate (ascending key order):

  key=-99 → students={"Bob"}         → print: Bob 99
  key=-78 → students={"Alice","Eve"} → print: Alice 78
                                              Eve 78
```

---

### Complexity

```
N insertions:
  map insert  → O(log N)
  set insert  → O(log N)
  per student → O(log N)
  total       → O(N log N)

Printing:
  each student printed once → O(N)

Total Time:  O(N log N)
Total Space: O(N)

Note: marks are 1–100 (constraint) → outer map has at most 100 keys
      regardless of N → map itself is O(1) space, bounded.
```
