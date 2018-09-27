[Moves << ](./problem_5.md) | [**Home**](../README.md) | [>> Tampering](./problem_7.md) 

# Problem 6: I want a constant Vector
**2018-09-27**

Say we want to print a `Vector`:

```C++
ostream &operator <<(ostream &out, const Vector &v) {
    for (size_t i = 0; i < v.size(); ++i) {
        out << v.itemAt(i) << " ";
    }
    return out;
}
```

This won't compile! \**lushman pls*\*

- Can't call `size()` and `itemAt()` on a `const` object. Compilers are worried that these methods might change the fields.
- Since they don't, declare them as `const`

```C++
struct Vector {
    ...
    size_t size() const;
    int &itemAt(size_t i) const;    
    // Means these methods will not modify fields
    // Can be called on const objects
    ...
};

size_t Vector::size() const {
    return n;
}

int &Vector:itemAt(size_t i) const {
    return theVector[i];
}
```

Now the loop will work.

**BUT:**

```C++
void f(const Vector &v) {
    v.itemAt(0) = 4;
    // v.itemAt(0) gets a reference back
    // I am free to changed the data that reference is pointing 
    //   at - nothing in compiler can stop me
    // Works!! v not very const...
}
```

- `v` is a const object - cannot change `n`, `cap`, `theVector` (ptr)
- You can changed items pointed to by `theVector`

Can we fix this?

```C++
struct Vector {
    ...
    const int &itemAt(size_t i) const;
    // adding a const in front of the declaration, prohibiting
    //   users from changing this reference
};

const int &itemAt(size_t i) const {
    // adding a const in front of the definition as well
    return theVector[i];
}
```

Now `v.itemAt(0) = 4` won't compile if `v` is const.

**BUT!** it also won't compile if `v` is not const. \**ahhhhhhh*\*

Problem: cannot define methods based on if user defined them as `const` or not 

To fix: **const overloading**

```C++
struct Vector {
    ...
    const int &itemAt(size_t i) const;
    // Will be called if the object is const
    int &itemAt(size_t i);
    // Will be called if object is non-const
};

inline const int &Vector::itemAt(size_t i) const {
    return theVector[i];
}

inline int &Vector::itemAt(size_t) {
    return theVector[i];
}
```

Putting in `inline` tells the compile to replace the function call with the function body to save the cost of having to call a function - \**Don't call it, just drop the function body right there*\*

Merely a suggestion, compiler can choose to ignore it if it sees fit. Good idea for small functions

So now `v.itemAt(0) = 4;` will only compile if and only if `v` is non-const

Now let's make it prettier:

```C++
struct Vector {
    size_t size() const {return n;}
    // Method body inside class implcity declares the method inline
    const int &operator[](size_t i) const {return theVector[i]};
    int &operator[](size_t i) {return theVector[i];}    
};

ostream &operator<<(ostream &out, const Vector &v) {
    for (size_t i = 0; i < v.size(); ++i) {
        out << v[i] << " ";
    }

    return out;
}
```

---
[Moves << ](./problem_5.md) | [**Home**](../README.md) | [>> Tampering](./problem_7.md) 
