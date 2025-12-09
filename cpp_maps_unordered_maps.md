# C++ STL — Maps & Unordered Maps
A complete and detailed guide to **`map`** and **`unordered_map`** in C++ STL with explanations, code snippets, complexities, and use-cases.

---

# 🔷 MAP (Ordered Map)
`map` stores **key–value pairs**, where:
- Keys are **sorted** (in ascending order by default)
- Implemented using **Red-Black Tree** → ensures **O(log N)** for most operations
- Keys must be **unique**

### ✔ Basic Example
```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    map<int,string> mpp;

    mpp[1] = "abc";
    mpp[5] = "efg";
    mpp[3] = "hij";
    mpp.insert({4, "aaa"});

    for (auto &it : mpp) {
        cout << it.first << "->" << it.second << endl;
    }
}
```
### 🔸 Notes
- Keys are printed in sorted order → **1,3,4,5**
- Inserting with `mpp[key] = value` → **O(log N)**

---

# 🔷 Printing a Map (TC Explained)
```cpp
void printMap(map<int, string>& mpp) {
    cout << "size:" << mpp.size() << endl;

    // Accessing also takes O(Log N)
    // Loop: O(N Log N)
    for (auto &it : mpp) {
        cout << it.first << "->" << it.second << endl;
    }
}
```
### Why O(N log N)?
- Each iteration retrieves a key-value → tree traversal → O(log N)
- Done for N elements → **N × logN**

---

# 🔷 Handling Duplicate Insertions
```cpp
mpp[5] = "BBB";
mpp[5] = "CCC";   // Overwrites old value
```
`map` does **not allow duplicate keys** → value gets replaced.

---

# 🔷 Searching in Map → `find()`
```cpp
auto it = mpp.find(0);  // O(log N)

if (it == mpp.end()) cout << "No Value";
else cout << it->first << " " << it->second;
```

### ✔ `find()` returns:
- Iterator to key
- `mpp.end()` if NOT found

---

# 🔷 Erasing from Map
```cpp
auto it = mpp.find(1);
mpp.erase(it);     // O(log N)
```
Works in **logarithmic time**.

---

# 🔷 Map with `string` Keys
```cpp
map<string, string> mpp;

mpp["ABC"] = "abc";   // TC: |S| * logN
printMap(mpp);
```
### Why extra `|S|` cost?
String comparisons during key comparison.

---

# 🔷 Problem: Count Unique Strings (Lexicographical Order)
```
N ≤ 10^5
|S| ≤ 100
```
```cpp
int N;
cin >> N;
map<string, int> mpp;

for (int i = 0; i < N; i++) {
    string s;
    cin >> s;
    mpp[s]++;        // O(log N)
}

for (auto pr : mpp) {
    cout << pr.first << " ->" << pr.second << endl;
}
```
### Why `map` here?
- Maintains **sorted order** automatically → lexicographical output

---

# 🔶 UNORDERED MAP
`unordered_map` stores key-value pairs in **hash tables**.

### Key Features
| Feature | `map` | `unordered_map` |
|---------|--------|-----------------|
| Ordering | Sorted | No order |
| Implementation | Red-Black Tree | Hash Table |
| Average Complexity | O(log N) | O(1) |
| Worst Case | O(log N) | O(N) |
| Keys allowed | All comparable keys | Only hashable keys |

---

# 🔷 Basic Example of `unordered_map`
```cpp
unordered_map<int, string> mpp;

mpp[0] = "One";   // O(1)
mpp[1] = "Two";
mpp[2] = "Three";
mpp[3] = "Four";

auto it = mpp.find(1);  // O(1)
if (it != mpp.end()) {
    mpp.erase(it);      // O(1)
}

// O(N)
for (auto pr : mpp) {
    cout << pr.first << " " << pr.second << endl;
}
```

### Important
- No guaranteed order
- Faster on average
- Keys must be hashable (e.g., `int`, `string`, not `pair` unless custom hash)

---

# 🔷 Use case: Frequency Queries with Huge Constraints
```
N ≤ 10^6
|S| ≤ 100
Q ≤ 10^6
```
→ Must use `unordered_map` (O(1) avg)

```cpp
unordered_map<string, int> mpp;

int n;
cin >> n;

for (int i = 0; i < n; i++) {
    string s;
    cin >> s;
    mpp[s]++;          // O(1)
}

int q;
cin >> q;

while (q--) {
    string s;
    cin >> s;
    cout << mpp[s] << endl;   // O(1)
}
```

### Why not `map`?
Because `map` → O(N log N) for insertions and queries → too slow for 10⁶ operations.

---

# 🔷 Summary Comparison
### Use **`map`** when:
✔ You need sorted keys
✔ Order matters
✔ Must iterate in lexicographical order
✔ Need lower_bound/upper_bound

### Use **`unordered_map`** when:
✔ You want fastest average performance
✔ Order doesn't matter
✔ Handling large input sizes (~10⁶)

---

If you'd like, I can also add:
- `multimap` and `unordered_multimap`
- Diagram of Red-Black Tree vs Hash table
- Practice questions
Just tell me and I will add!# Maps & Unordered Maps in C++ (Detailed Guide)

