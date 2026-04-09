# 🧠 Data Structures & Algorithms — Complete Interview Guide

> **Part 6 of Ahmed's shadow interview prep.**
> Every concept in **C++, Java, TypeScript, and Python** — a map for developers from any background.
> Topics ordered by logical prerequisite: each concept builds on the previous.

---

## 📑 Table of Contents

### Foundations
1. [Time & Space Complexity (Big O)](#1-time--space-complexity-big-o)

### Data Structures
2. [Arrays — Static & Dynamic](#2-arrays--static--dynamic)
3. [Linked Lists — Singly & Doubly](#3-linked-lists--singly--doubly)
4. [Stacks](#4-stacks)
5. [Queues, Deques & Circular Queues](#5-queues-deques--circular-queues)
6. [HashMaps & Sets](#6-hashmaps--sets)
7. [Trees & Binary Search Trees](#7-trees--binary-search-trees)
8. [Priority Queues & Heaps](#8-priority-queues--heaps)
9. [Graphs](#9-graphs)

### Algorithms — Basics
10. [Sorting Algorithms](#10-sorting-algorithms)
11. [Binary Search — Traditional & Condition-Based](#11-binary-search--traditional--condition-based)
12. [Recursion](#12-recursion)
13. [Recursive Backtracking](#13-recursive-backtracking)

### Algorithms — Traversal & Search
14. [Depth-First Search (DFS)](#14-depth-first-search-dfs)
15. [Breadth-First Search (BFS)](#15-breadth-first-search-bfs)

### Algorithms — Techniques
16. [Two Pointers & Sliding Window](#16-two-pointers--sliding-window)
17. [Binary Numbers & Bit Manipulation](#17-binary-numbers--bit-manipulation)

### Advanced Algorithms
18. [Greedy Algorithms](#18-greedy-algorithms)
19. [Dynamic Programming](#19-dynamic-programming)

### Reference
20. [Interview Cheatsheet & Common Patterns](#20-interview-cheatsheet--common-patterns)

---

## 1. Time & Space Complexity (Big O)

Big O notation describes how the runtime or memory usage of an algorithm **grows** as the input size grows. It describes the **worst case** unless stated otherwise.

### The Complexity Classes

```
O(1)       Constant    — doesn't grow with input (hash map lookup, array index)
O(log n)   Logarithmic — halves the problem each step (binary search)
O(n)       Linear      — one pass through input (linear search, single loop)
O(n log n) Linearithmic— sort a list (merge sort, heap sort)
O(n²)      Quadratic   — nested loops (bubble sort, insertion sort)
O(n³)      Cubic       — triple nested loops (naive matrix multiply)
O(2ⁿ)      Exponential — all subsets (brute-force subset problems)
O(n!)      Factorial   — all permutations (brute-force TSP)

Growth order (slowest to fastest):
O(1) < O(log n) < O(√n) < O(n) < O(n log n) < O(n²) < O(2ⁿ) < O(n!)
```

### Visualizing Growth

```
n = 1,000 operations per second budget

n       O(1)  O(log n)  O(n)     O(n log n)  O(n²)        O(2ⁿ)
10      1     3         10       33           100          1,024
100     1     7         100      664          10,000       1.27×10³⁰
1,000   1     10        1,000    9,966        1,000,000    too large
10,000  1     14        10,000   132,877      100,000,000  heat death of universe
```

### How to Analyze Complexity — The Rules

```python
# Rule 1: Drop constants
# O(2n) → O(n),  O(n/2) → O(n),  O(3n² + 5n + 7) → O(n²)

# Rule 2: Sequential steps ADD
def example(arr):
    for x in arr: pass      # O(n)
    for x in arr: pass      # O(n)
    # Total: O(n + n) = O(2n) = O(n)

# Rule 3: Nested steps MULTIPLY
def example(arr):
    for x in arr:           # O(n)
        for y in arr:       # O(n)
            pass            # Total: O(n × n) = O(n²)

# Rule 4: Different variables = different letters
def example(a, b):
    for x in a: pass        # O(n) where n = len(a)
    for x in b: pass        # O(m) where m = len(b)
    # Total: O(n + m)  NOT O(2n)

# Rule 5: Drop non-dominant terms
# O(n² + n) → O(n²)   (n becomes irrelevant for large n)
# O(n + 500) → O(n)   (500 is constant)

# Rule 6: Recursion = depth × work per level
# Binary search: depth = log n, work per level = O(1) → O(log n)
# Merge sort: depth = log n, work per level = O(n) → O(n log n)
```

### Space Complexity

```
Space complexity = extra memory used by the algorithm
(input itself usually not counted)

O(1) — fixed number of variables regardless of input
O(n) — creating an array the size of input (copy, output array)
O(n²)— 2D matrix the size of input
O(h) — recursion stack of depth h (h = log n for balanced tree, h = n for skewed)
```

### Quick Reference — Common Operations

| Operation | Array | Linked List | BST (balanced) | Hash Map |
|---|---|---|---|---|
| Access by index | O(1) | O(n) | N/A | N/A |
| Search | O(n) | O(n) | O(log n) | O(1) avg |
| Insert at end | O(1) amort. | O(1) with tail | O(log n) | O(1) avg |
| Insert at front | O(n) | O(1) | O(log n) | N/A |
| Delete | O(n) | O(1) with ref | O(log n) | O(1) avg |
| Min/Max | O(n) | O(n) | O(log n) | O(n) |

### 📖 Resources
- [Big-O Cheat Sheet](https://www.bigocheatsheet.com/)
- [Visualizing Algorithms](https://visualgo.net/en)
- [MIT 6.006 Introduction to Algorithms (free)](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-fall-2011/)

---

## 2. Arrays — Static & Dynamic

An array is a **contiguous block of memory** storing elements of the same type. The most fundamental data structure — everything else builds on this.

### Static vs Dynamic Arrays

```
Static Array:  fixed size at creation, cannot grow (C arrays, Java int[])
Dynamic Array: resizable — when full, allocate 2× space and copy (Python list, Java ArrayList, C++ vector, JS Array)

Dynamic array amortized O(1) append:
  n insertions → n + n/2 + n/4 + ... ≈ 2n copies total
  2n copies / n insertions = O(1) amortized per insertion
```

### Implementation in All 4 Languages

**C++**
```cpp
#include <vector>
#include <iostream>
#include <algorithm>
using namespace std;

int main() {
    // Static array
    int staticArr[5] = {1, 2, 3, 4, 5};
    cout << staticArr[0] << endl; // O(1) access

    // Dynamic array (vector)
    vector<int> v;
    v.push_back(10);  // amortized O(1)
    v.push_back(20);
    v.push_back(30);

    // Common operations
    cout << v.size()     << endl; // 3
    cout << v[1]         << endl; // 20
    v.insert(v.begin() + 1, 15); // O(n) — shifts elements
    v.erase(v.begin());           // O(n) — shifts elements

    // Iteration
    for (int x : v) cout << x << " "; // 15 20 30
    cout << endl;

    // Sort — O(n log n)
    sort(v.begin(), v.end());

    // 2D array
    vector<vector<int>> matrix(3, vector<int>(4, 0)); // 3×4 zeroed
    matrix[1][2] = 42;
}
```

**Java**
```java
import java.util.*;

public class ArrayDemo {
    public static void main(String[] args) {
        // Static array
        int[] staticArr = {1, 2, 3, 4, 5};
        System.out.println(staticArr[0]); // O(1)

        // Dynamic array
        ArrayList<Integer> list = new ArrayList<>();
        list.add(10);       // amortized O(1)
        list.add(20);
        list.add(1, 15);    // O(n) — insert at index 1
        list.remove(0);     // O(n) — remove at index

        System.out.println(list.get(0)); // O(1)
        System.out.println(list.size()); // 2

        // Sort
        Collections.sort(list); // O(n log n)

        // Convert array ↔ ArrayList
        Integer[] arr = list.toArray(new Integer[0]);
        List<Integer> fromArr = Arrays.asList(arr);

        // 2D array
        int[][] matrix = new int[3][4]; // all zeros
        matrix[1][2] = 42;

        // Iteration
        for (int x : list) System.out.print(x + " ");
    }
}
```

**TypeScript**
```typescript
// TypeScript arrays are always dynamic
const arr: number[] = [1, 2, 3, 4, 5];

// Common operations
console.log(arr[0]);         // O(1)
arr.push(6);                 // O(1) amortized — append
arr.unshift(0);              // O(n) — prepend (shifts all)
arr.pop();                   // O(1) — remove last
arr.shift();                 // O(n) — remove first (shifts all)
arr.splice(2, 1);            // O(n) — remove at index 2
arr.splice(1, 0, 99);        // O(n) — insert 99 at index 1

// Functional operations (all O(n), all return NEW array)
const doubled  = arr.map(x => x * 2);
const evens    = arr.filter(x => x % 2 === 0);
const sum      = arr.reduce((acc, x) => acc + x, 0);
const found    = arr.find(x => x > 3);
const idx      = arr.findIndex(x => x > 3);
const includes = arr.includes(99);

// Sort (mutates!) — default sorts LEXICOGRAPHICALLY
arr.sort((a, b) => a - b); // numeric ascending
arr.sort((a, b) => b - a); // numeric descending

// 2D array
const matrix: number[][] = Array.from({ length: 3 }, () => new Array(4).fill(0));
matrix[1][2] = 42;

// Destructuring and spread
const [first, second, ...rest] = arr;
const copy = [...arr]; // shallow copy — O(n)
const merged = [...arr, ...doubled];
```

**Python**
```python
# Python lists are dynamic arrays
arr = [1, 2, 3, 4, 5]

# Common operations
print(arr[0])           # O(1) — access
print(arr[-1])          # O(1) — last element
arr.append(6)           # O(1) amortized
arr.insert(0, 0)        # O(n) — shift right
arr.pop()               # O(1) — remove last
arr.pop(0)              # O(n) — remove first
arr.remove(3)           # O(n) — remove first occurrence of 3
idx = arr.index(2)      # O(n) — find index
arr.sort()              # O(n log n) — in-place
sorted_arr = sorted(arr)# O(n log n) — returns new
arr.reverse()           # O(n) — in-place
arr_copy = arr[:]       # O(n) — slice = shallow copy
arr_copy = arr.copy()   # same

# List comprehension — Pythonic way to build arrays
squares  = [x**2 for x in range(10)]
evens    = [x for x in arr if x % 2 == 0]
matrix   = [[0] * 4 for _ in range(3)]  # 3×4 zeroed
# ⚠️ DON'T do: [[0]*4]*3 — creates same row reference 3 times!

# Slicing — all O(k) where k is slice size
sub  = arr[1:4]         # indices 1,2,3
rev  = arr[::-1]        # reversed copy
step = arr[::2]         # every other element

# Unpacking
first, *middle, last = arr

# Useful functions
print(len(arr))         # O(1)
print(sum(arr))         # O(n)
print(min(arr))         # O(n)
print(max(arr))         # O(n)
```

### Key Array Patterns

```python
# Pattern 1: Two-pointer reverse
def reverse_array(arr):
    left, right = 0, len(arr) - 1
    while left < right:
        arr[left], arr[right] = arr[right], arr[left]
        left += 1
        right -= 1

# Pattern 2: Prefix sums — precompute for O(1) range sum queries
def build_prefix(arr):
    prefix = [0] * (len(arr) + 1)
    for i, x in enumerate(arr):
        prefix[i + 1] = prefix[i] + x
    return prefix

# range_sum(l, r) = prefix[r+1] - prefix[l]

# Pattern 3: Kadane's Algorithm — max subarray sum O(n)
def max_subarray(arr):
    max_sum = current = arr[0]
    for x in arr[1:]:
        current = max(x, current + x)
        max_sum = max(max_sum, current)
    return max_sum
```

---

## 3. Linked Lists — Singly & Doubly

A linked list is a collection of **nodes** where each node stores a value and a pointer to the next node. Unlike arrays, nodes are not contiguous in memory.

```
Singly Linked:  [1|→] → [2|→] → [3|→] → [4|null]
Doubly Linked:  null←[1|→] ↔ [←2|→] ↔ [←3|→] ↔ [←4]→null
```

### When to Use Linked List over Array

```
Use linked list when:
  ✅ Frequent insertions/deletions at the front or middle (O(1) with pointer)
  ✅ You don't know the size in advance and don't want amortized resizing
  ✅ Implementing stacks, queues, or LRU cache

Prefer array when:
  ✅ You need O(1) random access by index
  ✅ Cache performance matters (contiguous memory = better cache locality)
  ✅ You iterate forward frequently (arrays are faster in practice due to CPU cache)
```

### Singly Linked List — Full Implementation

**C++**
```cpp
#include <iostream>
using namespace std;

struct Node {
    int val;
    Node* next;
    Node(int v) : val(v), next(nullptr) {}
};

class SinglyLinkedList {
public:
    Node* head;
    int size;

    SinglyLinkedList() : head(nullptr), size(0) {}

    // Prepend — O(1)
    void prepend(int val) {
        Node* node = new Node(val);
        node->next = head;
        head = node;
        size++;
    }

    // Append — O(n) without tail pointer, O(1) with tail
    void append(int val) {
        Node* node = new Node(val);
        if (!head) { head = node; size++; return; }
        Node* curr = head;
        while (curr->next) curr = curr->next;
        curr->next = node;
        size++;
    }

    // Delete by value — O(n)
    bool deleteVal(int val) {
        if (!head) return false;
        if (head->val == val) {
            Node* tmp = head;
            head = head->next;
            delete tmp;
            size--;
            return true;
        }
        Node* curr = head;
        while (curr->next && curr->next->val != val)
            curr = curr->next;
        if (!curr->next) return false;
        Node* tmp = curr->next;
        curr->next = tmp->next;
        delete tmp;
        size--;
        return true;
    }

    // Reverse — O(n) — KEY interview question
    void reverse() {
        Node* prev = nullptr;
        Node* curr = head;
        while (curr) {
            Node* next = curr->next; // save next
            curr->next = prev;       // reverse pointer
            prev = curr;             // advance prev
            curr = next;             // advance curr
        }
        head = prev;
    }

    // Detect cycle — Floyd's algorithm O(n) time, O(1) space
    bool hasCycle() {
        Node* slow = head;
        Node* fast = head;
        while (fast && fast->next) {
            slow = slow->next;
            fast = fast->next->next;
            if (slow == fast) return true;
        }
        return false;
    }

    // Find middle — slow/fast pointers
    Node* findMiddle() {
        Node* slow = head;
        Node* fast = head;
        while (fast && fast->next) {
            slow = slow->next;
            fast = fast->next->next;
        }
        return slow; // slow is at middle
    }

    void print() {
        Node* curr = head;
        while (curr) {
            cout << curr->val;
            if (curr->next) cout << " → ";
            curr = curr->next;
        }
        cout << " → null\n";
    }

    ~SinglyLinkedList() {
        Node* curr = head;
        while (curr) {
            Node* next = curr->next;
            delete curr;
            curr = next;
        }
    }
};
```

**Java**
```java
public class SinglyLinkedList<T> {
    private static class Node<T> {
        T val;
        Node<T> next;
        Node(T v) { val = v; next = null; }
    }

    private Node<T> head;
    private int size;

    public void prepend(T val) {
        Node<T> node = new Node<>(val);
        node.next = head;
        head = node;
        size++;
    }

    public void append(T val) {
        Node<T> node = new Node<>(val);
        if (head == null) { head = node; size++; return; }
        Node<T> curr = head;
        while (curr.next != null) curr = curr.next;
        curr.next = node;
        size++;
    }

    // Reverse — iterative O(n)
    public void reverse() {
        Node<T> prev = null, curr = head;
        while (curr != null) {
            Node<T> next = curr.next;
            curr.next = prev;
            prev = curr;
            curr = next;
        }
        head = prev;
    }

    // Floyd's cycle detection
    public boolean hasCycle() {
        Node<T> slow = head, fast = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            if (slow == fast) return true;
        }
        return false;
    }

    public void print() {
        Node<T> curr = head;
        while (curr != null) {
            System.out.print(curr.val + (curr.next != null ? " → " : ""));
            curr = curr.next;
        }
        System.out.println(" → null");
    }
}
```

**TypeScript**
```typescript
class ListNode<T> {
  val: T;
  next: ListNode<T> | null = null;
  constructor(val: T) { this.val = val; }
}

class SinglyLinkedList<T> {
  head: ListNode<T> | null = null;
  size = 0;

  prepend(val: T): void {
    const node = new ListNode(val);
    node.next = this.head;
    this.head = node;
    this.size++;
  }

  append(val: T): void {
    const node = new ListNode(val);
    if (!this.head) { this.head = node; this.size++; return; }
    let curr = this.head;
    while (curr.next) curr = curr.next;
    curr.next = node;
    this.size++;
  }

  // Reverse — iterative O(n)
  reverse(): void {
    let prev: ListNode<T> | null = null;
    let curr = this.head;
    while (curr) {
      const next = curr.next;
      curr.next = prev;
      prev = curr;
      curr = next;
    }
    this.head = prev;
  }

  // Floyd's cycle detection
  hasCycle(): boolean {
    let slow = this.head, fast = this.head;
    while (fast && fast.next) {
      slow = slow!.next;
      fast = fast.next.next;
      if (slow === fast) return true;
    }
    return false;
  }

  toArray(): T[] {
    const result: T[] = [];
    let curr = this.head;
    while (curr) { result.push(curr.val); curr = curr.next; }
    return result;
  }
}
```

**Python**
```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

class SinglyLinkedList:
    def __init__(self):
        self.head = None
        self.size = 0

    def prepend(self, val):           # O(1)
        self.head = ListNode(val, self.head)
        self.size += 1

    def append(self, val):            # O(n)
        node = ListNode(val)
        if not self.head:
            self.head = node
            self.size += 1
            return
        curr = self.head
        while curr.next:
            curr = curr.next
        curr.next = node
        self.size += 1

    def reverse(self):                # O(n) — KEY interview pattern
        prev, curr = None, self.head
        while curr:
            nxt = curr.next
            curr.next = prev
            prev = curr
            curr = nxt
        self.head = prev

    def has_cycle(self) -> bool:      # Floyd's — O(n) time, O(1) space
        slow = fast = self.head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
            if slow is fast:
                return True
        return False

    def find_middle(self):            # slow/fast pointer
        slow = fast = self.head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
        return slow                   # slow points to middle

    def to_list(self):
        result, curr = [], self.head
        while curr:
            result.append(curr.val)
            curr = curr.next
        return result
```

### Doubly Linked List

```python
class DNode:
    def __init__(self, val=0):
        self.val = val
        self.prev = None
        self.next = None

class DoublyLinkedList:
    def __init__(self):
        # Sentinel nodes — simplify edge cases (no null checks!)
        self.head = DNode(0)  # dummy head
        self.tail = DNode(0)  # dummy tail
        self.head.next = self.tail
        self.tail.prev = self.head
        self.size = 0

    def _insert_after(self, node: DNode, new_node: DNode):
        """Insert new_node after node — O(1)"""
        new_node.prev = node
        new_node.next = node.next
        node.next.prev = new_node
        node.next = new_node
        self.size += 1

    def _remove(self, node: DNode):
        """Remove a node — O(1) given the node"""
        node.prev.next = node.next
        node.next.prev = node.prev
        self.size -= 1

    def prepend(self, val):
        self._insert_after(self.head, DNode(val))

    def append(self, val):
        self._insert_after(self.tail.prev, DNode(val))

    def remove_first(self):
        if self.size == 0: raise IndexError("Empty list")
        node = self.head.next
        self._remove(node)
        return node.val

    def remove_last(self):
        if self.size == 0: raise IndexError("Empty list")
        node = self.tail.prev
        self._remove(node)
        return node.val
```

### Must-Know Linked List Interview Problems

```python
# 1. Merge Two Sorted Lists — O(n + m)
def merge_sorted(l1, l2):
    dummy = ListNode(0)
    curr = dummy
    while l1 and l2:
        if l1.val <= l2.val:
            curr.next = l1
            l1 = l1.next
        else:
            curr.next = l2
            l2 = l2.next
        curr = curr.next
    curr.next = l1 or l2
    return dummy.next

# 2. Find Nth node from end — one pass O(n)
def nth_from_end(head, n):
    fast = slow = head
    for _ in range(n):          # advance fast by n
        fast = fast.next
    while fast:                 # move both until fast hits end
        slow = slow.next
        fast = fast.next
    return slow                 # slow is n from end

# 3. Check palindrome — O(n) time, O(1) space
def is_palindrome(head):
    # Find middle, reverse second half, compare
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    # Reverse second half
    prev, curr = None, slow
    while curr:
        nxt = curr.next
        curr.next = prev
        prev = curr
        curr = nxt
    # Compare
    left, right = head, prev
    while right:
        if left.val != right.val: return False
        left = left.next
        right = right.next
    return True
```

---

## 4. Stacks

A stack is a **Last-In, First-Out (LIFO)** data structure. Like a stack of plates — you add and remove from the top.

```
Push → [5, 3, 7, 2]
Pop  → removes 2 (top)
Peek → sees 2 without removing
```

### All 4 Implementations

**C++**
```cpp
#include <stack>
#include <vector>
#include <stdexcept>
using namespace std;

// Using STL
stack<int> s;
s.push(10);
s.push(20);
s.push(30);
cout << s.top()  << endl; // 30 — peek
s.pop();                   // removes 30
cout << s.size() << endl; // 2
cout << s.empty()<< endl; // false

// Custom stack using vector (more control)
template<typename T>
class Stack {
    vector<T> data;
public:
    void push(T val) { data.push_back(val); }    // O(1) amortized
    T pop() {
        if (empty()) throw runtime_error("Stack underflow");
        T val = data.back();
        data.pop_back();
        return val;
    }
    T& peek() {
        if (empty()) throw runtime_error("Stack empty");
        return data.back();
    }
    bool empty() const { return data.empty(); }
    int size() const { return data.size(); }
};
```

**Java**
```java
import java.util.*;

// Java's Stack class (legacy — use Deque instead)
Deque<Integer> stack = new ArrayDeque<>();
stack.push(10);          // addFirst
stack.push(20);
stack.push(30);
System.out.println(stack.peek()); // 30
System.out.println(stack.pop());  // 30
System.out.println(stack.size()); // 2

// Custom stack
public class Stack<T> {
    private final List<T> data = new ArrayList<>();

    public void push(T val) { data.add(val); }
    public T pop() {
        if (isEmpty()) throw new EmptyStackException();
        return data.remove(data.size() - 1);
    }
    public T peek() {
        if (isEmpty()) throw new EmptyStackException();
        return data.get(data.size() - 1);
    }
    public boolean isEmpty() { return data.isEmpty(); }
    public int size() { return data.size(); }
}
```

**TypeScript**
```typescript
class Stack<T> {
  private data: T[] = [];

  push(val: T): void { this.data.push(val); }     // O(1) amortized
  pop(): T {
    if (this.isEmpty()) throw new Error("Stack underflow");
    return this.data.pop()!;
  }
  peek(): T {
    if (this.isEmpty()) throw new Error("Stack empty");
    return this.data[this.data.length - 1];
  }
  isEmpty(): boolean { return this.data.length === 0; }
  size(): number { return this.data.length; }
}

// Usage
const stack = new Stack<number>();
stack.push(10); stack.push(20); stack.push(30);
console.log(stack.peek()); // 30
console.log(stack.pop());  // 30
```

**Python**
```python
# Python list works perfectly as a stack
stack = []
stack.append(10)    # push — O(1) amortized
stack.append(20)
stack.append(30)
print(stack[-1])    # peek — O(1)
stack.pop()         # pop — O(1)
print(len(stack))   # 2
print(not stack)    # isEmpty

# Or use collections.deque (thread-safe, O(1) guaranteed)
from collections import deque
stack = deque()
stack.append(10)
stack.append(20)
stack.pop()

# Custom stack class
class Stack:
    def __init__(self):
        self._data = []

    def push(self, val): self._data.append(val)
    def pop(self):
        if self.is_empty(): raise IndexError("Stack underflow")
        return self._data.pop()
    def peek(self):
        if self.is_empty(): raise IndexError("Stack empty")
        return self._data[-1]
    def is_empty(self): return len(self._data) == 0
    def size(self): return len(self._data)
```

### Key Stack Applications & Patterns

**Pattern 1: Valid Parentheses (Classic interview question)**
```python
def is_valid(s: str) -> bool:
    stack = []
    pairs = {')': '(', ']': '[', '}': '{'}
    for ch in s:
        if ch in '([{':
            stack.append(ch)
        elif ch in ')]}':
            if not stack or stack[-1] != pairs[ch]:
                return False
            stack.pop()
    return len(stack) == 0

# Test: "([])" → True,  "([)]" → False,  "{[]}" → True
```

**Pattern 2: Monotonic Stack — Next Greater Element**
```python
def next_greater(arr):
    """For each element, find the next greater element to its right. O(n)"""
    n = len(arr)
    result = [-1] * n
    stack = []  # stores indices of elements waiting for their next greater

    for i in range(n):
        # While stack has elements and current is greater than top
        while stack and arr[i] > arr[stack[-1]]:
            idx = stack.pop()
            result[idx] = arr[i]  # arr[i] is the next greater for arr[idx]
        stack.append(i)

    return result

# [2, 1, 2, 4, 3] → [4, 2, 4, -1, -1]
```

**Pattern 3: Evaluate Postfix Expression**
```python
def eval_postfix(tokens):
    stack = []
    ops = {'+': lambda a,b: a+b, '-': lambda a,b: a-b,
           '*': lambda a,b: a*b, '/': lambda a,b: int(a/b)}
    for token in tokens:
        if token in ops:
            b, a = stack.pop(), stack.pop()  # order matters
            stack.append(ops[token](a, b))
        else:
            stack.append(int(token))
    return stack[0]

# ["2","1","+","3","*"] → 9    (postfix of (2+1)*3)
```

**Pattern 4: Min Stack — Get minimum in O(1)**
```python
class MinStack:
    def __init__(self):
        self.stack = []      # (val, current_min) pairs

    def push(self, val):
        min_val = min(val, self.stack[-1][1]) if self.stack else val
        self.stack.append((val, min_val))

    def pop(self): self.stack.pop()
    def top(self): return self.stack[-1][0]
    def get_min(self): return self.stack[-1][1]  # O(1)!
```

---

## 5. Queues, Deques & Circular Queues

### Queue — First-In, First-Out (FIFO)

```
Enqueue → [front: 1, 2, 3, 4 :rear]
Dequeue → removes 1 (front)
```

**C++**
```cpp
#include <queue>
#include <deque>
using namespace std;

// STL queue
queue<int> q;
q.push(1);    // enqueue
q.push(2);
q.push(3);
cout << q.front() << endl; // 1
cout << q.back()  << endl; // 3
q.pop();                    // dequeue — removes front
cout << q.size()  << endl; // 2
```

**Java**
```java
Deque<Integer> queue = new ArrayDeque<>();
queue.offer(1);   // enqueue
queue.offer(2);
queue.offer(3);
System.out.println(queue.peek());  // 1 — front
System.out.println(queue.poll());  // 1 — dequeue
System.out.println(queue.size());  // 2
```

**TypeScript**
```typescript
// Built on array — O(n) dequeue (shift). Use a proper deque for O(1).
class Queue<T> {
  private data: T[] = [];
  enqueue(val: T): void { this.data.push(val); }
  dequeue(): T {
    if (this.isEmpty()) throw new Error("Queue empty");
    return this.data.shift()!; // O(n) — use deque for O(1)
  }
  front(): T { return this.data[0]; }
  isEmpty(): boolean { return this.data.length === 0; }
  size(): number { return this.data.length; }
}
```

**Python**
```python
from collections import deque

# deque is the correct Python queue — O(1) append AND popleft
q = deque()
q.append(1)     # enqueue — O(1)
q.append(2)
q.append(3)
print(q[0])     # front — O(1)
q.popleft()     # dequeue — O(1) (list.pop(0) is O(n)!)
print(len(q))   # 2
```

### Deque (Double-Ended Queue)

Supports O(1) insert and remove from **both** ends.

```python
from collections import deque

dq = deque()
dq.appendleft(1)  # prepend
dq.append(2)      # append
dq.appendleft(0)  # prepend

print(dq)         # deque([0, 1, 2])
dq.popleft()      # remove front — O(1)
dq.pop()          # remove back  — O(1)
print(dq)         # deque([1])

# Sliding window maximum uses deque!
# Fixed-size deque:
dq = deque(maxlen=3)
for x in [1,2,3,4,5]:
    dq.append(x)  # auto-removes oldest when full
```

```cpp
// C++ deque
#include <deque>
deque<int> dq;
dq.push_front(1);  // O(1)
dq.push_back(2);   // O(1)
dq.pop_front();    // O(1)
dq.pop_back();     // O(1)
cout << dq.front() << " " << dq.back() << endl;
```

### Circular Queue (Ring Buffer) — Fixed Capacity, O(1) All Ops

```python
class CircularQueue:
    """
    Fixed-size queue using array as ring buffer.
    head/tail pointers wrap around using modulo.
    """
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.data = [None] * capacity
        self.head = 0       # index of front element
        self.tail = 0       # index where next element goes
        self.size = 0

    def enqueue(self, val) -> bool:
        if self.is_full(): return False
        self.data[self.tail] = val
        self.tail = (self.tail + 1) % self.capacity  # wrap around!
        self.size += 1
        return True

    def dequeue(self):
        if self.is_empty(): return None
        val = self.data[self.head]
        self.data[self.head] = None  # optional cleanup
        self.head = (self.head + 1) % self.capacity  # wrap around!
        self.size -= 1
        return val

    def front(self):
        return None if self.is_empty() else self.data[self.head]

    def rear(self):
        idx = (self.tail - 1) % self.capacity
        return None if self.is_empty() else self.data[idx]

    def is_empty(self) -> bool: return self.size == 0
    def is_full(self)  -> bool: return self.size == self.capacity
```

```cpp
// C++ Circular Queue
class CircularQueue {
    vector<int> data;
    int head, tail, sz, capacity;
public:
    CircularQueue(int cap) : data(cap), head(0), tail(0), sz(0), capacity(cap) {}

    bool enqueue(int val) {
        if (sz == capacity) return false;
        data[tail] = val;
        tail = (tail + 1) % capacity;
        sz++;
        return true;
    }
    bool dequeue() {
        if (sz == 0) return false;
        head = (head + 1) % capacity;
        sz--;
        return true;
    }
    int front() { return sz ? data[head] : -1; }
    bool isEmpty() { return sz == 0; }
    bool isFull()  { return sz == capacity; }
};
```

### Why Queues? — Common Applications

```
BFS traversal            — process nodes level by level
Task schedulers          — OS process scheduling
Print spooler            — jobs processed in order
Rate limiting (token bucket) — fixed-rate processing
Producer-consumer        — decoupled async processing
Sliding window maximum   — deque stores candidates
```

---

## 6. HashMaps & Sets

A HashMap stores **key-value pairs** with O(1) average lookup, insert, and delete. A Set stores **unique values** with O(1) membership testing.

### How Hashing Works

```
hash(key) → index in array

"name" → hash function → 7 → arr[7] = "Ahmed"
"age"  → hash function → 3 → arr[3] = 25

Collision: two keys hash to same index
  Solution 1: Chaining — each bucket is a linked list
  Solution 2: Open Addressing — probe to find next empty slot

Load factor = n / capacity (number of entries / array size)
Rehash when load factor > 0.7 → copy to new 2× array
```

### All 4 Language Implementations

**C++**
```cpp
#include <unordered_map>
#include <unordered_set>
#include <map>          // ordered by key — O(log n) operations
using namespace std;

// HashMap — unordered_map
unordered_map<string, int> freq;
freq["apple"]  = 1;
freq["banana"] = 2;
freq["apple"]++;         // increment

// Check existence
if (freq.count("apple")) cout << freq["apple"] << endl;
if (freq.find("cherry") == freq.end()) cout << "not found\n";

// Iterate
for (auto& [key, val] : freq) {
    cout << key << ": " << val << endl;
}

// Erase
freq.erase("banana");

// Default value (operator[] creates with default if missing)
unordered_map<char, int> charCount;
string s = "hello";
for (char c : s) charCount[c]++;  // ++ creates entry with 0 first

// HashSet
unordered_set<int> seen;
seen.insert(1);
seen.insert(2);
seen.insert(1);         // duplicate — ignored
cout << seen.size();    // 2
cout << seen.count(1);  // 1 (exists), 0 (doesn't)
seen.erase(1);
```

**Java**
```java
import java.util.*;

// HashMap
Map<String, Integer> map = new HashMap<>();
map.put("apple", 1);
map.put("banana", 2);
map.put("apple", 3);          // overwrites previous value

System.out.println(map.get("apple"));         // 3
System.out.println(map.getOrDefault("cherry", 0)); // 0 (safe default)
System.out.println(map.containsKey("banana"));// true
map.remove("banana");

// Increment pattern (frequency counting)
for (char c : "hello".toCharArray())
    map.put(String.valueOf(c), map.getOrDefault(String.valueOf(c), 0) + 1);

// Iterate
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}

// LinkedHashMap — preserves insertion order
Map<String, Integer> ordered = new LinkedHashMap<>();

// TreeMap — sorted by key O(log n) per op
Map<String, Integer> sorted = new TreeMap<>();

// HashSet
Set<Integer> set = new HashSet<>();
set.add(1); set.add(2); set.add(1);
System.out.println(set.size());       // 2
System.out.println(set.contains(1));  // true
set.remove(1);

// Set operations
Set<Integer> a = new HashSet<>(Arrays.asList(1,2,3));
Set<Integer> b = new HashSet<>(Arrays.asList(2,3,4));
a.retainAll(b);   // intersection in-place: {2,3}
a.addAll(b);      // union in-place
a.removeAll(b);   // difference in-place
```

**TypeScript**
```typescript
// HashMap — built-in Map
const map = new Map<string, number>();
map.set("apple", 1);
map.set("banana", 2);
map.set("apple", 3);         // overwrites

console.log(map.get("apple"));           // 3
console.log(map.has("banana"));          // true
console.log(map.get("cherry") ?? 0);     // 0 (default)
map.delete("banana");
console.log(map.size);                   // 1

// Iterate
for (const [key, val] of map) {
  console.log(`${key}: ${val}`);
}

// Frequency counting
const freq = new Map<string, number>();
for (const ch of "hello") {
  freq.set(ch, (freq.get(ch) ?? 0) + 1);
}

// Plain object as map (string keys only, has prototype pollution risk)
const obj: Record<string, number> = {};
obj["key"] = 1;

// HashSet — built-in Set
const set = new Set<number>();
set.add(1); set.add(2); set.add(1);
console.log(set.size);        // 2
console.log(set.has(1));      // true
set.delete(1);

// Set operations
const a = new Set([1, 2, 3]);
const b = new Set([2, 3, 4]);
const intersection = new Set([...a].filter(x => b.has(x))); // {2,3}
const union        = new Set([...a, ...b]);                  // {1,2,3,4}
const difference   = new Set([...a].filter(x => !b.has(x))); // {1}
```

**Python**
```python
# HashMap — dict (insertion-ordered since Python 3.7)
freq = {}
freq["apple"]  = 1
freq["banana"] = 2
freq["apple"]  = 3   # overwrites

print(freq.get("apple"))          # 3
print(freq.get("cherry", 0))      # 0 (safe default)
print("banana" in freq)           # True
del freq["banana"]                # remove
freq.pop("apple", None)           # remove safely (no KeyError)

# Iteration
for key, val in freq.items():
    print(f"{key}: {val}")

for key in freq:   print(key)
for val in freq.values(): print(val)

# Frequency counting — defaultdict
from collections import defaultdict, Counter
freq = defaultdict(int)
for ch in "hello":
    freq[ch] += 1                 # no KeyError on missing key

freq2 = Counter("hello")          # {'l': 2, 'h': 1, 'e': 1, 'o': 1}
most_common = freq2.most_common(2) # [('l', 2), ('h', 1)]

# OrderedDict (preserves insertion order explicitly)
from collections import OrderedDict
od = OrderedDict()

# HashSet
s = {1, 2, 3}
s.add(4)
s.add(2)       # duplicate — ignored
print(len(s))  # 4
print(2 in s)  # True — O(1)
s.remove(2)    # raises KeyError if missing
s.discard(99)  # safe remove — no error

# Set operations
a = {1, 2, 3}
b = {2, 3, 4}
print(a & b)   # intersection: {2, 3}
print(a | b)   # union:        {1, 2, 3, 4}
print(a - b)   # difference:   {1}
print(a ^ b)   # symmetric diff: {1, 4}
```

### Key HashMap Patterns

```python
# Pattern 1: Frequency count + find duplicates
def contains_duplicate(nums):
    seen = set()
    for n in nums:
        if n in seen: return True
        seen.add(n)
    return False

# Pattern 2: Two Sum (classic)
def two_sum(nums, target):
    seen = {}  # value → index
    for i, n in enumerate(nums):
        complement = target - n
        if complement in seen:
            return [seen[complement], i]
        seen[n] = i
    return []

# Pattern 3: Group anagrams
from collections import defaultdict
def group_anagrams(strs):
    groups = defaultdict(list)
    for s in strs:
        key = tuple(sorted(s))  # "eat" → ('a','e','t')
        groups[key].append(s)
    return list(groups.values())

# Pattern 4: LRU Cache (HashMap + Doubly Linked List)
# HashMap: key → node (O(1) lookup)
# DLL: order by recency (O(1) move to front)
class LRUCache:
    def __init__(self, capacity):
        self.cap = capacity
        self.cache = {}             # key → DNode
        self.head = DNode(0)        # dummy head (most recent end)
        self.tail = DNode(0)        # dummy tail (LRU end)
        self.head.next = self.tail
        self.tail.prev = self.head

    def _remove(self, node):
        node.prev.next = node.next
        node.next.prev = node.prev

    def _insert_front(self, node):  # most recently used
        node.next = self.head.next
        node.prev = self.head
        self.head.next.prev = node
        self.head.next = node

    def get(self, key):
        if key not in self.cache: return -1
        node = self.cache[key]
        self._remove(node)
        self._insert_front(node)    # mark as recently used
        return node.val

    def put(self, key, value):
        if key in self.cache:
            self._remove(self.cache[key])
        node = DNode(value)
        node.key = key
        self.cache[key] = node
        self._insert_front(node)
        if len(self.cache) > self.cap:
            lru = self.tail.prev    # least recently used
            self._remove(lru)
            del self.cache[lru.key]
```

---

## 7. Trees & Binary Search Trees

A tree is a hierarchical data structure. Each node has a value and zero or more children. There are no cycles.

### Tree Terminology

```
Root:      The top node (no parent)
Leaf:      Node with no children
Height:    Longest path from root to leaf
Depth:     Distance from root to a node
Degree:    Number of children a node has

Binary tree:    each node has at most 2 children
Complete tree:  all levels full except possibly last (filled left to right)
Perfect tree:   all internal nodes have 2 children, all leaves at same level
Balanced tree:  height is O(log n) — e.g., AVL, Red-Black
BST:            left < node < right for all nodes
```

### Binary Tree Node & Traversals

**C++**
```cpp
#include <iostream>
#include <queue>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int v) : val(v), left(nullptr), right(nullptr) {}
};

// Inorder: left → root → right  (gives sorted order for BST!)
void inorder(TreeNode* root) {
    if (!root) return;
    inorder(root->left);
    cout << root->val << " ";
    inorder(root->right);
}

// Preorder: root → left → right  (copy a tree, serialize)
void preorder(TreeNode* root) {
    if (!root) return;
    cout << root->val << " ";
    preorder(root->left);
    preorder(root->right);
}

// Postorder: left → right → root  (delete a tree, calculate folder sizes)
void postorder(TreeNode* root) {
    if (!root) return;
    postorder(root->left);
    postorder(root->right);
    cout << root->val << " ";
}

// Level-order (BFS) — queue-based
void levelOrder(TreeNode* root) {
    if (!root) return;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int levelSize = q.size();
        for (int i = 0; i < levelSize; i++) {
            TreeNode* node = q.front(); q.pop();
            cout << node->val << " ";
            if (node->left)  q.push(node->left);
            if (node->right) q.push(node->right);
        }
        cout << endl; // newline between levels
    }
}

// Height of tree
int height(TreeNode* root) {
    if (!root) return 0;
    return 1 + max(height(root->left), height(root->right));
}
```

**Java**
```java
import java.util.*;

public class BinaryTree {
    static class TreeNode {
        int val;
        TreeNode left, right;
        TreeNode(int v) { val = v; }
    }

    // Inorder — recursive
    static void inorder(TreeNode root) {
        if (root == null) return;
        inorder(root.left);
        System.out.print(root.val + " ");
        inorder(root.right);
    }

    // Inorder — iterative (uses explicit stack — avoids call stack overflow)
    static List<Integer> inorderIterative(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        Deque<TreeNode> stack = new ArrayDeque<>();
        TreeNode curr = root;
        while (curr != null || !stack.isEmpty()) {
            while (curr != null) { stack.push(curr); curr = curr.left; }
            curr = stack.pop();
            result.add(curr.val);
            curr = curr.right;
        }
        return result;
    }

    // Level-order BFS
    static List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> result = new ArrayList<>();
        if (root == null) return result;
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        while (!queue.isEmpty()) {
            int size = queue.size();
            List<Integer> level = new ArrayList<>();
            for (int i = 0; i < size; i++) {
                TreeNode node = queue.poll();
                level.add(node.val);
                if (node.left  != null) queue.offer(node.left);
                if (node.right != null) queue.offer(node.right);
            }
            result.add(level);
        }
        return result;
    }

    static int height(TreeNode root) {
        return root == null ? 0 : 1 + Math.max(height(root.left), height(root.right));
    }
}
```

**TypeScript**
```typescript
class TreeNode {
  val: number;
  left: TreeNode | null = null;
  right: TreeNode | null = null;
  constructor(val: number) { this.val = val; }
}

// Traversals
const inorder = (root: TreeNode | null, result: number[] = []): number[] => {
  if (!root) return result;
  inorder(root.left, result);
  result.push(root.val);
  inorder(root.right, result);
  return result;
};

const levelOrder = (root: TreeNode | null): number[][] => {
  if (!root) return [];
  const result: number[][] = [];
  const queue: TreeNode[] = [root];
  while (queue.length) {
    const level: number[] = [];
    const size = queue.length;
    for (let i = 0; i < size; i++) {
      const node = queue.shift()!;
      level.push(node.val);
      if (node.left)  queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
    result.push(level);
  }
  return result;
};

const height = (root: TreeNode | null): number =>
  root ? 1 + Math.max(height(root.left), height(root.right)) : 0;
```

**Python**
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val   = val
        self.left  = left
        self.right = right

# The 4 traversals
def inorder(root):       # left → root → right
    return inorder(root.left) + [root.val] + inorder(root.right) if root else []

def preorder(root):      # root → left → right
    return [root.val] + preorder(root.left) + preorder(root.right) if root else []

def postorder(root):     # left → right → root
    return postorder(root.left) + postorder(root.right) + [root.val] if root else []

def level_order(root):   # BFS — level by level
    if not root: return []
    from collections import deque
    result, queue = [], deque([root])
    while queue:
        level = []
        for _ in range(len(queue)):
            node = queue.popleft()
            level.append(node.val)
            if node.left:  queue.append(node.left)
            if node.right: queue.append(node.right)
        result.append(level)
    return result

def height(root):
    return 1 + max(height(root.left), height(root.right)) if root else 0

def is_balanced(root):
    """Check if tree is height-balanced — O(n)"""
    def check(node):
        if not node: return 0
        left  = check(node.left)
        if left == -1: return -1
        right = check(node.right)
        if right == -1: return -1
        if abs(left - right) > 1: return -1
        return 1 + max(left, right)
    return check(root) != -1
```

### Binary Search Tree (BST)

In a BST: **left subtree values < root < right subtree values** (for all nodes).

```python
class BST:
    def __init__(self):
        self.root = None

    def insert(self, val):        # O(h) — h = height
        self.root = self._insert(self.root, val)

    def _insert(self, node, val):
        if not node: return TreeNode(val)
        if val < node.val:
            node.left = self._insert(node.left, val)
        elif val > node.val:
            node.right = self._insert(node.right, val)
        # if val == node.val: duplicate — ignore (or handle as needed)
        return node

    def search(self, val) -> bool:  # O(h)
        return self._search(self.root, val)

    def _search(self, node, val):
        if not node: return False
        if val == node.val: return True
        if val < node.val: return self._search(node.left, val)
        return self._search(node.right, val)

    def delete(self, val):          # O(h)
        self.root = self._delete(self.root, val)

    def _delete(self, node, val):
        if not node: return None
        if val < node.val:
            node.left = self._delete(node.left, val)
        elif val > node.val:
            node.right = self._delete(node.right, val)
        else:
            # Case 1: leaf node
            if not node.left and not node.right: return None
            # Case 2: one child
            if not node.left:  return node.right
            if not node.right: return node.left
            # Case 3: two children — replace with inorder successor (min of right)
            successor = node.right
            while successor.left:
                successor = successor.left
            node.val = successor.val
            node.right = self._delete(node.right, successor.val)
        return node

    def is_valid_bst(self) -> bool:
        """Validate BST property — O(n)"""
        def validate(node, min_val, max_val):
            if not node: return True
            if node.val <= min_val or node.val >= max_val: return False
            return (validate(node.left, min_val, node.val) and
                    validate(node.right, node.val, max_val))
        return validate(self.root, float('-inf'), float('inf'))

    def kth_smallest(self, k: int) -> int:
        """In-order traversal gives sorted order — find kth"""
        self._count = 0
        self._result = None
        def inorder(node):
            if not node or self._result is not None: return
            inorder(node.left)
            self._count += 1
            if self._count == k:
                self._result = node.val
                return
            inorder(node.right)
        inorder(self.root)
        return self._result
```

```cpp
// C++ BST operations
struct TreeNode { int val; TreeNode *left, *right; TreeNode(int v):val(v),left(nullptr),right(nullptr){} };

TreeNode* insert(TreeNode* root, int val) {
    if (!root) return new TreeNode(val);
    if (val < root->val) root->left  = insert(root->left,  val);
    else if (val > root->val) root->right = insert(root->right, val);
    return root;
}

bool search(TreeNode* root, int val) {
    if (!root) return false;
    if (val == root->val) return true;
    return val < root->val ? search(root->left, val) : search(root->right, val);
}

// Find minimum (leftmost node)
TreeNode* findMin(TreeNode* root) {
    while (root->left) root = root->left;
    return root;
}

TreeNode* deleteNode(TreeNode* root, int val) {
    if (!root) return nullptr;
    if (val < root->val) root->left  = deleteNode(root->left,  val);
    else if (val > root->val) root->right = deleteNode(root->right, val);
    else {
        if (!root->left)  return root->right;
        if (!root->right) return root->left;
        TreeNode* succ = findMin(root->right);
        root->val = succ->val;
        root->right = deleteNode(root->right, succ->val);
    }
    return root;
}
```

### Must-Know Tree Interview Problems

```python
# Lowest Common Ancestor (LCA) — O(n)
def lowest_common_ancestor(root, p, q):
    if not root or root == p or root == q: return root
    left  = lowest_common_ancestor(root.left, p, q)
    right = lowest_common_ancestor(root.right, p, q)
    if left and right: return root   # p and q in different subtrees
    return left or right              # both in same subtree

# Maximum path sum — O(n)
def max_path_sum(root):
    best = [float('-inf')]
    def dfs(node):
        if not node: return 0
        left  = max(dfs(node.left), 0)   # ignore negative gains
        right = max(dfs(node.right), 0)
        # Path through this node
        best[0] = max(best[0], node.val + left + right)
        return node.val + max(left, right)  # return one direction
    dfs(root)
    return best[0]

# Serialize / Deserialize BST — O(n)
def serialize(root) -> str:
    return ','.join(preorder(root))  # preorder preserves BST structure

def deserialize(data: str):
    from collections import deque
    vals = deque(int(v) for v in data.split(',') if v)
    def build(min_v, max_v):
        if not vals or not (min_v < vals[0] < max_v): return None
        val = vals.popleft()
        node = TreeNode(val)
        node.left  = build(min_v, val)
        node.right = build(val, max_v)
        return node
    return build(float('-inf'), float('inf'))
```

---

## 8. Priority Queues & Heaps

A heap is a **complete binary tree** where the parent is always greater (max-heap) or smaller (min-heap) than its children. Used to implement priority queues.

```
Min-Heap:          Max-Heap:
     1                  9
   /   \              /   \
  3     2            7     8
 / \   /            / \
5   4 6            3   4
Parent ≤ children       Parent ≥ children

Stored as array: [1, 3, 2, 5, 4, 6]
Parent of i:   (i-1) // 2
Left child:    2*i + 1
Right child:   2*i + 2
```

### Operations & Complexity

```
insert (push):  Add to end, bubble UP   — O(log n)
extract min/max: Remove root, swap last to root, bubble DOWN — O(log n)
peek:           Look at root            — O(1)
heapify:        Build heap from array  — O(n)  (not O(n log n)!)
```

### Implementation in All 4 Languages

**C++**
```cpp
#include <queue>
#include <vector>
using namespace std;

// Min-heap (default in C++)
priority_queue<int, vector<int>, greater<int>> minHeap;
minHeap.push(5);
minHeap.push(1);
minHeap.push(3);
cout << minHeap.top(); // 1 (minimum)
minHeap.pop();         // removes 1
cout << minHeap.top(); // 3

// Max-heap (default)
priority_queue<int> maxHeap;
maxHeap.push(5);
maxHeap.push(1);
maxHeap.push(3);
cout << maxHeap.top(); // 5 (maximum)

// Custom comparator — sort pairs by second element
priority_queue<pair<int,int>, vector<pair<int,int>>,
               [](auto& a, auto& b){ return a.second > b.second; }> custom;

// Heapify an existing array — O(n)
vector<int> arr = {5, 3, 8, 1, 4};
make_heap(arr.begin(), arr.end());           // max-heap in-place
make_heap(arr.begin(), arr.end(), greater<int>{}); // min-heap in-place
```

**Java**
```java
import java.util.*;

// Min-heap (default Java PriorityQueue)
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
minHeap.offer(5);
minHeap.offer(1);
minHeap.offer(3);
System.out.println(minHeap.peek()); // 1
System.out.println(minHeap.poll()); // 1 — removes

// Max-heap
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
maxHeap.offer(5); maxHeap.offer(1); maxHeap.offer(3);
System.out.println(maxHeap.peek()); // 5

// Custom — sort by second element of int[]
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[1] - b[1]);
pq.offer(new int[]{0, 5});  // [node, distance]
pq.offer(new int[]{1, 2});
System.out.println(pq.peek()[0]); // node 1 (distance 2 is min)

// Heapify from collection — O(n)
PriorityQueue<Integer> fromList = new PriorityQueue<>(Arrays.asList(5, 3, 8, 1, 4));
```

**TypeScript**
```typescript
// JavaScript has no built-in heap — implement one
class MinHeap {
  private heap: number[] = [];

  private parent = (i: number) => Math.floor((i - 1) / 2);
  private left   = (i: number) => 2 * i + 1;
  private right  = (i: number) => 2 * i + 2;
  private swap   = (i: number, j: number) =>
    ([this.heap[i], this.heap[j]] = [this.heap[j], this.heap[i]]);

  push(val: number): void {
    this.heap.push(val);
    this._bubbleUp(this.heap.length - 1);
  }

  pop(): number | undefined {
    if (this.heap.length === 0) return undefined;
    const min = this.heap[0];
    const last = this.heap.pop()!;
    if (this.heap.length > 0) {
      this.heap[0] = last;
      this._siftDown(0);
    }
    return min;
  }

  peek(): number | undefined { return this.heap[0]; }
  size(): number { return this.heap.length; }

  private _bubbleUp(i: number): void {
    while (i > 0) {
      const p = this.parent(i);
      if (this.heap[p] <= this.heap[i]) break;
      this.swap(i, p);
      i = p;
    }
  }

  private _siftDown(i: number): void {
    const n = this.heap.length;
    while (true) {
      let smallest = i;
      const l = this.left(i), r = this.right(i);
      if (l < n && this.heap[l] < this.heap[smallest]) smallest = l;
      if (r < n && this.heap[r] < this.heap[smallest]) smallest = r;
      if (smallest === i) break;
      this.swap(i, smallest);
      i = smallest;
    }
  }
}
```

**Python**
```python
import heapq

# Python's heapq is a MIN-heap
heap = []
heapq.heappush(heap, 5)
heapq.heappush(heap, 1)
heapq.heappush(heap, 3)

print(heap[0])              # 1 — peek (don't remove)
print(heapq.heappop(heap))  # 1 — removes smallest
print(heap[0])              # 3

# Heapify in-place — O(n)
arr = [5, 3, 8, 1, 4]
heapq.heapify(arr)
print(arr[0])               # 1

# Max-heap trick — negate values
max_heap = []
for val in [5, 1, 3]:
    heapq.heappush(max_heap, -val)  # negate
print(-heapq.heappop(max_heap))     # 5 (negate back)

# Push and pop atomically (more efficient)
heapq.heappushpop(heap, 0)    # push then pop (one sift operation)
heapq.heapreplace(heap, 0)    # pop then push (one sift operation)

# n smallest/largest — more efficient than full sort
print(heapq.nsmallest(3, [5,1,3,8,4]))  # [1,3,4]
print(heapq.nlargest(3,  [5,1,3,8,4]))  # [8,5,4]

# Custom comparison — use tuples (compared lexicographically)
heap2 = []
heapq.heappush(heap2, (2, "task B"))  # (priority, value)
heapq.heappush(heap2, (1, "task A"))
heapq.heappush(heap2, (3, "task C"))
print(heapq.heappop(heap2))           # (1, "task A")
```

### Key Heap Patterns

```python
# Pattern 1: Kth Largest Element — O(n log k) time, O(k) space
def kth_largest(nums, k):
    heap = []
    for n in nums:
        heapq.heappush(heap, n)
        if len(heap) > k:
            heapq.heappop(heap)   # keep only k largest
    return heap[0]                # smallest of top k = kth largest

# Pattern 2: Merge K Sorted Lists — O(n log k)
def merge_k_sorted(lists):
    dummy = curr = ListNode(0)
    heap = []
    for i, node in enumerate(lists):
        if node:
            heapq.heappush(heap, (node.val, i, node))
    while heap:
        val, i, node = heapq.heappop(heap)
        curr.next = node
        curr = curr.next
        if node.next:
            heapq.heappush(heap, (node.next.val, i, node.next))
    return dummy.next

# Pattern 3: Find Median from Data Stream
class MedianFinder:
    def __init__(self):
        self.lo = []   # max-heap (store negated) — lower half
        self.hi = []   # min-heap — upper half

    def add_num(self, num):
        heapq.heappush(self.lo, -num)          # push to lower half
        # Balance: move top of lo to hi
        heapq.heappush(self.hi, -heapq.heappop(self.lo))
        # Keep lo at least as large as hi
        if len(self.lo) < len(self.hi):
            heapq.heappush(self.lo, -heapq.heappop(self.hi))

    def find_median(self):
        if len(self.lo) > len(self.hi):
            return -self.lo[0]
        return (-self.lo[0] + self.hi[0]) / 2
```

---

## 9. Graphs

A graph is a set of **vertices** (nodes) connected by **edges**. The most general data structure.

```
Types:
  Directed   — edges have direction (A→B ≠ B→A)  e.g., Twitter follows
  Undirected — edges are bidirectional (A-B = B-A) e.g., Facebook friends
  Weighted   — edges have costs/distances           e.g., road distances
  Cyclic     — contains at least one cycle
  DAG        — Directed Acyclic Graph              e.g., task dependencies
  Connected  — there's a path between any two vertices

Representations:
  Adjacency List  — {node: [neighbors]} — sparse graphs (most common)
  Adjacency Matrix— 2D boolean array   — dense graphs or quick edge lookups
  Edge List       — [(u, v, w), ...]   — simple, used in some algorithms
```

### Graph Representation in All 4 Languages

**C++**
```cpp
#include <vector>
#include <unordered_map>
#include <list>
using namespace std;

// Adjacency list — unweighted undirected
int n = 5; // 5 vertices
vector<vector<int>> adj(n);  // adj[u] = list of neighbors

auto addEdge = [&](int u, int v) {
    adj[u].push_back(v);
    adj[v].push_back(u);  // remove for directed
};

// Adjacency list — weighted directed
vector<vector<pair<int,int>>> wAdj(n); // wAdj[u] = [(v, weight)]
auto addWeightedEdge = [&](int u, int v, int w) {
    wAdj[u].push_back({v, w});
};

// Adjacency matrix — dense graph
vector<vector<bool>> matrix(n, vector<bool>(n, false));
auto addMatrixEdge = [&](int u, int v) {
    matrix[u][v] = matrix[v][u] = true;
};

// Graph with string nodes
unordered_map<string, vector<string>> graph;
graph["A"] = {"B", "C"};
graph["B"] = {"A", "D"};
```

**Java**
```java
import java.util.*;

class Graph {
    int vertices;
    List<List<Integer>> adj;  // adjacency list

    Graph(int v) {
        vertices = v;
        adj = new ArrayList<>();
        for (int i = 0; i < v; i++) adj.add(new ArrayList<>());
    }

    void addEdge(int u, int v) {
        adj.get(u).add(v);
        adj.get(v).add(u); // undirected
    }

    // Weighted graph
    static class WeightedGraph {
        Map<Integer, List<int[]>> adj = new HashMap<>();  // {node: [[neighbor, weight]]}

        void addEdge(int u, int v, int w) {
            adj.computeIfAbsent(u, k -> new ArrayList<>()).add(new int[]{v, w});
            adj.computeIfAbsent(v, k -> new ArrayList<>()).add(new int[]{u, w});
        }
    }
}
```

**TypeScript**
```typescript
// Adjacency list using Map
class Graph {
  private adj = new Map<number, number[]>();

  addVertex(v: number): void {
    if (!this.adj.has(v)) this.adj.set(v, []);
  }

  addEdge(u: number, v: number, directed = false): void {
    this.addVertex(u); this.addVertex(v);
    this.adj.get(u)!.push(v);
    if (!directed) this.adj.get(v)!.push(u);
  }

  getNeighbors(v: number): number[] {
    return this.adj.get(v) ?? [];
  }
}

// Weighted graph with adjacency list
type WeightedAdj = Map<number, { neighbor: number; weight: number }[]>;
```

**Python**
```python
from collections import defaultdict

class Graph:
    def __init__(self):
        self.adj = defaultdict(list)  # {node: [neighbors]}

    def add_edge(self, u, v, directed=False):
        self.adj[u].append(v)
        if not directed:
            self.adj[v].append(u)

    def add_weighted_edge(self, u, v, w, directed=False):
        self.adj[u].append((v, w))
        if not directed:
            self.adj[v].append((u, w))

# Simple functional representation — most used in interviews
graph = {
    0: [1, 2],
    1: [0, 3, 4],
    2: [0, 5],
    3: [1],
    4: [1],
    5: [2],
}

# Weighted — adjacency list of (neighbor, weight) tuples
weighted = {
    0: [(1, 4), (2, 1)],
    1: [(3, 1)],
    2: [(1, 2), (3, 5)],
    3: [],
}
```

### Must-Know Graph Algorithms

```python
# Dijkstra's Shortest Path — O((V + E) log V)
import heapq
def dijkstra(graph, start):
    dist = {node: float('inf') for node in graph}
    dist[start] = 0
    heap = [(0, start)]  # (distance, node)

    while heap:
        d, u = heapq.heappop(heap)
        if d > dist[u]: continue  # stale entry

        for v, w in graph[u]:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                heapq.heappush(heap, (dist[v], v))

    return dist

# Topological Sort (DAG only) — Kahn's Algorithm BFS O(V+E)
def topological_sort(graph, in_degree):
    from collections import deque
    queue = deque([u for u in graph if in_degree[u] == 0])
    order = []

    while queue:
        u = queue.popleft()
        order.append(u)
        for v in graph[u]:
            in_degree[v] -= 1
            if in_degree[v] == 0:
                queue.append(v)

    return order if len(order) == len(graph) else []  # empty if cycle exists

# Union-Find (Disjoint Set) — O(α(n)) per operation (essentially O(1))
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank   = [0] * n

    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])  # path compression
        return self.parent[x]

    def union(self, x, y) -> bool:
        px, py = self.find(x), self.find(y)
        if px == py: return False  # already connected (would form cycle)
        # Union by rank
        if self.rank[px] < self.rank[py]:
            px, py = py, px
        self.parent[py] = px
        if self.rank[px] == self.rank[py]:
            self.rank[px] += 1
        return True

    def connected(self, x, y) -> bool:
        return self.find(x) == self.find(y)
```
---

## 10. Sorting Algorithms

### Big Picture — When to Use Which Sort

| Algorithm | Best | Average | Worst | Space | Stable | Use When |
|---|---|---|---|---|---|---|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | Yes | Never in practice, learn concepts |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | No | Never in practice |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | Yes | Small arrays, nearly sorted data |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes | Stable sort needed, linked lists |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | No | General purpose, cache-friendly |
| Counting Sort | O(n+k) | O(n+k) | O(n+k) | O(k) | Yes | Integers in small range |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | No | In-place & guaranteed O(n log n) |

### Bubble Sort — O(n²)

```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        swapped = False
        for j in range(n - i - 1):   # last i elements are sorted
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
                swapped = True
        if not swapped: break         # O(n) best case — already sorted
    return arr
```

```cpp
void bubbleSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n; i++) {
        bool swapped = false;
        for (int j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j+1]) {
                swap(arr[j], arr[j+1]);
                swapped = true;
            }
        }
        if (!swapped) break;
    }
}
```

### Selection Sort — O(n²)

```python
def selection_sort(arr):
    n = len(arr)
    for i in range(n):
        min_idx = i
        for j in range(i + 1, n):
            if arr[j] < arr[min_idx]:
                min_idx = j
        arr[i], arr[min_idx] = arr[min_idx], arr[i]  # put minimum in position i
    return arr
```

```typescript
function selectionSort(arr: number[]): number[] {
  const n = arr.length;
  for (let i = 0; i < n; i++) {
    let minIdx = i;
    for (let j = i + 1; j < n; j++) {
      if (arr[j] < arr[minIdx]) minIdx = j;
    }
    [arr[i], arr[minIdx]] = [arr[minIdx], arr[i]];
  }
  return arr;
}
```

### Insertion Sort — O(n²) avg, O(n) nearly sorted

```python
def insertion_sort(arr):
    for i in range(1, len(arr)):
        key = arr[i]
        j = i - 1
        while j >= 0 and arr[j] > key:
            arr[j + 1] = arr[j]   # shift right to make room
            j -= 1
        arr[j + 1] = key          # insert key in correct position
    return arr
```

```java
static void insertionSort(int[] arr) {
    for (int i = 1; i < arr.length; i++) {
        int key = arr[i];
        int j = i - 1;
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key;
    }
}
```

### Merge Sort — O(n log n) guaranteed, stable, O(n) space

```python
def merge_sort(arr):
    if len(arr) <= 1: return arr

    mid   = len(arr) // 2
    left  = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    return merge(left, right)

def merge(left, right):
    result = []
    i = j  = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i]); i += 1
        else:
            result.append(right[j]); j += 1
    result.extend(left[i:])
    result.extend(right[j:])
    return result
```

```cpp
void merge(vector<int>& arr, int left, int mid, int right) {
    vector<int> temp(right - left + 1);
    int i = left, j = mid + 1, k = 0;

    while (i <= mid && j <= right) {
        if (arr[i] <= arr[j]) temp[k++] = arr[i++];
        else                   temp[k++] = arr[j++];
    }
    while (i <= mid)   temp[k++] = arr[i++];
    while (j <= right) temp[k++] = arr[j++];
    for (int x = 0; x < k; x++) arr[left + x] = temp[x];
}

void mergeSort(vector<int>& arr, int left, int right) {
    if (left >= right) return;
    int mid = left + (right - left) / 2;
    mergeSort(arr, left, mid);
    mergeSort(arr, mid + 1, right);
    merge(arr, left, mid, right);
}
```

```java
static void mergeSort(int[] arr, int left, int right) {
    if (left >= right) return;
    int mid = left + (right - left) / 2;
    mergeSort(arr, left, mid);
    mergeSort(arr, mid + 1, right);
    merge(arr, left, mid, right);
}

static void merge(int[] arr, int left, int mid, int right) {
    int[] temp = new int[right - left + 1];
    int i = left, j = mid + 1, k = 0;
    while (i <= mid && j <= right)
        temp[k++] = arr[i] <= arr[j] ? arr[i++] : arr[j++];
    while (i <= mid)   temp[k++] = arr[i++];
    while (j <= right) temp[k++] = arr[j++];
    for (int x = 0; x < temp.length; x++) arr[left + x] = temp[x];
}
```

### Quick Sort — O(n log n) avg, O(n²) worst (avoid with random pivot)

```python
import random

def quick_sort(arr, low=0, high=None):
    if high is None: high = len(arr) - 1
    if low < high:
        pivot_idx = partition(arr, low, high)
        quick_sort(arr, low, pivot_idx - 1)
        quick_sort(arr, pivot_idx + 1, high)
    return arr

def partition(arr, low, high):
    # Randomize pivot to avoid O(n²) worst case on sorted input
    rand = random.randint(low, high)
    arr[rand], arr[high] = arr[high], arr[rand]

    pivot = arr[high]
    i = low - 1                 # i = boundary of "less than pivot" region

    for j in range(low, high):
        if arr[j] <= pivot:
            i += 1
            arr[i], arr[j] = arr[j], arr[i]

    arr[i + 1], arr[high] = arr[high], arr[i + 1]  # place pivot
    return i + 1
```

```cpp
int partition(vector<int>& arr, int low, int high) {
    int randIdx = low + rand() % (high - low + 1);
    swap(arr[randIdx], arr[high]);

    int pivot = arr[high];
    int i = low - 1;
    for (int j = low; j < high; j++) {
        if (arr[j] <= pivot) {
            i++;
            swap(arr[i], arr[j]);
        }
    }
    swap(arr[i + 1], arr[high]);
    return i + 1;
}

void quickSort(vector<int>& arr, int low, int high) {
    if (low < high) {
        int pi = partition(arr, low, high);
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
}
```

### Counting Sort — O(n + k), non-comparison based

```python
def counting_sort(arr):
    if not arr: return arr
    max_val = max(arr)
    count   = [0] * (max_val + 1)

    for x in arr:
        count[x] += 1              # count occurrences

    # Prefix sum for stable sort
    for i in range(1, len(count)):
        count[i] += count[i - 1]  # count[i] = # elements <= i

    output = [0] * len(arr)
    for x in reversed(arr):       # reversed for stability
        count[x] -= 1
        output[count[x]] = x

    return output

# Radix sort — sort digit by digit using counting sort
def radix_sort(arr):
    max_val = max(arr)
    exp = 1
    while max_val // exp > 0:
        counting_sort_by_digit(arr, exp)
        exp *= 10
```

---

## 11. Binary Search — Traditional & Condition-Based

Binary search finds a target in a **sorted** array by halving the search space each step. O(log n) — for n=1,000,000,000, that's only 30 comparisons.

### The 3 Templates You Must Know

**Template 1: Classic — exact target**

```python
def binary_search(arr, target):
    left, right = 0, len(arr) - 1

    while left <= right:
        mid = left + (right - left) // 2  # prevents integer overflow vs (l+r)//2

        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1   # target is in right half
        else:
            right = mid - 1  # target is in left half

    return -1  # not found
```

```cpp
int binarySearch(vector<int>& arr, int target) {
    int left = 0, right = arr.size() - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
```

```java
static int binarySearch(int[] arr, int target) {
    int left = 0, right = arr.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
```

```typescript
function binarySearch(arr: number[], target: number): number {
  let left = 0, right = arr.length - 1;
  while (left <= right) {
    const mid = left + Math.floor((right - left) / 2);
    if (arr[mid] === target) return mid;
    else if (arr[mid] < target) left = mid + 1;
    else right = mid - 1;
  }
  return -1;
}
```

**Template 2: Find leftmost (first occurrence / lower bound)**

```python
def lower_bound(arr, target):
    """Returns index of first element >= target"""
    left, right = 0, len(arr)   # right = len(arr) — NOT len-1

    while left < right:          # NOT <=
        mid = (left + right) // 2
        if arr[mid] < target:
            left = mid + 1
        else:
            right = mid          # include mid — might be the answer

    return left  # left == right — the answer

def first_occurrence(arr, target):
    idx = lower_bound(arr, target)
    return idx if idx < len(arr) and arr[idx] == target else -1
```

**Template 3: Find rightmost (last occurrence / upper bound)**

```python
def upper_bound(arr, target):
    """Returns index of first element > target"""
    left, right = 0, len(arr)

    while left < right:
        mid = (left + right) // 2
        if arr[mid] <= target:
            left = mid + 1
        else:
            right = mid

    return left

def last_occurrence(arr, target):
    idx = upper_bound(arr, target) - 1
    return idx if idx >= 0 and arr[idx] == target else -1
```

### Condition-Based Binary Search — The Key Insight

> The real power of binary search: **search on the answer**, not just the array.
>
> Whenever you can ask: "Is X a valid answer?" and the answers form `[False, False, True, True, True]` — binary search on X.

```python
# Classic condition-based template
def condition_binary_search(feasible_fn, lo, hi):
    """Find minimum value x in [lo, hi] for which feasible(x) is True"""
    while lo < hi:
        mid = (lo + hi) // 2
        if feasible_fn(mid):
            hi = mid      # mid might be the answer, search left
        else:
            lo = mid + 1  # mid is not feasible, search right
    return lo             # lo == hi == answer
```

**Example 1: Minimum days to make M bouquets**

```python
def min_days(bloom_day, m, k):
    """
    Can we make m bouquets of k adjacent flowers, all bloomed by day d?
    """
    def feasible(d):
        bouquets = consecutive = 0
        for day in bloom_day:
            if day <= d:
                consecutive += 1
                if consecutive == k:
                    bouquets += 1
                    consecutive = 0
            else:
                consecutive = 0
        return bouquets >= m

    if m * k > len(bloom_day): return -1
    return condition_binary_search(feasible, min(bloom_day), max(bloom_day))
```

**Example 2: Koko Eating Bananas — find minimum speed**

```python
def min_eating_speed(piles, h):
    """Find minimum k (bananas/hour) to eat all piles in h hours"""
    import math
    def feasible(k):
        return sum(math.ceil(p / k) for p in piles) <= h

    return condition_binary_search(feasible, 1, max(piles))
```

**Example 3: Binary search on rotated sorted array**

```python
def search_rotated(nums, target):
    left, right = 0, len(nums) - 1
    while left <= right:
        mid = (left + right) // 2
        if nums[mid] == target: return mid

        # Determine which half is sorted
        if nums[left] <= nums[mid]:          # left half is sorted
            if nums[left] <= target < nums[mid]:
                right = mid - 1
            else:
                left = mid + 1
        else:                                # right half is sorted
            if nums[mid] < target <= nums[right]:
                left = mid + 1
            else:
                right = mid - 1

    return -1
```

---

## 12. Recursion

Recursion solves a problem by breaking it into **smaller instances of the same problem**. Every recursive function needs a **base case** (stop condition) and a **recursive case**.

### The Recursion Mental Model

```
Think of recursion as:
  1. What is the smallest input I know the answer to? (Base case)
  2. If I had the answer to a smaller problem, how do I build the full answer? (Recursive case)
  3. Trust the recursive call — don't trace the entire call stack mentally.
```

### Examples in All 4 Languages

**Factorial**

```python
# Python
def factorial(n):
    if n <= 1: return 1          # base case
    return n * factorial(n - 1)  # recursive case
# n=5 → 5 * factorial(4) → 5 * 4 * 3 * 2 * 1 = 120
```

```cpp
// C++
long long factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}
```

```java
// Java
static long factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}
```

```typescript
// TypeScript
function factorial(n: number): number {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}
```

**Fibonacci — naive O(2ⁿ) vs memoized O(n)**

```python
# Naive — exponential (DON'T use)
def fib_naive(n):
    if n <= 1: return n
    return fib_naive(n-1) + fib_naive(n-2)  # recalculates same subproblems!

# Memoized — O(n) time, O(n) space
def fib_memo(n, memo={}):
    if n in memo: return memo[n]
    if n <= 1: return n
    memo[n] = fib_memo(n-1, memo) + fib_memo(n-2, memo)
    return memo[n]

# Bottom-up DP — O(n) time, O(1) space (best)
def fib_dp(n):
    if n <= 1: return n
    a, b = 0, 1
    for _ in range(n - 1):
        a, b = b, a + b
    return b
```

**Tower of Hanoi — O(2ⁿ) moves required (mathematically minimal)**

```python
def hanoi(n, from_rod, to_rod, aux_rod):
    if n == 1:
        print(f"Move disk 1: {from_rod} → {to_rod}")
        return
    hanoi(n-1, from_rod, aux_rod, to_rod)   # move n-1 disks to aux
    print(f"Move disk {n}: {from_rod} → {to_rod}")  # move biggest
    hanoi(n-1, aux_rod, to_rod, from_rod)   # move n-1 disks from aux to dest
```

**Power function — O(log n) via repeated squaring**

```python
def power(base, exp):
    if exp == 0: return 1
    if exp % 2 == 0:
        half = power(base, exp // 2)
        return half * half             # avoid redundant recursive call
    return base * power(base, exp - 1)

# O(log n) — much better than O(n) naive multiplication
```

**Recursion on data structures**

```python
# Flatten nested list — elegant recursion
def flatten(arr):
    result = []
    for item in arr:
        if isinstance(item, list):
            result.extend(flatten(item))
        else:
            result.append(item)
    return result
# flatten([1, [2, [3, 4]], 5]) → [1, 2, 3, 4, 5]

# Count paths in grid from top-left to bottom-right (right/down only)
def count_paths(m, n):
    if m == 1 or n == 1: return 1
    return count_paths(m-1, n) + count_paths(m, n-1)
```

### Recursion vs Iteration — Tail Call & Stack Overflow

```python
# Recursive factorial → stack frame for each call
# n=10000 → RecursionError in Python (default limit 1000)

import sys
sys.setrecursionlimit(10000)  # increase limit

# Convert to iterative when recursion depth is large
def factorial_iterative(n):
    result = 1
    while n > 1:
        result *= n
        n -= 1
    return result

# Or use tail-call style (Python doesn't optimize this, but conceptually clean)
def factorial_tail(n, acc=1):
    if n <= 1: return acc
    return factorial_tail(n - 1, n * acc)
```

---

## 13. Recursive Backtracking

Backtracking is a form of recursion where you **explore options, commit, recurse, then undo (backtrack)** if the path doesn't lead to a solution.

```
Pattern:
  def backtrack(state, choices):
      if is_solution(state):
          record_solution(state)
          return
      for choice in choices:
          if is_valid(state, choice):
              make_choice(state, choice)    # commit
              backtrack(state, next_choices) # recurse
              undo_choice(state, choice)    # backtrack (undo)
```

### Subsets / Power Set — O(n × 2ⁿ)

```python
def subsets(nums):
    result = []
    def backtrack(start, current):
        result.append(current[:])    # record current subset
        for i in range(start, len(nums)):
            current.append(nums[i])  # choose
            backtrack(i + 1, current)
            current.pop()            # undo
    backtrack(0, [])
    return result
# subsets([1,2,3]) → [[], [1], [1,2], [1,2,3], [1,3], [2], [2,3], [3]]
```

```typescript
function subsets(nums: number[]): number[][] {
  const result: number[][] = [];
  function backtrack(start: number, current: number[]): void {
    result.push([...current]);
    for (let i = start; i < nums.length; i++) {
      current.push(nums[i]);
      backtrack(i + 1, current);
      current.pop();
    }
  }
  backtrack(0, []);
  return result;
}
```

### Permutations — O(n × n!)

```python
def permutations(nums):
    result = []
    def backtrack(current, remaining):
        if not remaining:
            result.append(current[:])
            return
        for i, num in enumerate(remaining):
            current.append(num)
            backtrack(current, remaining[:i] + remaining[i+1:])
            current.pop()
    backtrack([], nums)
    return result

# More efficient — swap in-place
def permutations_inplace(nums):
    result = []
    def backtrack(start):
        if start == len(nums):
            result.append(nums[:])
            return
        for i in range(start, len(nums)):
            nums[start], nums[i] = nums[i], nums[start]   # swap
            backtrack(start + 1)
            nums[start], nums[i] = nums[i], nums[start]   # swap back
    backtrack(0)
    return result
```

```cpp
void permute(vector<int>& nums, int start, vector<vector<int>>& result) {
    if (start == nums.size()) { result.push_back(nums); return; }
    for (int i = start; i < nums.size(); i++) {
        swap(nums[start], nums[i]);
        permute(nums, start + 1, result);
        swap(nums[start], nums[i]);  // backtrack
    }
}
```

### Combinations — O(n! / (k!(n-k)!))

```python
def combinations(n, k):
    result = []
    def backtrack(start, current):
        if len(current) == k:
            result.append(current[:])
            return
        # Pruning: if remaining elements can't complete k, stop
        remaining_needed = k - len(current)
        remaining_available = n - start + 1
        if remaining_available < remaining_needed: return

        for i in range(start, n + 1):
            current.append(i)
            backtrack(i + 1, current)
            current.pop()
    backtrack(1, [])
    return result
```

### N-Queens — Place N queens so none attacks another

```python
def solve_n_queens(n):
    result = []
    cols = set()       # columns with queens
    diag1 = set()      # (row - col) constant on ↗ diagonal
    diag2 = set()      # (row + col) constant on ↘ diagonal

    def backtrack(row, board):
        if row == n:
            result.append(["".join(r) for r in board])
            return
        for col in range(n):
            if col in cols or (row - col) in diag1 or (row + col) in diag2:
                continue                              # attacked — try next column
            # Place queen
            cols.add(col); diag1.add(row - col); diag2.add(row + col)
            board[row][col] = 'Q'
            backtrack(row + 1, board)
            # Remove queen (backtrack)
            board[row][col] = '.'
            cols.remove(col); diag1.remove(row - col); diag2.remove(row + col)

    board = [['.' for _ in range(n)] for _ in range(n)]
    backtrack(0, board)
    return result
```

### Sudoku Solver

```python
def solve_sudoku(board):
    def is_valid(r, c, num):
        # Check row, column, 3×3 box
        box_r, box_c = 3 * (r // 3), 3 * (c // 3)
        for i in range(9):
            if board[r][i] == num: return False
            if board[i][c] == num: return False
            if board[box_r + i//3][box_c + i%3] == num: return False
        return True

    def backtrack():
        for r in range(9):
            for c in range(9):
                if board[r][c] == '.':
                    for num in '123456789':
                        if is_valid(r, c, num):
                            board[r][c] = num
                            if backtrack(): return True
                            board[r][c] = '.'   # backtrack
                    return False  # no valid number found
        return True  # all cells filled

    backtrack()
```

### Word Search in Grid

```python
def word_search(board, word):
    rows, cols = len(board), len(board[0])

    def backtrack(r, c, idx):
        if idx == len(word): return True  # found!
        if r < 0 or r >= rows or c < 0 or c >= cols: return False
        if board[r][c] != word[idx]: return False

        temp = board[r][c]
        board[r][c] = '#'    # mark as visited (no extra visited set needed!)

        found = (backtrack(r+1, c, idx+1) or backtrack(r-1, c, idx+1) or
                 backtrack(r, c+1, idx+1) or backtrack(r, c-1, idx+1))

        board[r][c] = temp   # restore (backtrack)
        return found

    for r in range(rows):
        for c in range(cols):
            if backtrack(r, c, 0): return True
    return False
```

---

## 14. Depth-First Search (DFS)

DFS explores as **deep as possible** before backtracking. Uses a stack (implicit via recursion, or explicit).

### DFS on Graphs

```python
# Recursive DFS — O(V + E)
def dfs_recursive(graph, start, visited=None):
    if visited is None: visited = set()
    visited.add(start)
    print(start, end=" ")
    for neighbor in graph[start]:
        if neighbor not in visited:
            dfs_recursive(graph, neighbor, visited)
    return visited

# Iterative DFS — explicit stack (avoids recursion limit)
def dfs_iterative(graph, start):
    visited = set()
    stack = [start]
    order = []
    while stack:
        node = stack.pop()
        if node in visited: continue
        visited.add(node)
        order.append(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                stack.append(neighbor)
    return order
```

```cpp
// C++ DFS — graph as adjacency list
void dfs(int node, vector<vector<int>>& adj, vector<bool>& visited) {
    visited[node] = true;
    cout << node << " ";
    for (int neighbor : adj[node]) {
        if (!visited[neighbor])
            dfs(neighbor, adj, visited);
    }
}

// Iterative DFS
void dfsIterative(int start, vector<vector<int>>& adj) {
    int n = adj.size();
    vector<bool> visited(n, false);
    stack<int> stk;
    stk.push(start);
    while (!stk.empty()) {
        int node = stk.top(); stk.pop();
        if (visited[node]) continue;
        visited[node] = true;
        cout << node << " ";
        for (int neighbor : adj[node])
            if (!visited[neighbor]) stk.push(neighbor);
    }
}
```

```java
// Java DFS
static void dfs(int node, List<List<Integer>> adj, boolean[] visited) {
    visited[node] = true;
    System.out.print(node + " ");
    for (int neighbor : adj.get(node)) {
        if (!visited[neighbor]) dfs(neighbor, adj, visited);
    }
}
```

```typescript
function dfs(graph: Map<number, number[]>, start: number): number[] {
  const visited = new Set<number>();
  const order: number[] = [];

  function explore(node: number): void {
    if (visited.has(node)) return;
    visited.add(node);
    order.push(node);
    for (const neighbor of graph.get(node) ?? []) {
      explore(neighbor);
    }
  }

  explore(start);
  return order;
}
```

### DFS on Trees — Classic Problems

```python
# Path sum — does any root-to-leaf path sum to target?
def has_path_sum(root, target):
    if not root: return False
    if not root.left and not root.right: return root.val == target  # leaf
    return (has_path_sum(root.left,  target - root.val) or
            has_path_sum(root.right, target - root.val))

# Diameter of binary tree — longest path between any two nodes
def diameter_of_tree(root):
    diameter = [0]
    def dfs(node):
        if not node: return 0
        left  = dfs(node.left)
        right = dfs(node.right)
        diameter[0] = max(diameter[0], left + right)  # path through this node
        return 1 + max(left, right)                   # return height
    dfs(root)
    return diameter[0]

# Number of islands — DFS to flood-fill each island
def num_islands(grid):
    if not grid: return 0
    rows, cols = len(grid), len(grid[0])
    count = 0

    def dfs(r, c):
        if r < 0 or r >= rows or c < 0 or c >= cols or grid[r][c] != '1':
            return
        grid[r][c] = '#'  # mark visited by changing the cell
        dfs(r+1, c); dfs(r-1, c); dfs(r, c+1); dfs(r, c-1)

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '1':
                dfs(r, c)
                count += 1
    return count

# Clone a graph — DFS with a visited map
def clone_graph(node):
    if not node: return None
    cloned = {}  # original → clone

    def dfs(n):
        if n in cloned: return cloned[n]
        clone = GraphNode(n.val)
        cloned[n] = clone
        for neighbor in n.neighbors:
            clone.neighbors.append(dfs(neighbor))
        return clone

    return dfs(node)
```

```cpp
// Number of islands — C++
class Solution {
    void dfs(vector<vector<char>>& grid, int r, int c) {
        int rows = grid.size(), cols = grid[0].size();
        if (r < 0 || r >= rows || c < 0 || c >= cols || grid[r][c] != '1') return;
        grid[r][c] = '#';
        dfs(grid, r+1, c); dfs(grid, r-1, c);
        dfs(grid, r, c+1); dfs(grid, r, c-1);
    }
public:
    int numIslands(vector<vector<char>>& grid) {
        int count = 0;
        for (int r = 0; r < grid.size(); r++)
            for (int c = 0; c < grid[0].size(); c++)
                if (grid[r][c] == '1') { dfs(grid, r, c); count++; }
        return count;
    }
};
```

---

## 15. Breadth-First Search (BFS)

BFS explores all neighbors at the current depth before going deeper. Uses a **queue**. Always finds the **shortest path** in unweighted graphs.

```
BFS key insight: "process level by level"
DFS key insight: "go as deep as possible first"

Use BFS when:
  - Shortest path in unweighted graph/grid
  - Level-by-level tree traversal
  - Finding minimum number of steps/moves
```

### BFS Templates

```python
from collections import deque

# Graph BFS — O(V + E)
def bfs(graph, start, end=None):
    visited = {start}
    queue   = deque([start])
    distance = {start: 0}

    while queue:
        node = queue.popleft()
        if node == end: return distance[node]  # shortest path found

        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                distance[neighbor] = distance[node] + 1
                queue.append(neighbor)

    return distance  # distances from start to all reachable nodes

# Grid BFS — shortest path on a 2D grid
def shortest_path_grid(grid, start, end):
    rows, cols = len(grid), len(grid[0])
    visited = set([start])
    queue = deque([(start[0], start[1], 0)])  # (row, col, steps)
    directions = [(0,1),(0,-1),(1,0),(-1,0)]

    while queue:
        r, c, steps = queue.popleft()
        if (r, c) == end: return steps

        for dr, dc in directions:
            nr, nc = r + dr, c + dc
            if (0 <= nr < rows and 0 <= nc < cols and
                grid[nr][nc] != '#' and (nr, nc) not in visited):
                visited.add((nr, nc))
                queue.append((nr, nc, steps + 1))

    return -1  # no path

# Multi-source BFS — start from multiple sources simultaneously
def multi_source_bfs(grid, sources):
    """Find distance from any source to every cell"""
    rows, cols = len(grid), len(grid[0])
    dist = [[float('inf')] * cols for _ in range(rows)]
    queue = deque()

    for r, c in sources:
        dist[r][c] = 0
        queue.append((r, c))

    while queue:
        r, c = queue.popleft()
        for dr, dc in [(0,1),(0,-1),(1,0),(-1,0)]:
            nr, nc = r + dr, c + dc
            if 0 <= nr < rows and 0 <= nc < cols and dist[nr][nc] == float('inf'):
                dist[nr][nc] = dist[r][c] + 1
                queue.append((nr, nc))

    return dist
```

```cpp
// BFS — C++
#include <queue>
vector<int> bfs(vector<vector<int>>& adj, int start, int n) {
    vector<int> dist(n, -1);
    queue<int> q;
    dist[start] = 0;
    q.push(start);
    while (!q.empty()) {
        int node = q.front(); q.pop();
        for (int neighbor : adj[node]) {
            if (dist[neighbor] == -1) {
                dist[neighbor] = dist[node] + 1;
                q.push(neighbor);
            }
        }
    }
    return dist;
}
```

```java
// BFS — Java
static int[] bfs(List<List<Integer>> adj, int start, int n) {
    int[] dist = new int[n];
    Arrays.fill(dist, -1);
    Queue<Integer> queue = new LinkedList<>();
    dist[start] = 0;
    queue.offer(start);
    while (!queue.isEmpty()) {
        int node = queue.poll();
        for (int neighbor : adj.get(node)) {
            if (dist[neighbor] == -1) {
                dist[neighbor] = dist[node] + 1;
                queue.offer(neighbor);
            }
        }
    }
    return dist;
}
```

```typescript
function bfs(graph: Map<number, number[]>, start: number): Map<number, number> {
  const dist = new Map<number, number>();
  const queue: number[] = [start];
  dist.set(start, 0);

  while (queue.length) {
    const node = queue.shift()!;
    for (const neighbor of graph.get(node) ?? []) {
      if (!dist.has(neighbor)) {
        dist.set(neighbor, dist.get(node)! + 1);
        queue.push(neighbor);
      }
    }
  }
  return dist;
}
```

### Classic BFS Interview Problems

```python
# Word Ladder — minimum transformations from beginWord to endWord
from collections import deque

def word_ladder(begin_word, end_word, word_list):
    word_set = set(word_list)
    if end_word not in word_set: return 0

    queue = deque([(begin_word, 1)])  # (word, steps)
    visited = {begin_word}

    while queue:
        word, steps = queue.popleft()
        for i in range(len(word)):
            for c in 'abcdefghijklmnopqrstuvwxyz':
                new_word = word[:i] + c + word[i+1:]
                if new_word == end_word: return steps + 1
                if new_word in word_set and new_word not in visited:
                    visited.add(new_word)
                    queue.append((new_word, steps + 1))

    return 0  # unreachable

# 0/1 Matrix — distance of each cell from nearest 0 (multi-source BFS)
def update_matrix(mat):
    rows, cols = len(mat), len(mat[0])
    dist = [[float('inf')] * cols for _ in range(rows)]
    queue = deque()

    for r in range(rows):
        for c in range(cols):
            if mat[r][c] == 0:
                dist[r][c] = 0
                queue.append((r, c))

    for dr, dc in [(0,1),(0,-1),(1,0),(-1,0)]:
        while queue:
            r, c = queue.popleft()
            nr, nc = r + dr, c + dc
            if 0 <= nr < rows and 0 <= nc < cols and dist[nr][nc] > dist[r][c] + 1:
                dist[nr][nc] = dist[r][c] + 1
                queue.append((nr, nc))

    return dist
```

---

## 16. Two Pointers & Sliding Window

### Two Pointers — O(n) technique replacing O(n²) nested loops

**Pattern 1: Opposite ends (sorted array)**

```python
def two_sum_sorted(arr, target):
    """Find pair that sums to target in sorted array — O(n)"""
    left, right = 0, len(arr) - 1
    while left < right:
        total = arr[left] + arr[right]
        if total == target:
            return [left, right]
        elif total < target:
            left += 1    # need bigger sum
        else:
            right -= 1   # need smaller sum
    return []

def three_sum(nums):
    """Find all triplets that sum to 0 — O(n²)"""
    nums.sort()
    result = []
    for i in range(len(nums) - 2):
        if i > 0 and nums[i] == nums[i-1]: continue  # skip duplicates
        left, right = i + 1, len(nums) - 1
        while left < right:
            total = nums[i] + nums[left] + nums[right]
            if total == 0:
                result.append([nums[i], nums[left], nums[right]])
                while left < right and nums[left] == nums[left+1]: left += 1
                while left < right and nums[right] == nums[right-1]: right -= 1
                left += 1; right -= 1
            elif total < 0: left += 1
            else: right -= 1
    return result

def container_with_most_water(height):
    """Max area between two vertical lines — O(n)"""
    left, right = 0, len(height) - 1
    max_area = 0
    while left < right:
        area = min(height[left], height[right]) * (right - left)
        max_area = max(max_area, area)
        if height[left] < height[right]: left += 1
        else: right -= 1
    return max_area
```

```cpp
// Two Sum Sorted — C++
vector<int> twoSumSorted(vector<int>& arr, int target) {
    int left = 0, right = arr.size() - 1;
    while (left < right) {
        int sum = arr[left] + arr[right];
        if (sum == target) return {left, right};
        else if (sum < target) left++;
        else right--;
    }
    return {};
}
```

**Pattern 2: Same direction (fast/slow pointers)**

```python
def remove_duplicates_sorted(arr):
    """Remove duplicates in-place from sorted array — O(n)"""
    if not arr: return 0
    slow = 0
    for fast in range(1, len(arr)):
        if arr[fast] != arr[slow]:
            slow += 1
            arr[slow] = arr[fast]
    return slow + 1  # new length

def move_zeros(arr):
    """Move all zeros to end maintaining order — O(n)"""
    slow = 0
    for fast in range(len(arr)):
        if arr[fast] != 0:
            arr[slow], arr[fast] = arr[fast], arr[slow]
            slow += 1
```

### Sliding Window — Fixed & Variable Size

**Fixed Size Window — O(n)**

```python
def max_sum_subarray_k(arr, k):
    """Maximum sum of any k consecutive elements"""
    window_sum = sum(arr[:k])
    max_sum    = window_sum

    for i in range(k, len(arr)):
        window_sum += arr[i] - arr[i - k]  # add new, remove old
        max_sum = max(max_sum, window_sum)

    return max_sum

def find_anagrams(s, p):
    """Find all start indices of p's anagrams in s"""
    from collections import Counter
    result = []
    p_count = Counter(p)
    w_count = Counter(s[:len(p)])
    if w_count == p_count: result.append(0)

    for i in range(len(p), len(s)):
        # Slide: add new char, remove old char
        w_count[s[i]] += 1
        w_count[s[i - len(p)]] -= 1
        if w_count[s[i - len(p)]] == 0:
            del w_count[s[i - len(p)]]
        if w_count == p_count:
            result.append(i - len(p) + 1)

    return result
```

**Variable Size Window — Shrink when invalid**

```python
def longest_substring_no_repeat(s):
    """Longest substring without repeating characters — O(n)"""
    char_idx = {}   # char → last seen index
    max_len  = 0
    left     = 0

    for right, ch in enumerate(s):
        if ch in char_idx and char_idx[ch] >= left:
            left = char_idx[ch] + 1   # shrink: move left past last occurrence
        char_idx[ch] = right
        max_len = max(max_len, right - left + 1)

    return max_len

def min_subarray_sum(target, nums):
    """Minimum length subarray with sum >= target — O(n)"""
    min_len  = float('inf')
    left     = 0
    curr_sum = 0

    for right in range(len(nums)):
        curr_sum += nums[right]
        while curr_sum >= target:          # window is valid — try to shrink
            min_len = min(min_len, right - left + 1)
            curr_sum -= nums[left]
            left += 1

    return min_len if min_len != float('inf') else 0

def longest_substring_k_distinct(s, k):
    """Longest substring with at most k distinct characters — O(n)"""
    from collections import defaultdict
    count    = defaultdict(int)
    max_len  = 0
    left     = 0

    for right, ch in enumerate(s):
        count[ch] += 1
        while len(count) > k:              # too many distinct — shrink
            count[s[left]] -= 1
            if count[s[left]] == 0:
                del count[s[left]]
            left += 1
        max_len = max(max_len, right - left + 1)

    return max_len
```

```typescript
// Longest substring no repeat — TypeScript
function lengthOfLongestSubstring(s: string): number {
  const charIdx = new Map<string, number>();
  let maxLen = 0, left = 0;

  for (let right = 0; right < s.length; right++) {
    const ch = s[right];
    if (charIdx.has(ch) && charIdx.get(ch)! >= left) {
      left = charIdx.get(ch)! + 1;
    }
    charIdx.set(ch, right);
    maxLen = Math.max(maxLen, right - left + 1);
  }
  return maxLen;
}
```

---

## 17. Binary Numbers & Bit Manipulation

### Binary Fundamentals

```
Decimal: 13 = 1×8 + 1×4 + 0×2 + 1×1 = 1101 in binary

Bit operations:
  AND  (&):  1 & 1 = 1,  1 & 0 = 0,  0 & 0 = 0
  OR   (|):  1 | 1 = 1,  1 | 0 = 1,  0 | 0 = 0
  XOR  (^):  1 ^ 1 = 0,  1 ^ 0 = 1,  0 ^ 0 = 0  (same=0, diff=1)
  NOT  (~):  ~0 = 1,  ~1 = 0  (inverts all bits)
  Left shift  (<<): x << n = x × 2ⁿ  (shift bits left, fill 0s on right)
  Right shift (>>): x >> n = x ÷ 2ⁿ  (shift bits right)
```

### Essential Bit Tricks

```python
# Check if nth bit is set (bit counting from 0, rightmost)
def is_bit_set(n, bit): return (n >> bit) & 1 == 1
# is_bit_set(13, 3) → 13 = 1101, bit 3 = 1 → True
# is_bit_set(13, 1) → bit 1 = 0 → False

# Set nth bit (turn it on)
def set_bit(n, bit): return n | (1 << bit)
# set_bit(5, 1) → 0101 | 0010 = 0111 = 7

# Clear nth bit (turn it off)
def clear_bit(n, bit): return n & ~(1 << bit)
# clear_bit(7, 1) → 0111 & 1101 = 0101 = 5

# Toggle nth bit
def toggle_bit(n, bit): return n ^ (1 << bit)
# toggle_bit(5, 1) → 0101 ^ 0010 = 0111 = 7

# Check if power of 2 — only one bit is set
def is_power_of_two(n): return n > 0 and (n & (n - 1)) == 0
# 8 = 1000, 8-1 = 0111, 1000 & 0111 = 0000 ✓
# 6 = 0110, 6-1 = 0101, 0110 & 0101 = 0100 ✗

# Count set bits (Hamming weight) — Brian Kernighan's algorithm
def count_bits(n):
    count = 0
    while n:
        n &= (n - 1)  # removes lowest set bit
        count += 1
    return count
# 13 = 1101: 1101 & 1100 = 1100, 1100 & 1011 = 1000, 1000 & 0111 = 0000 → 3 bits

# Get lowest set bit
def lowest_set_bit(n): return n & (-n)
# 12 = 1100, -12 = 0100 (two's complement), 1100 & 0100 = 0100 = 4

# XOR — finds the single unique number in array where all others appear twice
def single_number(nums):
    result = 0
    for n in nums: result ^= n
    return result
# [4,1,2,1,2] → 4^1^2^1^2 = 4^(1^1)^(2^2) = 4^0^0 = 4

# Swap without temp variable (XOR swap)
def swap(a, b):
    a ^= b
    b ^= a
    a ^= b
    return a, b

# Extract bits from range [i, j]
def extract_bits(n, i, j):
    mask = ((1 << (j - i + 1)) - 1) << i
    return (n & mask) >> i
```

```cpp
// C++ bit tricks
#include <bit>  // C++20

// Count set bits (popcount)
int popcount(unsigned int n) { return __builtin_popcount(n); }  // GCC built-in

// Count leading zeros
int clz(unsigned int n) { return __builtin_clz(n); }

// Count trailing zeros
int ctz(unsigned int n) { return __builtin_ctz(n); }

// All bit tricks work the same as Python above — just different syntax
bool isPowerOfTwo(int n) { return n > 0 && (n & (n - 1)) == 0; }
int  singleNumber(vector<int>& nums) {
    int res = 0;
    for (int n : nums) res ^= n;
    return res;
}
```

```java
// Java
int popcount = Integer.bitCount(n);           // count set bits
int highestOneBit = Integer.highestOneBit(n); // highest set bit
int numberOfLeadingZeros = Integer.numberOfLeadingZeros(n);
boolean isPowerOfTwo = n > 0 && (n & (n - 1)) == 0;
int singleNumber(int[] nums) {
    int res = 0;
    for (int n : nums) res ^= n;
    return res;
}
```

```typescript
// TypeScript
function countBits(n: number): number {
  let count = 0;
  while (n) { n &= n - 1; count++; }
  return count;
}
function isPowerOfTwo(n: number): boolean { return n > 0 && (n & (n-1)) === 0; }
function singleNumber(nums: number[]): number {
  return nums.reduce((a, b) => a ^ b, 0);
}
```

### Bitmask DP — Represent subsets as integers

```python
# Each bit i represents whether element i is in the subset
# {0,1,2} → bit 0, bit 1, bit 2 → integer 0b111 = 7

def count_subsets_with_xor(nums, target):
    """Count subsets with XOR equal to target — O(n × max_val)"""
    dp = {0: 1}  # {xor_value: count}
    for num in nums:
        new_dp = {}
        for xor, count in dp.items():
            new_xor = xor ^ num
            new_dp[new_xor] = new_dp.get(new_xor, 0) + count
            new_dp[xor]     = new_dp.get(xor, 0) + count
        dp = new_dp
    return dp.get(target, 0)

# Traveling Salesman with bitmask DP — O(2ⁿ × n²)
def tsp(dist, n):
    INF = float('inf')
    # dp[mask][i] = min cost to visit all cities in mask, ending at i
    dp = [[INF] * n for _ in range(1 << n)]
    dp[1][0] = 0  # start at city 0, visited = {0} = bitmask 1

    for mask in range(1 << n):
        for u in range(n):
            if dp[mask][u] == INF: continue
            if not (mask >> u & 1): continue  # u not in mask
            for v in range(n):
                if mask >> v & 1: continue     # v already visited
                new_mask = mask | (1 << v)
                dp[new_mask][v] = min(dp[new_mask][v], dp[mask][u] + dist[u][v])

    full_mask = (1 << n) - 1
    return min(dp[full_mask][i] + dist[i][0] for i in range(n))
```

---

## 18. Greedy Algorithms

A greedy algorithm makes the **locally optimal choice at each step** hoping it leads to a globally optimal solution.

> **Greedy works when:** local optimum = global optimum. Proven with an exchange argument ("swapping any non-greedy choice for the greedy choice never makes things worse").

### Classic Greedy Problems

**Activity Selection / Interval Scheduling**

```python
def max_non_overlapping_intervals(intervals):
    """
    Select maximum number of non-overlapping intervals.
    Greedy: always pick interval with earliest END time.
    """
    intervals.sort(key=lambda x: x[1])  # sort by end time
    count   = 0
    last_end = float('-inf')

    for start, end in intervals:
        if start >= last_end:      # doesn't overlap with last selected
            count += 1
            last_end = end

    return count

# Variant: minimum intervals to remove to make all non-overlapping
def erase_overlap_intervals(intervals):
    intervals.sort(key=lambda x: x[1])
    removed  = 0
    last_end = float('-inf')
    for start, end in intervals:
        if start >= last_end:
            last_end = end
        else:
            removed += 1           # remove current (keep earlier-ending one)
    return removed
```

**Jump Game**

```python
def can_jump(nums):
    """Can you reach the last index? Greedy O(n)"""
    max_reach = 0
    for i, jump in enumerate(nums):
        if i > max_reach: return False  # stranded — can't reach here
        max_reach = max(max_reach, i + jump)
    return True

def jump_game_ii(nums):
    """Minimum number of jumps to reach last index — O(n)"""
    jumps = 0
    current_end = farthest = 0
    for i in range(len(nums) - 1):
        farthest = max(farthest, i + nums[i])
        if i == current_end:        # exhausted current jump range
            jumps += 1
            current_end = farthest
    return jumps
```

**Gas Station — Circular Route**

```python
def can_complete_circuit(gas, cost):
    """
    Find starting gas station to complete circular route.
    Greedy insight: if total gas >= total cost, a solution always exists.
    The valid start is the station after the last deficit.
    """
    total_tank = current_tank = start = 0
    for i in range(len(gas)):
        diff = gas[i] - cost[i]
        total_tank   += diff
        current_tank += diff
        if current_tank < 0:   # can't reach next station from start
            start        = i + 1
            current_tank = 0
    return start if total_tank >= 0 else -1
```

**Huffman Encoding (Greedy + Min-Heap)**

```python
import heapq

def huffman_encoding(frequencies):
    """
    Build optimal prefix-free code. Greedy: always merge two lowest-frequency nodes.
    """
    heap = [[freq, [char, ""]] for char, freq in frequencies.items()]
    heapq.heapify(heap)

    while len(heap) > 1:
        lo = heapq.heappop(heap)  # lowest frequency
        hi = heapq.heappop(heap)  # second lowest

        for pair in lo[1:]:  pair[1] = '0' + pair[1]  # prepend 0 to lo's codes
        for pair in hi[1:]:  pair[1] = '1' + pair[1]  # prepend 1 to hi's codes

        heapq.heappush(heap, [lo[0] + hi[0]] + lo[1:] + hi[1:])

    return sorted(heap[0][1:], key=lambda p: (len(p[-1]), p))

# Kruskal's MST — also greedy!
def kruskal_mst(n, edges):
    """
    Minimum Spanning Tree. Greedy: always add cheapest edge that doesn't form cycle.
    """
    edges.sort(key=lambda e: e[2])  # sort by weight
    uf  = UnionFind(n)
    mst = []
    total_cost = 0

    for u, v, w in edges:
        if uf.union(u, v):          # no cycle formed
            mst.append((u, v, w))
            total_cost += w
        if len(mst) == n - 1: break # MST complete

    return mst, total_cost
```

```cpp
// Interval scheduling — C++
int maxNonOverlapping(vector<pair<int,int>>& intervals) {
    sort(intervals.begin(), intervals.end(),
         [](auto& a, auto& b){ return a.second < b.second; });
    int count = 0, lastEnd = INT_MIN;
    for (auto& [start, end] : intervals) {
        if (start >= lastEnd) { count++; lastEnd = end; }
    }
    return count;
}

// Jump Game — C++
bool canJump(vector<int>& nums) {
    int maxReach = 0;
    for (int i = 0; i < nums.size(); i++) {
        if (i > maxReach) return false;
        maxReach = max(maxReach, i + nums[i]);
    }
    return true;
}
```

**Two Classic Greedy Proofs**

```
Interval Scheduling exchange argument:
  Suppose we have a non-greedy solution S.
  S picked interval A (ends at time eA) instead of greedy's B (ends at eB ≤ eA).
  Swapping A for B: the interval ends earlier (eB ≤ eA), so it still doesn't
  conflict with the next interval. The count doesn't decrease.
  Therefore greedy is at least as good → optimal.

Activity scheduling greedy is optimal!
```

---

## 19. Dynamic Programming

DP solves problems by **breaking them into overlapping subproblems** and storing results to avoid recomputation. It's applicable when the problem has:
1. **Optimal substructure** — optimal solution built from optimal subproblems
2. **Overlapping subproblems** — same subproblems computed multiple times

### DP vs Greedy vs Divide & Conquer

```
Divide & Conquer: subproblems DON'T overlap (merge sort, binary search)
Greedy:           one pass, local choice, subproblems don't need to be saved
DP:               subproblems DO overlap, save all results, build bottom-up or top-down
```

### Two DP Approaches

```python
# Problem: Fibonacci
# Approach 1: Top-down (memoization) — recursion + cache
from functools import lru_cache

@lru_cache(maxsize=None)
def fib_topdown(n):
    if n <= 1: return n
    return fib_topdown(n-1) + fib_topdown(n-2)

# Approach 2: Bottom-up (tabulation) — fill table from smallest to largest
def fib_bottomup(n):
    if n <= 1: return n
    dp = [0] * (n + 1)
    dp[0] = 0; dp[1] = 1
    for i in range(2, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]

# Space-optimized — O(1)
def fib_optimized(n):
    a, b = 0, 1
    for _ in range(n): a, b = b, a + b
    return a
```

### Classic DP Problems

**1D DP — Climbing Stairs**

```python
def climb_stairs(n):
    """How many ways to climb n stairs (1 or 2 steps at a time)? = Fibonacci"""
    if n <= 2: return n
    dp = [0] * (n + 1)
    dp[1] = 1; dp[2] = 2
    for i in range(3, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]
```

**1D DP — House Robber**

```python
def rob(nums):
    """
    Max money from houses — can't rob adjacent houses.
    State: dp[i] = max money robbing houses 0..i
    Transition: dp[i] = max(dp[i-1], dp[i-2] + nums[i])
    """
    if not nums: return 0
    if len(nums) == 1: return nums[0]

    prev2, prev1 = 0, nums[0]
    for i in range(1, len(nums)):
        curr = max(prev1, prev2 + nums[i])
        prev2, prev1 = prev1, curr
    return prev1
```

```java
static int rob(int[] nums) {
    if (nums.length == 1) return nums[0];
    int prev2 = 0, prev1 = nums[0];
    for (int i = 1; i < nums.length; i++) {
        int curr = Math.max(prev1, prev2 + nums[i]);
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```

**2D DP — Unique Paths in Grid**

```python
def unique_paths(m, n):
    """
    Count paths from top-left to bottom-right (only right or down).
    dp[i][j] = dp[i-1][j] + dp[i][j-1]
    """
    dp = [[1] * n for _ in range(m)]  # first row and column are all 1
    for i in range(1, m):
        for j in range(1, n):
            dp[i][j] = dp[i-1][j] + dp[i][j-1]
    return dp[m-1][n-1]

# Space-optimized to O(n)
def unique_paths_opt(m, n):
    dp = [1] * n
    for _ in range(1, m):
        for j in range(1, n):
            dp[j] += dp[j-1]
    return dp[-1]
```

**0/1 Knapsack — Include or Exclude**

```python
def knapsack(weights, values, capacity):
    """
    Maximum value fitting in bag of given capacity.
    State: dp[i][w] = max value using first i items with capacity w
    Transition: dp[i][w] = max(dp[i-1][w], dp[i-1][w-wt[i]] + val[i])
    """
    n  = len(weights)
    dp = [[0] * (capacity + 1) for _ in range(n + 1)]

    for i in range(1, n + 1):
        for w in range(capacity + 1):
            # Don't take item i
            dp[i][w] = dp[i-1][w]
            # Take item i (if it fits)
            if weights[i-1] <= w:
                dp[i][w] = max(dp[i][w], dp[i-1][w - weights[i-1]] + values[i-1])

    return dp[n][capacity]

# Space-optimized to O(W) — go backwards to avoid using item twice!
def knapsack_opt(weights, values, capacity):
    dp = [0] * (capacity + 1)
    for w, v in zip(weights, values):
        for c in range(capacity, w - 1, -1):  # MUST go backwards for 0/1
            dp[c] = max(dp[c], dp[c - w] + v)
    return dp[capacity]
```

**Longest Common Subsequence (LCS)**

```python
def lcs(s1, s2):
    """
    dp[i][j] = LCS of s1[:i] and s2[:j]
    If s1[i-1] == s2[j-1]: dp[i][j] = dp[i-1][j-1] + 1
    Else:                   dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    """
    m, n = len(s1), len(s2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]

    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if s1[i-1] == s2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])

    return dp[m][n]

# Longest Common Substring — must be contiguous
def lcss(s1, s2):
    m, n = len(s1), len(s2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    longest = 0
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if s1[i-1] == s2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
                longest = max(longest, dp[i][j])
            else:
                dp[i][j] = 0   # must be contiguous!
    return longest
```

**Longest Increasing Subsequence (LIS)**

```python
def lis(nums):
    """
    Standard DP: O(n²) time
    dp[i] = length of LIS ending at index i
    """
    n = len(nums)
    dp = [1] * n
    for i in range(1, n):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    return max(dp)

# Binary search optimized: O(n log n)
def lis_nlogn(nums):
    tails = []  # tails[i] = smallest tail of LIS with length i+1
    for num in nums:
        lo, hi = 0, len(tails)
        while lo < hi:           # binary search for position to insert num
            mid = (lo + hi) // 2
            if tails[mid] < num: lo = mid + 1
            else: hi = mid
        if lo == len(tails): tails.append(num)
        else: tails[lo] = num    # replace — doesn't affect length, optimizes future
    return len(tails)
```

**Edit Distance (Levenshtein)**

```python
def edit_distance(word1, word2):
    """
    Minimum operations (insert, delete, replace) to transform word1 → word2.
    dp[i][j] = edit distance between word1[:i] and word2[:j]
    """
    m, n = len(word1), len(word2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]

    # Base cases
    for i in range(m + 1): dp[i][0] = i  # delete all from word1
    for j in range(n + 1): dp[0][j] = j  # insert all from word2

    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if word1[i-1] == word2[j-1]:
                dp[i][j] = dp[i-1][j-1]       # no operation needed
            else:
                dp[i][j] = 1 + min(
                    dp[i-1][j],     # delete from word1
                    dp[i][j-1],     # insert into word1
                    dp[i-1][j-1],   # replace
                )

    return dp[m][n]
```

**Coin Change — Unbounded Knapsack**

```python
def coin_change(coins, amount):
    """
    Minimum coins to make amount. dp[i] = min coins for amount i.
    Unbounded: each coin can be used multiple times → forward iteration!
    """
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0

    for a in range(1, amount + 1):
        for coin in coins:
            if coin <= a:
                dp[a] = min(dp[a], dp[a - coin] + 1)

    return dp[amount] if dp[amount] != float('inf') else -1

def coin_change_ways(coins, amount):
    """Number of ways to make amount (order doesn't matter)"""
    dp = [0] * (amount + 1)
    dp[0] = 1
    for coin in coins:
        for a in range(coin, amount + 1):
            dp[a] += dp[a - coin]
    return dp[amount]
```

**Matrix Chain Multiplication — Interval DP**

```python
def matrix_chain(dims):
    """
    Minimum scalar multiplications to multiply a chain of matrices.
    dims[i] × dims[i+1] = dimension of matrix i.
    Interval DP: dp[i][j] = min cost to multiply matrices i through j.
    """
    n = len(dims) - 1  # number of matrices
    dp = [[0] * n for _ in range(n)]

    for length in range(2, n + 1):       # length of chain
        for i in range(n - length + 1):
            j = i + length - 1
            dp[i][j] = float('inf')
            for k in range(i, j):        # try splitting at k
                cost = dp[i][k] + dp[k+1][j] + dims[i] * dims[k+1] * dims[j+1]
                dp[i][j] = min(dp[i][j], cost)

    return dp[0][n-1]
```

```cpp
// Coin change — C++
int coinChange(vector<int>& coins, int amount) {
    vector<int> dp(amount + 1, INT_MAX);
    dp[0] = 0;
    for (int a = 1; a <= amount; a++) {
        for (int coin : coins) {
            if (coin <= a && dp[a - coin] != INT_MAX) {
                dp[a] = min(dp[a], dp[a - coin] + 1);
            }
        }
    }
    return dp[amount] == INT_MAX ? -1 : dp[amount];
}

// LCS — C++
int lcs(string& s1, string& s2) {
    int m = s1.size(), n = s2.size();
    vector<vector<int>> dp(m+1, vector<int>(n+1, 0));
    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++)
            dp[i][j] = s1[i-1] == s2[j-1] ? dp[i-1][j-1] + 1
                                            : max(dp[i-1][j], dp[i][j-1]);
    return dp[m][n];
}
```

```java
// Knapsack 0/1 — Java
static int knapsack(int[] weights, int[] values, int capacity) {
    int n = weights.length;
    int[] dp = new int[capacity + 1];
    for (int i = 0; i < n; i++)
        for (int c = capacity; c >= weights[i]; c--)
            dp[c] = Math.max(dp[c], dp[c - weights[i]] + values[i]);
    return dp[capacity];
}

// Edit distance — Java
static int editDistance(String w1, String w2) {
    int m = w1.length(), n = w2.length();
    int[][] dp = new int[m+1][n+1];
    for (int i = 0; i <= m; i++) dp[i][0] = i;
    for (int j = 0; j <= n; j++) dp[0][j] = j;
    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++)
            dp[i][j] = w1.charAt(i-1) == w2.charAt(j-1) ? dp[i-1][j-1]
                      : 1 + Math.min(dp[i-1][j], Math.min(dp[i][j-1], dp[i-1][j-1]));
    return dp[m][n];
}
```

---

## 20. Interview Cheatsheet & Common Patterns

### Pattern Recognition Guide

```
Input: sorted array or search by value → Binary Search
Input: graph, find connected components → DFS + visited set
Input: graph, shortest path (unweighted) → BFS
Input: graph, shortest path (weighted) → Dijkstra (heap + relaxation)
Input: tree traversal → Recursion (DFS) or BFS for level-order
Input: "find all" / "enumerate" → Backtracking
Input: "minimum/maximum" + overlapping subproblems → DP
Input: "minimum/maximum" + provable greedy property → Greedy
Input: contiguous subarray sum/length → Sliding Window
Input: pair/triplet in sorted array → Two Pointers
Input: duplicates / fast lookup → HashMap or Set
Input: "kth largest/smallest" → Heap (PriorityQueue)
Input: ranges / intervals → Sort by start or end
Input: parentheses / expression evaluation → Stack
```

### Time Complexity Quick Reference

```python
# Sorting and searching
sorted_list.sort()         # O(n log n)
bisect.bisect(sorted, x)   # O(log n)

# Collections
dict/set lookup/insert     # O(1) average
heappush/heappop           # O(log n)
deque.append/popleft       # O(1)

# String operations
s + t                      # O(len(s) + len(t)) — creates new string
"".join(list_of_strs)      # O(total chars) — USE THIS for concatenation!

# Common patterns and their complexities
two_pointers(arr)          # O(n)
sliding_window(arr)        # O(n)
binary_search(arr)         # O(log n)
bfs(graph)                 # O(V + E)
dfs(graph)                 # O(V + E)
merge_sort(arr)            # O(n log n)
dijkstra(graph, n)         # O((V+E) log V)
bellman_ford(graph, n)     # O(VE)
floyd_warshall(graph, n)   # O(V³)
```

### The UMPIRE Problem-Solving Framework

```
U — Understand:  Clarify the problem. Edge cases. Examples.
M — Match:       What pattern does this match? (see Pattern Recognition above)
P — Plan:        Algorithm + data structures. Time/space complexity. Verify with example.
I — Implement:   Write clean code. Name variables well.
R — Review:      Trace through example. Check edge cases. Off-by-one?
E — Evaluate:    Time + space complexity. Can it be improved?
```

### Interview Quick-Fire Q&A

**Q: When does binary search work?**
When the search space is sorted OR when you can define a monotone condition: values shift from False to True (or True to False). Search on the answer, not just the array.

**Q: When to use BFS vs DFS?**
BFS: shortest path (unweighted), level-order processing, any "minimum steps" problem. DFS: detecting cycles, topological sort, backtracking, exploring all paths, tree serialization.

**Q: When to use a heap vs sorting?**
Heap when you need **online** access to minimum/maximum as data arrives, or when you only need top-k elements (O(n log k) vs O(n log n) for full sort). Sort when you need the entire ordered list.

**Q: How do you recognize a DP problem?**
"Optimal" (min/max/longest/shortest) + overlapping subproblems. Try: "can I break this into smaller versions of the same problem?" If yes, and the subproblems repeat, it's DP.

**Q: When does greedy fail?**
When local optimal doesn't lead to global optimal. Classic failure: coin change with non-standard denominations (e.g., [1, 3, 4], amount=6: greedy picks 4+1+1=3 coins, but DP finds 3+3=2 coins). Greedy works for activity scheduling, Huffman, Dijkstra, Prim's, Kruskal's.

**Q: How do you handle cycle detection?**
Directed graph: DFS with gray/black coloring (or topological sort — cycle if not all nodes included). Undirected graph: DFS tracking parent, or Union-Find (cycle if union returns false). Linked list: Floyd's slow/fast pointers.

**Q: What's amortized analysis?**
Average cost per operation over a sequence, even if individual operations vary. Dynamic array: pushing n elements = n + resizing costs (n/2 + n/4 + ...) ≈ 2n total = O(1) amortized per push. The expensive operation is "paid for" by earlier cheap ones.

---

## 📖 Master Resource List

- [LeetCode Patterns](https://seanprashad.com/leetcode-patterns/) — problems by pattern
- [NeetCode 150](https://neetcode.io/practice) — curated problem list with video solutions
- [VisuAlgo](https://visualgo.net/en) — animated algorithm visualizations
- [Big-O Cheat Sheet](https://www.bigocheatsheet.com/)
- [CP-Algorithms](https://cp-algorithms.com/) — competitive programming algorithms with proofs
- [MIT 6.006 Introduction to Algorithms](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-fall-2011/)
- [CLRS — Introduction to Algorithms](https://mitpress.mit.edu/books/introduction-algorithms) — the bible
- [Grokking Algorithms](https://www.manning.com/books/grokking-algorithms) — illustrated, beginner friendly
- [The Algorithm Design Manual — Skiena](https://www.algorist.com/)

---

*Every problem is just a pattern you've seen before, wearing a different costume. Learn the patterns, Ahmed. You've got this. 🧠*

> **Legend:** O(1)=constant | O(log n)=binary search | O(n)=linear | O(n log n)=sort | O(n²)=nested | O(2ⁿ)=exponential
