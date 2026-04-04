# Monk and the Candy Bags — Problem Explanation

---

## The Problem (Plain English)

You have **N bags**, each with some candies.  
Every minute you can pick **one bag**, eat all its candies, and the bag **refills to ⌊X/2⌋** candies (floor of half).  
You have **K minutes** total. Goal: **maximize total candies eaten**.

---

## Key Observations

### 1. Always pick the largest bag

Every minute you get exactly **one pick** — so greedily, you should always eat from the bag with the **most candies** that minute.  
There is no benefit in saving a big bag for later, because every pick you make only shrinks bags (X → ⌊X/2⌋), never grows them.

### 2. After eating, the bag goes back in the pool

The bag doesn't disappear. It refills to ⌊X/2⌋ and is available again next minute.  
So the same bag can be picked multiple times — it just halves each time.

### 3. Example walkthrough

```
N=5, K=3
Bags: [2, 1, 7, 4, 2]
```

| Minute | Bags (sorted) | Pick | Candies Eaten | Bag Refills To |
|--------|--------------|------|---------------|----------------|
| 1 | [1, 2, 2, 4, **7**] | 7 | 7 | ⌊7/2⌋ = 3 |
| 2 | [1, 2, 2, **3**, 4] | 4 | 4 | ⌊4/2⌋ = 2 |
| 3 | [1, 2, 2, 2, **3**] | 3 | 3 | — |

**Total = 7 + 4 + 3 = 14** ✅

---

## Why `multiset` Is The Perfect Fit Here

This problem needs a data structure that does **three things repeatedly** across K iterations:

| Need | Operation | Why |
|------|-----------|-----|
| Find the largest bag | `--bags.end()` → last element | `multiset` is always sorted |
| Remove only that one bag | `erase(iterator)` | Removes exactly ONE element |
| Insert the refilled bag (X/2) | `insert(new_val)` | Auto-placed in sorted order |

`multiset` handles all three natively and efficiently.

---

## Why Not Other Containers?

### ❌ `set`
Doesn't allow duplicates. Bags can have equal candy counts (e.g., two bags with 2 candies).  
Inserting a duplicate into `set` silently ignores it — **you'd lose bags**, giving wrong answers.

### ❌ `vector` + `sort`
You could sort a vector and always pick the last element, but after eating and refilling you'd need to re-sort or re-insert in the right position — **O(n) per insert** instead of O(log n).  
For K = 10⁵ and N = 10⁵ that's 10¹⁰ operations — too slow.

### ❌ `priority_queue` (max-heap)
Seems like a natural fit — but `priority_queue` does not support `erase(iterator)`.  
You can only pop the top. After popping X and pushing X/2, the heap rebalances — that's fine.  
Actually a max-heap works here too, but `multiset` makes the intent cleaner with direct iterator access,  
and critically, `erase(iterator)` on multiset removes **exactly one** occurrence — the same guarantee a heap pop gives.

### ✅ `multiset` — why it wins
- **Sorted at all times** — largest element always at `--bags.end()`.
- **Allows duplicates** — multiple bags with the same count are all stored.
- **`erase(iterator)` removes exactly one** — not all bags with that count.
- **`insert` places the new value in the right position automatically** — no manual sorting needed.

---

## The Critical `erase` Distinction

```cpp
bags.erase(it);          // ✅ removes only the ONE bag we just ate
bags.erase(candies);     // ❌ would remove ALL bags with that candy count
```

This is the most important reason `multiset` is used with an **iterator**, not a value.

If bags = `{2, 2, 7}` and you eat the 7:
- `erase(it)` → bags becomes `{2, 2, 3}` ✅
- `erase(7)` → bags becomes `{2, 2, 3}` ✅ (fine here, only one 7)

But if bags = `{4, 4, 4}` and you eat one 4:
- `erase(it)` → bags becomes `{2, 4, 4}` ✅ (only removed one)
- `erase(4)` → bags becomes `{2}` ❌ (removed all three 4s — wrong!)

---

## Solution Walkthrough

```cpp
multiset<long long> bags;

// Insert all bags → O(n log n)
for (int i = 0; i < n; i++) {
    long long candies;
    cin >> candies;
    bags.insert(candies);
}

long long total_candies = 0;

// K greedy picks → O(k log n)
for (int i = 0; i < k; i++) {
    auto it = (--bags.end());      // iterator to the largest bag  → O(1)
    long long candies = *it;

    total_candies += candies;
    bags.erase(it);                // remove exactly this bag      → O(log n)

    bags.insert(candies / 2);      // refill and re-insert         → O(log n)
}

cout << total_candies << endl;
```

### Why `--bags.end()` and not `bags.end()`?

`bags.end()` is a **past-the-end sentinel** — it points to nothing.  
`--bags.end()` steps back one position to the **last actual element** — the largest value.  
`prev(bags.end())` is equivalent and slightly more readable.

---

## Complexity Analysis

### Time Complexity

| Phase | Complexity | Reason |
|-------|-----------|--------|
| Reading & inserting N bags | O(N log N) | Each `insert` into multiset is O(log n) |
| K greedy iterations | O(K log N) | Each iteration: O(1) find + O(log n) erase + O(log n) insert |
| **Total** | **O((N + K) log N)** | |

For N = K = 10⁵ → ~10⁵ × 17 ≈ **1.7 million operations** — well within limits.

### Space Complexity

| What | Space |
|------|-------|
| multiset storage | O(N) — always holds exactly N bags |
| Per iteration overhead | O(1) — no extra allocation |
| **Total** | **O(N)** |

The multiset size never changes — every iteration removes one element and inserts one back.

---

## Edge Cases

| Case | What Happens |
|------|-------------|
| K > N and bags reduce to 0 | Bag with 0 candies stays in the set, ⌊0/2⌋ = 0, contributes 0 to total — handled correctly |
| All bags have same value | `erase(iterator)` safely removes only one — duplicates preserved |
| K = 0 | Loop doesn't execute, total = 0 — correct |
| N = 1 | Same bag is picked repeatedly, halving each time |
| Ai = 0 | All picks yield 0 candies — total = 0 |
| Ai up to 10¹⁰ | `long long` required — `int` overflows (max int ≈ 2.1 × 10⁹) |

---

## Summary

> Use `multiset` when you need to **repeatedly find, remove, and reinsert the maximum (or minimum) element**, especially when **duplicates must be preserved** and **only one occurrence should be removed at a time**.

This problem is a textbook greedy + multiset pattern:  
**sorted order + duplicate support + single-element erase by iterator** = `multiset`.