## ⭐ Introduction
C++ provides two very important associative containers in STL:

- **`map`** → implemented using **Red-Black Tree (balanced BST)**
- **`unordered_map`** → implemented using **Hash Table**

Both store key–value pairs, but differ in ordering, speed, and use cases.

---

# 📌 `map` in C++ (Ordered Map)
- Stores values in **sorted order** of keys.
- Implemented using **Red-Black Tree** → all operations are **O(log N)**.
- Keys must be **unique**.
- Supports any comparable datatype as key.

---

## ✔ Basic Example
```cpp
#include <bits/stdc++.h>
using namespace std;

int main() 
{
    map<int,string> mpp;
    
    mpp[1] = "abc";
    mpp[5] = "efg";
    mpp[3] = "hij";
    mpp.insert({4, "aaa"});
    
    for(auto &it : mpp) {
      cout << it.first << "->" << it.second << endl;
    }
}
```
### ✔ Output is sorted by key
```
1->abc
3->hij
4->aaa
5->efg
```

---

# 🧩 Inserting, Updating & Printing Map
```cpp
map<int,string> mpp;

mpp[1] = "AAA";   // O(log n)
mpp[5] = "BBB";
mpp[5] = "CCC";   // updates value
mpp[3] = "DDD";
mpp.insert({4, "DDD"});
```

### Why `O(log N)`?
Because every operation works on a balanced BST.

---

## 📌 Printing With Function
```cpp
void printMap(map<int, string>& mpp) {
  cout << "size: " << mpp.size() << endl;
  for(auto &it : mpp) {
      cout << it.first << "->" << it.second << endl;
  }
}
```

---

# 🔍 Searching in map (`find`)
```cpp
auto it = mpp.find(0); // O(log n)

if(it == mpp.end()) {
  cout << "No Value";
} else {
  cout << it->first << " " << it->second << endl;
}
```

### `find()` returns iterator to key, or `end()` if not found.

---

# ❌ Deleting From Map
```cpp
auto it = mpp.find(1);
mpp.erase(it); // O(log n)
printMap(mpp);
```

You can also do:
```cpp
mpp.erase(5); // erase by key, O(log n)
```

---

# 🔠 Map With String Keys
```cpp
map<string,string> mpp;

mpp["ABC"] = "abc"; // TC = |string| * log(n)
printMap(mpp);
```

### Why extra cost?
Comparing strings while sorting → **O(|S|)**.

---

# 📝 Problem: Print Unique Strings & Frequency
### Constraints
```
N ≤ 10^5
|S| ≤ 100
```

### Solution (map gives lexicographical order automatically)
```cpp
int N;
cin >> N;

map<string, int> mpp;

for(int i=0;i<N;i++) {
  string s;
  cin >> s;
  mpp[s]++;
}

for(auto pr : mpp) {
  cout << pr.first << " ->" << pr.second << endl;
}
```

---

# 🚀 `unordered_map` (Fast Hash Map)
### ✔ Properties
- Implemented using **Hash Table**.
- Average complexity → **O(1)** for insert, find, erase.
- Worst case → **O(N)** (rare, due to hash collisions).
- Does **NOT** store keys in sorted order.
- Only supports datatypes that have valid hash functions.

---

## ✔ Basic Example
```cpp
unordered_map<int, string> mpp;

mpp[0] = "One";    // O(1)
mpp[1] = "Two";
mpp[2] = "Three";
mpp[3] = "Four";

auto it = mpp.find(1); // O(1)
if(it != mpp.end()) {
    mpp.erase(it);      // O(1)
}

for(auto pr : mpp) {
    cout << pr.first << " " << pr.second << endl;
}
```

---

# 📝 Problem: String Frequencies With Many Queries
### Constraints
```
N ≤ 10^6
|S| ≤ 100
Q ≤ 10^6
```
### Use `unordered_map` for **O(1)** average
```cpp
unordered_map<string, int> mpp;

int n;
cin >> n;

for(int i=0;i<n;i++) {
    string s;
    cin >> s;
    mpp[s]++;
}

int q;
cin >> q;

while(q--) {
    string s;
    cin >> s;
    cout << mpp[s] << endl;
}
```

---

# 🆚 Map vs Unordered Map
| Feature | `map` | `unordered_map` |
|--------|-------|------------------|
| Internal structure | Red-Black Tree | Hash Table |
| Ordering | Sorted order | No order |
| Insert, Find, Erase | O(log N) | O(1) avg |
| Worst Case | O(log N) | O(N) |
| String key cost | O(|S| log N) | O(|S|) |
| Uses | Sorted tasks, ordered output | Very fast lookups |

---

# ✔ When to Use What?
### Use **map** when:
- You need **sorted keys**.
- You need **lower_bound / upper_bound**.
- You want **predictable traversal order**.

### Use **unordered_map** when:
- Speed is priority.
- No need for order.
- Very large input size.

---

# 🎯 Summary
- `map` → ordered, slower → O(log N)
- `unordered_map` → not ordered, fastest → O(1)
- Use depending on problem requirements.

---

Let me know if you want **Multimap**, **Unordered_multimap**, **custom comparator maps**, or **hash function