# 🚀 Lambda Functions & In-built Algorithms in C++ STL

This document explains how **lambda functions** work along with some powerful **STL algorithms** like `all_of`, `any_of`, `none_of`, and `sort`.

---

## 🔹 What is a Lambda Function?

A **lambda function** is an anonymous (unnamed) function that can be defined inline.

### Syntax:
```cpp
[capture](parameters) {
    // function body
};
```

### Example:
```cpp
auto func = [](int x, int y){
    return x + y;
};

cout << func(2, 2); // Output: 4
```

👉 Here:
- `[]` → Capture clause (captures variables from outside scope)
- `(int x, int y)` → Parameters
- `{ return x + y; }` → Function body

---

## 🔹 Code Explanation

```cpp
vector<int> arr = {5, 3, 4};
```
We create a vector with elements `{5, 3, 4}`.

---

## 🔸 1. all_of()

```cpp
cout << all_of(arr.begin(), arr.end(), [](int x)
               { return x > 0; });
```

### 💡 Meaning:
Checks if **ALL elements** satisfy the condition.

### 🧠 Logic:
- For each element `x`
- Check if `x > 0`
- If all return `true` → Output: `1` (true)

### ✅ Output:
```
1
```

---

## 🔸 2. any_of()

```cpp
cout << any_of(arr.begin(), arr.end(), [](int x)
               { return x > 0; });
```

### 💡 Meaning:
Checks if **ANY element** satisfies the condition.

### 🧠 Logic:
- If at least one element returns `true`
- Output becomes `1`

### ✅ Output:
```
1
```

---

## 🔸 3. none_of()

```cpp
cout << none_of(arr.begin(), arr.end(), [](int x)
                { return x > 0; });
```

### 💡 Meaning:
Checks if **NO elements** satisfy the condition.

### 🧠 Logic:
- If all elements return `false` → Output: `1`
- Otherwise → Output: `0`

### ❌ In this case:
All elements are greater than 0, so condition is true for all.

### ✅ Output:
```
0
```

---

## 🔸 4. sort() with Lambda

```cpp
sort(arr.begin(), arr.end(),
     [](int x, int y)
     { return x > y; });
```

### 💡 Meaning:
Sorts the array using a custom comparator.

### 🧠 Logic:
- `x > y` → Sort in **descending order**

### Before Sorting:
```
5 3 4
```

### After Sorting:
```
5 4 3
```

---

## 🔸 Final Loop

```cpp
for (int i = 0; i < arr.size(); i++)
{
    cout << arr[i] << " ";
}
```

Prints all elements of the sorted vector.

---

## 🎯 Final Output

```
1
1
0
5 4 3
```

---

## 🧠 Key Takeaways

- Lambda functions allow writing **inline logic without defining separate functions**.
- `all_of()` → All elements must satisfy condition
- `any_of()` → At least one element satisfies condition
- `none_of()` → No elements satisfy condition
- `sort()` with lambda → Custom sorting logic

---

## ⭐ Pro Tip

Lambda functions are widely used in:
- STL algorithms
- Competitive programming
- Cleaner and shorter code

---

Happy Coding! 💻🔥

