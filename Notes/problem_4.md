[Linear Collections and Memory Management << ](./problem_3.md) | [**Home**](../README.md) | [>> Moves](./problem_5.md)

# Problem 4: Copies

> **2018-09-25**

```C++
Vector v;

v.pushback(100);
...
Vector w = v;  // Allowed - constructs w as a copy of v
w.itemAt(0);  // 100
v.itemAt(0); = 200;
w.itemAt(0);  // 200 - **shallow copy**, v and w share data
```

Problems:

- Operator= copies references from fields
- The program will like crash in the end of the execution, as destructor would try to free twice on the shared data

For `Vector w = v;`

- Constructs `w` as a copy of `v`
- Invokes the **copy constructor**

```C++
struct Vector {
    Vector(const Vector &other) {...}  // Copy constructor
    // Compiler supplied copy-ctor, copies all fields, shallow copy
};
```

If you want a deep copy, write your own copy constructor

```C++
struct Node {   // Vector: exercise
    int data;
    Node *next;
    ...
    Node (const Node &other):
        data{other.data},
        next{other.next ? new Node{*(other.next)} : nullptr}
        // Account for dereferencing a nullptr
        {...}
};
```

```C++
Vector v;
Vector w;

w = v;
// Copy, but not a construction
// Copy assignment operator
// Compiler supplied:
//   copies each field (shallow)
//   causes the program to crash at destruction
//   leaks w's old data
```

## Deep copy assignment

```C++
struct Node {   // Vector: excercise
    Node &operator=(const Node &other) {
        data = other.data;

        delete next;    // to prevent memory leak
        next = other.next ? new Node{*(other.next)} : nullptr;

        return *this;
    }
};
```

**WRONG - dangerous**

Consider:

```C++
Node n {...};
n = n;
```

Destroys `n`'s data and then copies it.

You might think: why would I ever do something like that? Consider:

```C++
*p = *q     // p and q points to the same object
```

Or:

```C++
for (int i = 0; i < ...; i++) {
    for (int j = 0; j < ...; j++) {
        a[i] = a[j];    // i and j might collide
    }
}
```

The point is, programmers might encounter this situation without knowing it. We must always ensure the operator = works in the case of self assignment.

```C++
Node &Node::operator=(const Node &other) {
    if (this != &other) {
        data = other.data;

        delete next;    // to prevent memory leak
        next = other.next ? new Node{*other.next} : nullptr;
    }

    // if other is this, do not do anything!

    return *this;
}
```

**Alternative: copy-and-swap idiom**

```C++
#include <utility>

struct Node {
    ...
    void swap(Node &other) {
        using std::swap;
        swap(data, other.data);
        swap(next, other.next);
    }

    Node &operator=(const Node &other) {
        Node tmp = other;
        // we would assume that a working version of **deep**
        //   copy constructor and destructor already exists

        swap(tmp);
        // tmp is destroyed (by the working destructor) when it
        //   is out of the scope

        return *this;
    }
};
```

---

[Linear Collections and Memory Management << ](./problem_3.md) | [**Home**](../README.md) | [>> Moves](./problem_5.md)
