# C++ STL Basics: Pairs & Vectors

This guide explains **Pairs** and **Vectors** in C++ STL using clean examples and concepts derived from the provided code. Perfect for beginners learning STL fundamentals.

---

# 📌 1. PAIRS in C++
A **pair** is a simple container that stores **two values** (can be of different data types). Useful when you want to keep two related values together.

## ✅ How to Declare a Pair
```cpp
pair<int, string> p;
```

## ✨ Ways to Assign Values
### 1. Using `make_pair()`
```cpp
p = make_pair(2, "ABC");
```
### 2. Using initializer list
```cpp
p = {2, "ABC"};
```
### 📤 Accessing Pair Elements
```cpp
cout << p.first << " " << p.second;
```

---

## 📌 Copying a Pair
### Copy by Value (Not Referenced)
```cpp
pair<int, string> p1 = p;  // copies values
p1.first = 3;              // does NOT affect original pair
```

### Copy by Reference
```cpp
pair<int, string>& p1 = p;
p1.first = 3;  // modifies original pair
```

---

## 📌 Why Use Pair?
Pairs help group two related values.

Example: Storing corresponding values of two arrays.
```cpp
pair<int, int> pArr[3];
pArr[0] = {1, 2};
pArr[1] = {2, 3};
pArr[2] = {3, 4};
```

### 🔄 Swapping Pairs
```cpp
swap(pArr[0], pArr[2]);
```

### 📥 Input in Pair
```cpp
pair<int, string> p;
cin >> p.first;
cin >> p.second;
```

---
# 📌 2. VECTORS in C++
A **vector** is a dynamic array whose size can grow at runtime.

```cpp
vector<int> arr;  // Dynamic size
int a[10];        // Fixed size array
```

You can also declare:
```cpp
vector<double> arr1;
vector<pair<int, string>> arr2;
```

---

## 📥 Input in Vectors
```cpp
vector<int> arr;
int n;
cin >> n;

for(int i = 0; i < n; i++){
    int x;
    cin >> x;
    arr.push_back(x);  // adds elements dynamically
}
```

---

## ⚠️ Vector Size Limits
Same as arrays:
- **Local limit**: up to `10^5`
- **Global limit**: up to `10^7`

---

# 📌 Vector Functions & Concepts

## 1. Declaring Vector with Initial Size
```cpp
vector<int> v(5);   // size = 5
v.push_back(7);     // size = 6
```

## 2. Prefilled Vector
```cpp
vector<int> v(5, 1);  // {1,1,1,1,1}
v.push_back(7);
```

## 3. Remove Last Element
```cpp
v.pop_back();
```

---

# 📌 Copying Vectors
### Copy by Reference
```cpp
vector<int> v = {7, 6};
vector<int>& v2 = v;   // reference copy
v2.push_back(5);
```
**Any change in `v2` affects `v`.**

---

# 📌 Utility Function Used in Code
```cpp
void printVec(vector<int>& arr) {
    cout << "size: " << arr.size() << endl;
    for(int i = 0; i < arr.size(); i++){
        cout << arr[i] << " ";
    }
    arr.push_back(4); // demonstrates reference behavior
    cout << endl;
}
```
This shows:
- How vector contents print
- How `push_back()` affects vector passed by reference

---

# ✅ Summary
### **Pairs**
- Store 2 related values
- Access using `.first` & `.second`
- Can copy by value or by reference
- Useful for mapping related data

### **Vectors**
- Dynamic arrays
- Support `push_back`, `pop_back`, copying, prefilling
- Can store any data type including pairs

---

# 🎯 Final Tip
STL Pairs & Vectors are the foundation for advanced topics like **maps, sets, priority queues**, and competitive programming techniques.

Keep this as a quick revision cheat sheet!

