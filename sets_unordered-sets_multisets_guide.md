# C++ STL Sets — Deep Dive Guide

> **Covers every operation** of `set`, `unordered_set`, and `multiset` —  
> with edge cases, Time Complexity, and Space Complexity. No fluff.

---

## Table of Contents

1. [`set<T>`](#1-sett)
2. [`unordered_set<T>`](#2-unordered_sett)
3. [`multiset<T>`](#3-multisett)
4. [Side-by-Side Comparison](#4-side-by-side-comparison)
5. [When to Use What](#5-when-to-use-what)

---

## 1. `set<T>`

### Internal Structure

Backed by a **Red-Black Tree** — a self-balancing Binary Search Tree.  
Every insert/delete automatically rebalances the tree, keeping height at **O(log n)** always.  
Elements are stored in **sorted order** and **duplicates are not allowed**.

---

### Operations

---

#### `insert(value)`

Inserts a value into the set.

- If the value **already exists**, the insertion is **silently ignored** — no error, no exception.
- The tree rebalances after insertion to maintain the Red-Black property.

| Case | Time Complexity |
|---|---|
| Normal insert | O(log n) |
| Duplicate insert (ignored) | O(log n) — still searches the tree |
| Insert into empty set | O(1) |

**Space:** O(1) per insertion — one new tree node allocated.

**Edge Cases:**
- Inserting a duplicate does nothing but still costs O(log n) because the tree is traversed to confirm the duplicate exists.
- Inserting into an empty set is O(1) — no traversal needed, root is set directly.
- The return value of `insert` is a `pair<iterator, bool>` — the bool is `false` if the element already existed. This is commonly ignored but useful for checking if insertion actually happened.

---

#### `find(value)`

Searches for a value and returns an iterator to it, or `st.end()` if not found.

- Always traverses the tree from root to leaf — at most O(log n) comparisons.
- `end()` is a sentinel — dereferencing it is **undefined behaviour**.

| Case | Time Complexity |
|---|---|
| Element exists | O(log n) |
| Element does not exist | O(log n) — full traversal to a leaf |

**Space:** O(1) — no extra allocation.

**Edge Cases:**
- Searching in an **empty set** returns `end()` immediately — O(1) in practice.
- Case sensitivity matters for strings — `"abc"` and `"ABC"` are treated as different keys.
- Never dereference the iterator without checking `it != st.end()` first.

---

#### `erase(value)` and `erase(iterator)`

Two forms of deletion:

**`erase(value)`** — finds the element by value and removes it.  
**`erase(iterator)`** — removes the element at a specific iterator position.

| Form | Time Complexity |
|---|---|
| `erase(value)` | O(log n) — search + delete |
| `erase(iterator)` | O(log n) — rebalancing cost |
| Erasing non-existent value | O(log n) — traverses fully, then does nothing |

**Space:** O(1) — one node freed.

**Edge Cases:**
- `erase(value)` on a **non-existent value** does nothing and does not throw — silent no-op, but still costs O(log n).
- `erase(iterator)` with an **invalid or `end()` iterator** is **undefined behaviour** — always validate the iterator first.
- After `erase`, all other iterators remain valid — Red-Black Tree erasure does not invalidate other iterators.

---

#### `count(value)`

Returns the number of times a value exists in the set — always **0 or 1** for `set`.

| Case | Time Complexity |
|---|---|
| Element exists | O(log n) |
| Element does not exist | O(log n) |

**Space:** O(1)

**Edge Cases:**
- Since `set` doesn't allow duplicates, `count` only tells you *whether* the element exists — it never returns more than 1.
- Prefer `find` over `count` when you intend to use the element after checking — `find` gives both existence and position in one call.

---

#### `lower_bound(value)` and `upper_bound(value)`

These are **ordered-set exclusive** operations — not available in `unordered_set`.

- `lower_bound(value)` → iterator to the **first element ≥ value**
- `upper_bound(value)` → iterator to the **first element > value**

| Operation | Time Complexity |
|---|---|
| `lower_bound` | O(log n) |
| `upper_bound` | O(log n) |

**Space:** O(1)

**Edge Cases:**
- If the value is **greater than all elements**, both return `end()`.
- If the value is **less than all elements**, `lower_bound` returns `begin()` — pointing to the smallest element.
- Using `std::lower_bound` (the algorithm header version) on a `set` iterator gives **O(n)** — it doesn't know about the tree structure. Always use the **member function** `st.lower_bound()` which is O(log n).

---

#### Iteration (range-for / iterator)

Iterating through a `set` always visits elements in **ascending sorted order**.

| Operation | Time Complexity |
|---|---|
| Full traversal | O(n) |
| `begin()` / `end()` | O(1) |

**Space:** O(1) — an iterator is just a pointer to a tree node.

**Edge Cases:**
- Modifying the set **while iterating** (inserting/erasing) invalidates iterators and causes **undefined behaviour** — collect elements to erase in a separate container, then erase after the loop.
- Reverse iteration using `rbegin()` / `rend()` gives descending order — also O(n).

---

#### `size()` and `empty()`

| Operation | Time Complexity |
|---|---|
| `size()` | O(1) |
| `empty()` | O(1) |

**Space:** O(1)

Size is maintained as a separate counter — not computed by tree traversal.

---

### Space Complexity Summary for `set`

| What | Space |
|---|---|
| Entire set of n elements | O(n) |
| Per-node overhead | ~3 pointers (left, right, parent) + color bit |
| Single operation overhead | O(1) |

---

---

## 2. `unordered_set<T>`

### Internal Structure

Backed by a **Hash Table** with chaining — each bucket holds a linked list of elements.  
A hash function maps each element to a bucket index.  
**No ordering** — iteration order is arbitrary and can change after rehash.  
**Duplicates not allowed.**

Average case is O(1) for most operations because a good hash function distributes elements uniformly.  
Worst case is O(n) when many elements collide into the same bucket.

---

### Operations

---

#### `insert(value)`

Hashes the value, finds the bucket, checks for duplicates in that bucket's chain, and inserts if not present.

| Case | Time Complexity |
|---|---|
| Average case | O(1) |
| Worst case (all elements in one bucket) | O(n) |
| Duplicate insert (ignored) | O(1) avg — hash computed, bucket checked, no insert |
| Insert triggers rehash | O(n) — all elements re-hashed into new table |

**Space:** O(1) amortized — occasional O(n) spike for rehash.

**Edge Cases:**
- When the **load factor** (elements / buckets) exceeds a threshold (~1.0), the table **rehashes** — doubles bucket count and reinserts all elements. This is O(n) but amortized O(1) per insertion.
- Rehashing **invalidates all iterators** — do not hold iterators across insertions.
- Duplicate inserts are silently ignored — same behaviour as `set`.
- Custom types require a custom hash function and `operator==` — otherwise it won't compile.

---

#### `find(value)`

Hashes the value, jumps directly to the bucket, scans the chain for the element.

| Case | Time Complexity |
|---|---|
| Average case | O(1) |
| Worst case (hash collision) | O(n) |

**Space:** O(1)

**Edge Cases:**
- Returns `end()` if not found — same rule as `set`: never dereference without checking.
- For **adversarial inputs** (common in competitive programming), a bad hash can force all elements into one bucket — worst case O(n). Use a custom hash with randomization for critical scenarios.
- Case sensitivity applies to strings — `"ABC"` and `"abc"` hash differently and are treated as different elements.

---

#### `erase(value)` and `erase(iterator)`

| Form | Time Complexity |
|---|---|
| `erase(value)` | O(1) avg, O(n) worst |
| `erase(iterator)` | O(1) avg, O(n) worst |
| Erasing non-existent value | O(1) avg — silent no-op |

**Space:** O(1)

**Edge Cases:**
- `erase(iterator)` with an invalid iterator is **undefined behaviour**.
- Erasing does not trigger a shrink/rehash — bucket count stays the same.
- Erasing a non-existent value is safe and silent — no exception thrown.

---

#### `count(value)`

Returns 0 or 1 — since duplicates aren't allowed.

| Case | Time Complexity |
|---|---|
| Average | O(1) |
| Worst | O(n) |

**Space:** O(1)

**Edge Cases:** Same trade-off as in `set` — prefer `find` if you need the iterator too, since `count` only confirms existence.

---

#### No `lower_bound` / `upper_bound`

`unordered_set` does **not** support these operations — there is no ordering.  
Using `std::lower_bound` from `<algorithm>` on an `unordered_set` compiles but gives **O(n)** — it degrades to a linear scan with no benefit.

---

#### Iteration

| Operation | Time Complexity |
|---|---|
| Full traversal | O(n) |
| `begin()` / `end()` | O(1) |

**Space:** O(1)

**Edge Cases:**
- Iteration order is **not sorted** and **not consistent** across runs or after rehashing.
- Do not insert or erase during iteration — a rehash can invalidate all iterators mid-loop.

---

#### `reserve(n)` and `max_load_factor(f)`

- `reserve(n)` — pre-allocates enough buckets to hold at least n elements without rehashing.
- `max_load_factor(f)` — controls the load factor threshold that triggers a rehash (default 1.0).

| Operation | Time Complexity |
|---|---|
| `reserve(n)` | O(n) — one-time rehash upfront |

**Edge Cases:**
- Forgetting to `reserve` on large inputs causes multiple mid-insertion rehashes — each O(n). Calling `reserve` once upfront avoids this entirely.
- Setting `max_load_factor` very low wastes memory; setting it very high increases collision probability.

---

### Space Complexity Summary for `unordered_set`

| What | Space |
|---|---|
| Entire set of n elements | O(n) |
| Bucket array | O(bucket_count) — can be larger than n |
| Per-element overhead | Node + pointer in chain |
| After rehash | O(n) new bucket array allocated |

---

---

## 3. `multiset<T>`

### Internal Structure

Backed by a **Red-Black Tree** — same as `set`.  
**Key difference:** duplicates are stored as **separate nodes** in the tree.  
Elements are kept in **sorted order**, and all standard BST operations apply.

---

### Operations

---

#### `insert(value)`

Inserts the value — duplicates are fully accepted and stored as new nodes.

| Case | Time Complexity |
|---|---|
| Normal insert | O(log n) |
| Insert duplicate | O(log n) — no different from any other insert |

**Space:** O(1) per insert.

**Edge Cases:**
- Every duplicate increases `size()` — memory grows linearly with each duplicate insertion.
- The return type of `insert` is just an `iterator` (not `pair<iterator, bool>` like `set`) — insertion always succeeds so there's no need for a success flag.

---

#### `find(value)`

Searches for the value and returns an iterator to the **first occurrence** — not all occurrences.

| Case | Time Complexity |
|---|---|
| Element exists (including duplicates) | O(log n) |
| Element does not exist | O(log n) |

**Space:** O(1)

**Edge Cases:**
- If 3 copies of `"ABC"` exist, `find("ABC")` points to only the **first one** in sorted order — the other two are not directly returned.
- To iterate over all duplicates, use `equal_range` — it returns the full range of matching elements.
- Returns `end()` if not found — always validate before dereferencing.

---

#### `erase(iterator)` vs `erase(value)` — The Most Critical Distinction

This is the **most important edge case** in the entire multiset.

**`erase(iterator)`** — deletes **exactly one** element at that iterator position.  
**`erase(value)`** — deletes **ALL** elements with that value.

| Form | What Is Deleted | Time Complexity |
|---|---|---|
| `erase(iterator)` | One specific element | O(log n) |
| `erase(value)` | All copies of that value | O(log n + k), where k = count of duplicates |

**Space:** O(1) per deletion.

**Edge Cases:**
- Using `erase(value)` when you only wanted to remove one copy is a **very common bug** — it silently deletes all matching elements with no warning.
- The safe pattern to erase a single duplicate: call `find` to get an iterator → verify it's not `end()` → call `erase(iterator)`.
- `erase(iterator)` with an invalid or `end()` iterator is **undefined behaviour**.
- After erasing all copies of a value, `count(value)` returns 0 and `find` returns `end()`.

---

#### `count(value)`

Returns the **actual count** of how many times a value appears — can be greater than 1.

| Case | Time Complexity |
|---|---|
| Element not present | O(log n) |
| k duplicates present | O(log n + k) |

**Space:** O(1)

**Edge Cases:**
- This is where `multiset` diverges sharply from `set` — `count` is genuinely useful here for knowing the frequency of an element.
- For very large k (e.g., all elements are the same), `count` approaches O(n).

---

#### `equal_range(value)`

Returns a `pair<iterator, iterator>` representing the range `[first, last)` covering **all elements equal to value**.  
Equivalent to calling `{lower_bound(value), upper_bound(value)}` in a single O(log n) call.

| Case | Time Complexity |
|---|---|
| Any case | O(log n) |

**Space:** O(1)

**Edge Cases:**
- If the value doesn't exist, both iterators in the pair point to the same position — the range is empty (first == last).
- This is the **correct and efficient way** to iterate over all duplicates of a value, instead of manually advancing an iterator and checking each step.

---

#### `lower_bound(value)` and `upper_bound(value)`

Same semantics as in `set`:
- `lower_bound(value)` → first element **≥ value**
- `upper_bound(value)` → first element **> value**

The range `[lower_bound(v), upper_bound(v))` spans all duplicates of value `v`.

| Operation | Time Complexity |
|---|---|
| `lower_bound` | O(log n) |
| `upper_bound` | O(log n) |

**Space:** O(1)

**Edge Cases:**
- Same warning as `set` — use the **member function** version, not `std::lower_bound` from `<algorithm>` (which gives O(n) on tree iterators).

---

#### Iteration

| Operation | Time Complexity |
|---|---|
| Full traversal | O(n) |
| `begin()` / `end()` | O(1) |

Always visits elements in **sorted order**, with duplicates appearing consecutively.

**Edge Cases:**
- Modifying (inserting/erasing) while iterating is **undefined behaviour**.
- To safely erase during iteration, use the return value of `erase(iterator)` — it returns the next valid iterator, allowing you to continue the loop safely.

---

#### `size()` and `empty()`

| Operation | Time Complexity |
|---|---|
| `size()` | O(1) |
| `empty()` | O(1) |

`size()` counts **every duplicate** as a separate element — it reflects the true number of nodes in the tree.

---

### Space Complexity Summary for `multiset`

| What | Space |
|---|---|
| Entire multiset of n elements (with duplicates) | O(n) — every duplicate is a full tree node |
| Per-node overhead | ~3 pointers (left, right, parent) + color bit |
| Single operation overhead | O(1) |

---

---

## 4. Side-by-Side Comparison

| Feature | `set` | `unordered_set` | `multiset` |
|---|---|---|---|
| **Underlying DS** | Red-Black Tree | Hash Table | Red-Black Tree |
| **Ordering** | ✅ Sorted | ❌ None | ✅ Sorted |
| **Duplicates** | ❌ Ignored | ❌ Ignored | ✅ Stored |
| **Insert** | O(log n) | O(1) avg / O(n) worst | O(log n) |
| **Find** | O(log n) | O(1) avg / O(n) worst | O(log n) — first occurrence |
| **Erase by value** | O(log n) | O(1) avg / O(n) worst | O(log n + k) — removes **ALL** |
| **Erase by iterator** | O(log n) | O(1) avg | O(log n) — removes **ONE** |
| **Count** | O(log n), max 1 | O(1) avg, max 1 | O(log n + k), can be > 1 |
| **lower / upper_bound** | ✅ O(log n) | ❌ Not available | ✅ O(log n) |
| **equal_range** | ✅ O(log n) | ❌ Not available | ✅ O(log n) |
| **Iterator stability on insert** | ✅ Stable | ❌ Invalidated on rehash | ✅ Stable |
| **Total space** | O(n) | O(n) + bucket overhead | O(n) |

---

---

## 5. When to Use What

```
Need a set-like container?
        │
        ├─► Need duplicates?
        │       └─ YES → multiset<T>
        │                  ├─ erase(iterator) → removes ONE copy
        │                  └─ erase(value)    → removes ALL copies ⚠️
        │
        └─► No duplicates needed
                │
                ├─► Need sorted order / range queries / lower_bound?
                │       └─ YES → set<T>
                │
                └─► Only need fast lookup, no ordering required?
                        └─ YES → unordered_set<T>
                                   └─ Use reserve() if input size is known upfront
```

---

### Critical Edge Case Cheatsheet

| Situation | What Happens | Safe Pattern |
|---|---|---|
| Insert duplicate into `set` | Silently ignored | Check `insert().second` if you care |
| Insert duplicate into `multiset` | Stored as a new node | Expected behaviour |
| `erase(value)` on `multiset` | Removes **ALL** copies | Use `erase(find(value))` to remove one |
| `find` on missing element | Returns `end()` | Always check `it != st.end()` |
| Dereference `end()` | Undefined behaviour | Never skip the `end()` check |
| `std::lower_bound` on set/multiset | O(n) — wrong tool | Use `st.lower_bound()` — O(log n) |
| Modify container during iteration | Undefined behaviour | Collect changes, apply after loop |
| `unordered_set` with bad hash | O(n) all operations | Use custom randomized hash |
| `unordered_set` iterator after insert | May be invalidated by rehash | Don't cache iterators across inserts |
| `multiset::count` on large k | Approaches O(n) | Use `equal_range` to iterate instead |

---

> **One-line memory aid:**  
> `set` = sorted + unique &nbsp;·&nbsp; `unordered_set` = fast + unique &nbsp;·&nbsp; `multiset` = sorted + duplicates allowed
