[<< A fixed-size object allocator](./problem_32.md) | [**Home**](../README.md)

# Problem 33: I want a (tiny bit) smaller vector class

> **2018-11-29** Last class :'(

**Currently:** `vector`/`vector_base` have an allocator field. Standard allocator is stateless (has no fields)

What is its size? 0? No - C++ does not allow 0 size types (messes things up, ex. pointer arithmetic).

Every type has at least size 1.

Practically - compiler adds a dummy `char` to the allocator.

So having an allocator field makes the `vector` larger by a byte. Probably more, due to alignment.

Having an allocator field in the vector_base, may add another byte (or more).

To save this space, C++ provides the **empty base optimization (EBO)**.

Under EBO, an empty base class does not have to occupy space in an object.

So we can eliminate the space cost of an allocator by making it a base class. At the same time, make `vector_base` a base class of a `vector`.

```C++
template <typename T, typename Alloc = allocator<T>>
struct vector_base: private Alloc {
    // struct has default public inheritance
    size_t n, cap;
    T *v;
    using Alloc::allocate;
    using Alloc::deallocate;
    // etc.
    // Because we don't know anything about Alloc, we need to bring these members into scope.
    vector_base(size_t n): n{0}, cap{n}, v{allocate(n)} {}
    ~vector_base() { deallocate(v); }
};

template <typename T, typename Alloc = allocator<T>>
class vector: vector_base<T, Alloc> {
    // private inheritance - no is-a relation
    using vector_base<T, Alloc>::n;
    using vector_base<T, Alloc>::cap;
    using vector_base<T, Alloc>::v;

    using Alloc:allocate;   // or say this->allocate
    using Alloc:deallocate; // this->deallocate

   public:
    ... Use n , cap, v instead of vb.n, vb.cap, vb.v
};
```

`uninitalized_copy`, etc. - need to call construct/destroy

- Simplest - let the take an allocator as a parameter

```C++
template <typename T, typename Alloc> {
    void uninitialized_fill(T *start, T *finish, const T &x, Alloc a) {
        ...
        a.construct(...)
        ...
        a.destroy(...)
        ...
    }
}
```

How can vector pass an allocator to these functions?

```C++
uninitialized_fill(v, v + n, x, static_cast<Alloc&>(*this));    // Cast yourself to base class reference
```

Remaining details - exercise

## Conclusions

How should we view what we learned this term?

This is not a course in C++, nor polymorphism, or templates. These are only the tools for abstraction. The goal is to make programmers easy to get the job done, without exposing them to unnecessary implementation.

C++ is good for this goal because it allows programmers to both think in Object-Oriented designs and create high-level abstractions, as well as manipulate the memory bit by bit.

In this course, we pretended to be the writers of the standard library. We need to consider the hard problems, and if we do a good job, we will make our users' lives easier.

C++ is easy because it is hard.

---

[<< A fixed-size object allocator](./problem_32.md) | [**Home**](../README.md)
