[Copies <<](./problem_4.md) | [**Home**](../README.md) | [>> I want a constant Vector](./problem_6.md)

# Problem 5: Moves
**2018-09-25**

Consider a function that intends to increment by one for all node values:

```C++
Node plusOne(Node n) {
    for (Node *p = &n; p; p = p->next) {
        ++p->data;
    }

    return n;
}

Node n {1, new Node {2, nullptr}};
Node m = plusOne(n);    // this calls for the copy constructor
```

In this case, "other" is a reference to the temporary object created to hold the result of `plusOne(n)`.

- "Other" is a reference to this temporary
- Copy constructor deep-copies the data from this temporary

**But!!!** the temporary is just going to be discarded as soon as the statement `Node m = plusOne(n)` is done

It is wasteful to deep copy the temporary object - why not "steal" the data instead - "You are gonna die anyway, why not let me take your stuff?"

Stealing the data saves the cost of a copy.

We need to be able to tell whether "other" is a reference to a temporary object, or a standalone object.

## Rvalue references

Christopher Strachey, the inventor of CPL language (the father of B - which is the father of C), stated that: every object can potentially have two value: Lvalue and Rvalue

- The Lvalue is something that can occur on the left hand side of an assignment: addresses denoting a storage location
- The Rvalue is something that cannot occur on the left hand side of assignments: usually temporary objects

`Node &&` is a reference to a temporary object (rvalue) of type `Node`. We need a version of the constructor that takes a `Node &&`

**Move Constructors** - steals other's data

```C++
struct Node {
    ...
    Node(Node &&other): data{other.data}, next{other.next} {
        other.next = nullptr;
        // right now both this and other have the same next field
        //   that's not stealing, that's sharing!
        //   when other is destroyed, it will take next with it
        //   so we would need ot take it away from it as well
    }
};
```

Similarly:
```C++
Node m;
m = plusOne(n);  // assignment from temporary
```

**Move assignment operator**

```C++
struct Node {
    ...
    Node &operator=(Node &&other) {     // steal other's data
        using std::swap;
        // need to destroy my old data
        // "Since you are about to die anyway, how about you take
        //   my old data with me?"
        
        swap(data, other.data);     // Easy: swap without the copy
        swap(next, other.next);     // Same thing

        return *this;
    }
};
```

> **2018-09-27**

Can combine copy/move assignment:

```C++
struct Node {
    ...
    // recall the swap function
    void swap(Node &other) {
        using std::swap;
        swap(data, other.data);
        swap(next, other.next);
    }

    Node &operator=(Node other) {
        swap(other);
        return *this;
    }
};
```

- Called unified assignment operator
    - Pass ``other`` by value
    - Invokes copy constructor if the argument is an lvalue
    - Invokes move constructor (if there is one) if the argument is an rvalue

**Note:** copy/swap can be expensive, hand-coded operator may do less copying

But now consider:

```C++
struct Student {
    std::string name;
    Student(const std::string &name): name{name} {  // copies name into field (copy ctor)
        ...
    }
};
```

What if `name` points to an rvalue?

```C++
struct Student {
    std::string name;

    Student (const std::string &name): name{name} {
        // {name} may come from an rvalue, but it is an lvalue
        ...
    }
};
```

Will copy if `name` is an lvalue, moves if `name` is an rvalue

```C++
struct Student {
    std::string name;
    Student(std::string name): name{std::move(name)} {
        // std::move(name) is treated as an rvalue, and name
        //   is now a name construction
    }
}
```

`name{std::move(name)}` forces `name` to be treated as an rvalue, now strings move constructor

Consider:

```C++
struct Student {
    ...
    Student(Student &&other): // move constructor
        name{other.name} {
            ...
        }
}
```

This will still call a copy constructor, as `other` is still an lvalue.

So we will solve the problem using `std::move`.

```C++
struct Student {
    ...
    Student(Student &&other): // move constructor
        name{std::move(other.name)} {
            // correct
            ...
        }
}
```

If you don't define move operations, copy operations will be used.

If you do define them, then replace copy operations whenever the arg is a temporary (rvalue).

## Copy/Move Elision

Consider the following:

```C++
Vector makeAVector() {
    return Vector{}   // Basic constructor
}

Vector v = makeAVector();   // move ctor? copy ctor?
```

Try in g++, just the basic constructor, not copy/move

In some circumstances, the compiler is allowed to skip calling the copy/move constructors (but doesn't have to). 

`makeAVector()` writes its result directly into the space occupied by `v`, rather than copy/move it later.

Ex.
```C++
Vector v = Vector{};
// Formally a basic construction and a copy/move construction
//   Vector{} is a basic constructor
// Here though, the compiler is *required* to elide the 
//   copy/move, so basic constructor here only 

Vector v = Vector{Vector{Vector{}}};    
// Still one basic ctor only
```

Ex.
``` C++
void doSomething(Vector v) {...};
// Pass-by-value - copy/move ctor

doSomething(makeAVector());
// Result of `makeAVector()` written directly into the 
//   param, there is no copy/move
```

This is allowed, even if dropping ctor calls would change the behaviour of the program (ex. if the constructors print something).

If you really need all of the constructors to run:

`g++14 -fno_elide_constructors ...`

**Note:** while possible, can slow down your program considerably

- Copying is an expensive procedure, the compiler skips these constructors as an optimization

**In summary: Rule of 5 (Big 5)**

If you need to customize any one of
- Copy constructor
- Copy assignment
- Move constructor
- Move assignment
- Destructor

then you usually need to customize all 5.

From now on , we will assume that Node & Vector have the Big 5 defined.

---
[Copies <<](./problem_4.md) | [**Home**](../README.md) | [>> I want a constant Vector](./problem_6.md)
