# C++ STL Stack and Queue — Complete Guide

---

## Table of Contents

1. [Stack](#1-stack)
2. [Queue](#2-queue)
3. [Stack vs Queue — Side by Side](#3-stack-vs-queue--side-by-side)
4. [When to Use What](#4-when-to-use-what)

---

## 1. Stack

### What Is a Stack?

A stack is a **LIFO** container — **Last In, First Out**.

The last element you push in is the first one to come out.
Think of a stack of plates — you always pick from the top.

```
Push order:  1 → 2 → 3 → 4

Stack state (top to bottom):

    ┌───┐
    │ 4 │  ← top (last pushed, first out)
    ├───┤
    │ 3 │
    ├───┤
    │ 2 │
    ├───┤
    │ 1 │  ← bottom (first pushed, last out)
    └───┘

Pop order:   4 → 3 → 2 → 1
```

---

### Internal Structure

By default, `stack<T>` is backed by a **`deque<T>`** internally.
You can also use `vector` or `list` as the underlying container:

```
stack<int>                  →  uses deque<int>   (default)
stack<int, vector<int>>     →  uses vector<int>
stack<int, list<int>>       →  uses list<int>
```

The stack is just a **wrapper** (adapter) — it restricts access to only one end (the top).
You cannot access the middle or bottom directly.

---

### Operations

---

#### `push(value)`

Adds an element to the **top** of the stack.

```
Before:              After push(5):
  ┌───┐               ┌───┐
  │ 3 │  ← top        │ 5 │  ← top (new)
  ├───┤               ├───┤
  │ 2 │               │ 3 │
  ├───┤               ├───┤
  │ 1 │               │ 2 │
  └───┘               ├───┤
                      │ 1 │
                      └───┘
```

| Case | Time Complexity |
|------|----------------|
| Normal push | O(1) amortized |
| Push triggers reallocation (vector backend) | O(n) — rare |

**Space:** O(1) per push.

**Edge Cases:**
- No size limit enforced by the stack itself — pushing indefinitely will exhaust heap memory.
- With `vector` backend, occasional O(n) reallocation happens (like `vector::push_back`), but amortized O(1).

---

#### `top()`

Returns a **reference** to the top element — does not remove it.

```
Stack:          top() returns:
  ┌───┐
  │ 4 │  ← top    →    4   (still in stack)
  ├───┤
  │ 3 │
  └───┘
```

| Case | Time Complexity |
|------|----------------|
| Any | O(1) |

**Space:** O(1)

**Edge Cases:**
- Calling `top()` on an **empty stack** is **undefined behaviour** — always check `!st.empty()` first.
- `top()` returns a reference — you can modify the top element in place: `st.top() = 99`.

---

#### `pop()`

Removes the top element. **Does not return it.**

```
Before pop():        After pop():
  ┌───┐               ┌───┐
  │ 4 │  ← removed    │ 3 │  ← new top
  ├───┤               ├───┤
  │ 3 │               │ 2 │
  ├───┤               └───┘
  │ 2 │
  └───┘
```

| Case | Time Complexity |
|------|----------------|
| Any | O(1) |

**Space:** O(1)

**Edge Cases:**
- Calling `pop()` on an **empty stack** is **undefined behaviour**.
- `pop()` does not return the value — if you need it, call `top()` first, save it, then `pop()`.

```
// Safe pattern to read and remove:
if (!st.empty()) {
    int val = st.top();   // save first
    st.pop();             // then remove
}
```

---

#### `empty()`

Returns `true` if the stack has no elements.

| Case | Time Complexity |
|------|----------------|
| Any | O(1) |

**Always check `empty()` before calling `top()` or `pop()`.**

---

#### `size()`

Returns the number of elements in the stack.

| Case | Time Complexity |
|------|----------------|
| Any | O(1) |

---

#### Traversal (Emptying the Stack)

Stack has no iterator — you **cannot** use a range-for loop.
The only way to traverse is to repeatedly `top()` and `pop()` until empty:

```
while (!st.empty()) {
    cout << st.top() << endl;   // read top
    st.pop();                   // remove top
}

Prints in LIFO order (last pushed → first printed):
push 1,2,3,4  →  prints 4,3,2,1
```

**Edge Case:** this **destroys** the stack as you go. If you need the elements later, copy the stack first.

---

### Space Complexity Summary for Stack

| What | Space |
|------|-------|
| Stack of n elements | O(n) |
| Per operation | O(1) |

---

### Real-World Use Cases

- **Undo/Redo** — every action pushed, undo pops the last action.
- **Browser back button** — visited pages pushed onto a stack.
- **Balanced parentheses check** — push open brackets, pop on closing.
- **Function call stack** — recursion internally uses a call stack.
- **Reverse a sequence** — push all elements, pop them back out.
- **DFS (Depth First Search)** — iterative DFS uses an explicit stack.

---
---

## 2. Queue

### What Is a Queue?

A queue is a **FIFO** container — **First In, First Out**.

The first element pushed in is the first one to come out.
Think of a line at a ticket counter — first person in line is served first.

```
Push order:  "abc" → "efg" → "sgg" → "tfs"

Queue state:

  front                              back
    ↓                                  ↓
  ┌───────┬───────┬───────┬───────┐
  │ "abc" │ "efg" │ "sgg" │ "tfs" │
  └───────┴───────┴───────┴───────┘
    ↑                                  ↑
  exits here (pop)            enters here (push)

Pop order:   "abc" → "efg" → "sgg" → "tfs"
```

---

### Internal Structure

By default, `queue<T>` is backed by a **`deque<T>`** internally.
Like stack, it is a **wrapper (adapter)** that restricts access:

```
You can only:
  - push to the BACK
  - pop from the FRONT
  - see the FRONT element
  - see the BACK element
```

No access to the middle — the deque's random access is hidden.

---

### Operations

---

#### `push(value)`

Adds an element to the **back** of the queue.

```
Before:                        After push("xyz"):
  ┌───────┬───────┬───────┐     ┌───────┬───────┬───────┬───────┐
  │ "abc" │ "efg" │ "sgg" │     │ "abc" │ "efg" │ "sgg" │ "xyz" │
  └───────┴───────┴───────┘     └───────┴───────┴───────┴───────┘
  front             back        front                      back (new)
```

| Case | Time Complexity |
|------|----------------|
| Any | O(1) amortized |

**Space:** O(1) per push.

---

#### `front()`

Returns a **reference** to the front element — does not remove it.

```
Queue:
  ┌───────┬───────┬───────┐
  │ "abc" │ "efg" │ "sgg" │
  └───────┴───────┴───────┘
      ↑
  front() returns "abc"   (still in queue)
```

| Case | Time Complexity |
|------|----------------|
| Any | O(1) |

**Space:** O(1)

**Edge Cases:**
- Calling `front()` on an **empty queue** is **undefined behaviour** — always check `!que.empty()` first.
- Returns a reference — you can modify in place: `que.front() = "xyz"`.

---

#### `back()`

Returns a **reference** to the back element — does not remove it.

```
Queue:
  ┌───────┬───────┬───────┐
  │ "abc" │ "efg" │ "sgg" │
  └───────┴───────┴───────┘
                       ↑
               back() returns "sgg"
```

| Case | Time Complexity |
|------|----------------|
| Any | O(1) |

**Edge Cases:**
- Calling `back()` on an **empty queue** is **undefined behaviour**.
- `back()` is unique to `queue` — `stack` has no equivalent (you can only see the top).

---

#### `pop()`

Removes the **front** element. **Does not return it.**

```
Before pop():                    After pop():
  ┌───────┬───────┬───────┐       ┌───────┬───────┐
  │ "abc" │ "efg" │ "sgg" │       │ "efg" │ "sgg" │
  └───────┴───────┴───────┘       └───────┴───────┘
      ↑ removed                       ↑ new front
```

| Case | Time Complexity |
|------|----------------|
| Any | O(1) |

**Space:** O(1)

**Edge Cases:**
- Calling `pop()` on an **empty queue** is **undefined behaviour**.
- Same safe pattern as stack — read with `front()` first, then `pop()`.

```
// Safe pattern:
if (!que.empty()) {
    string val = que.front();   // save first
    que.pop();                  // then remove
}
```

---

#### `empty()`

Returns `true` if the queue has no elements.

| Case | Time Complexity |
|------|----------------|
| Any | O(1) |

**Always check `empty()` before calling `front()`, `back()`, or `pop()`.**

---

#### `size()`

Returns the number of elements currently in the queue.

| Case | Time Complexity |
|------|----------------|
| Any | O(1) |

---

#### Traversal (Emptying the Queue)

Queue has no iterator — traverse by reading `front()` and calling `pop()` until empty:

```
while (!que.empty()) {
    cout << que.front() << endl;   // read front
    que.pop();                     // remove front
}

Prints in FIFO order (first pushed → first printed):
push "abc","efg","sgg","tfs"  →  prints "abc","efg","sgg","tfs"
```

---

### Space Complexity Summary for Queue

| What | Space |
|------|-------|
| Queue of n elements | O(n) |
| Per operation | O(1) |

---

### Real-World Use Cases

- **BFS (Breadth First Search)** — processes nodes level by level using a queue.
- **Task scheduling** — tasks processed in the order they arrive.
- **Print spooler** — print jobs queued and handled one by one.
- **Sliding window problems** — queue tracks the window as it moves.
- **Level-order traversal of a tree** — classic BFS with a queue.
- **Producer-consumer problems** — producer pushes, consumer pops from front.

---
---

## 3. Stack vs Queue — Side by Side

```
STACK (LIFO)                         QUEUE (FIFO)
─────────────────────────────────    ─────────────────────────────────

  Push → top                           Push → back
  Pop  ← top                           Pop  ← front

  ┌───┐                                front          back
  │ 4 │ ← top (in & out)                 ↓              ↓
  ├───┤                              ┌───┬───┬───┬───┐
  │ 3 │                              │ 1 │ 2 │ 3 │ 4 │
  ├───┤                              └───┴───┴───┴───┘
  │ 2 │                                ↑ exits        ↑ enters
  ├───┤
  │ 1 │ ← bottom
  └───┘
```

| Feature | `stack` | `queue` |
|---------|---------|---------|
| Order | LIFO | FIFO |
| Insert | `push()` → top | `push()` → back |
| Remove | `pop()` ← top | `pop()` ← front |
| Peek front | `top()` | `front()` |
| Peek back | ❌ not available | `back()` |
| Iterator | ❌ no | ❌ no |
| Default backend | `deque` | `deque` |
| Insert TC | O(1) | O(1) |
| Remove TC | O(1) | O(1) |
| Peek TC | O(1) | O(1) |
| Space | O(n) | O(n) |

---

## 4. When to Use What

```
Need to process elements in order?
          │
          ├─► Process in REVERSE order of arrival?
          │       └─ YES → stack
          │              (last in, first out)
          │              Examples: undo, DFS, balanced brackets
          │
          └─► Process in SAME order of arrival?
                  └─ YES → queue
                         (first in, first out)
                         Examples: BFS, scheduling, level-order traversal
```

### Quick Decision

| Scenario | Use |
|----------|-----|
| Undo/redo, back button | `stack` |
| DFS (iterative) | `stack` |
| Balanced parentheses | `stack` |
| Reverse a sequence | `stack` |
| BFS, level-order traversal | `queue` |
| Task / job scheduling | `queue` |
| Sliding window | `queue` or `deque` |
| Process items in arrival order | `queue` |

---

> **One-line memory aid:**
> `stack` = last plate placed, first plate taken · `queue` = first in line, first served
