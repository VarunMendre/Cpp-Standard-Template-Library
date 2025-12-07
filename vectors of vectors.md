# C++ STL – Array of Vectors & Vector of Vectors
A clean and beginner‑friendly guide explaining **Array of Vectors** and **Vector of Vectors** using the provided code snippets.

---
# 📌 1. Array of `pair<int,int>` Stored in a Vector
This example shows how to store multiple pairs using a vector of pairs.

## ✔ Creating a Vector of Pairs
```cpp
vector<pair<int, int>> v;
int n;
cin >> n;

for(int i = 0; i < n; i++){
    int x, y;
    cin >> x >> y;

    // Option 1
    // v.push_back({x, y});

    // Option 2
    v.push_back(make_pair(x, y));
}
```

## ✔ Printing Vector of Pairs
```cpp
void printVec(vector<pair<int, int>> &v) {
    cout << "size: " << v.size() << endl;
    for(int i = 0; i < v.size(); i++){
        cout << v[i].first << " " << v[i].second << endl;
    }
    cout << endl;
}
```
The function loops through each pair and prints its `.first` and `.second` values.

---
# 📌 2. Array of Vectors
This structure allows defining a **fixed number of vectors**, each of which can have different sizes.

### ✔ Declaration
```cpp
int N;
cin >> N;
vector<int> v[N];  // Array of vectors
```

### ✔ Input for Each Vector
```cpp
for(int i = 0; i < N; i++){
    int n;
    cin >> n;           // size of ith vector

    for(int j = 0; j < n; j++){
        int x;
        cin >> x;
        v[i].push_back(x);
    }
}
```

### ✔ Printing Each Vector
```cpp
for(int i = 0; i < N; i++){
    printVec(v[i]);
}
```

### 📌 Important
- Array size `N` is fixed.
- Each `v[i]` can grow dynamically.
- Useful when number of groups is fixed but group sizes vary.

---
# 📌 3. Vector of Vectors (`vector<vector<int>>`)
This is more flexible than an array of vectors.
- Both **outer** and **inner** sizes are dynamic.
- Most commonly used in competitive programming.

### ✔ Declaration
```cpp
int N;
cin >> N;
vector<vector<int>> v;
```

### ✔ Input Logic
```cpp
for(int i = 0; i < N; i++){
    int n;
    cin >> n;

    vector<int> temp;

    for(int j = 0; j < n; j++){
        int x;
        cin >> x;
        temp.push_back(x);
    }

    v.push_back(temp);
}
```
- `temp` stores one full vector.
- After filling, push into main vector.

### ✔ Printing All Vectors
```cpp
for(int i = 0; i < v.size(); i++){
    printVec(v[i]);
}
```

### ✔ Accessing Elements
```cpp
cout << v[2][1];  // prints 2nd element of 3rd vector
```

---
# 📌 Array of Vectors vs Vector of Vectors
| Feature | Array of Vectors | Vector of Vectors |
|--------|------------------|--------------------|
| Outer Size | Fixed (known before execution) | Dynamic (can grow anytime) |
| Memory | Static (for outer array) | Fully dynamic |
| Usage | When number of groups is known | Most flexible, commonly used |
| Syntax | `vector<int> v[N]` | `vector<vector<int>> v` |

---
# 📌 When to Use What?
### Use **Array of Vectors** when:
- Number of lists is fixed.
- You want slightly faster access.

### Use **Vector of Vectors** when:
- Number of lists changes at runtime.
- You need full dynamic behavior.
- You're doing CP problems like adjacency lists, matrices, etc.

---
# 🎯 Summary
### ✔ Vector of pairs → Stores multiple pairs neatly
### ✔ Array of vectors → Fixed outer size + variable inner sizes
### ✔ Vector of vectors → Completely dynamic, most flexible option

This mini‑guide gives a strong foundation for working with nested STL structures!

