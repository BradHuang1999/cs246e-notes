[Is vector exception safe? << ](./problem_15.md) | [**Home**](../README.md) | [>> Abstraction over containers?](./problem_17.md)

# Problem 16: Insert/Remove in the Middle

> **2018-10-23**

A method like `vector<T>::insert(size_t i, const T &x)` is easy to write.  
But for the same `list<T>` requires an upfront traversal.

Using iterators can be good for both.

```C++
template<typename T> class vector {
    ...
    public:
        iterator insert(iterator posn, const T&x) {
            ptrdiff_t offset = posn - begin();
            // ptrdiff_t incase result is negative (in general)
            increaseCap();
            iterator newPosn = begin() + offset;
            new (end()) T(std::move_if_no_except(*(end() - 1)));
            ++vb.n;
            // add new item to end
            for (iterator it = end() - 1; it != Posn; --it) {
                *it = std::move_if_no_except(*(it - 1));
            }
            // shuffle items down using move_if_no_except
            *newPosn = x;
            // iterator to point of insertion;
            return newPosn;
        }
    }
};
```

Is this exception safe? Assuming `T`'s copy/move operations are exception safe (at least basic guarantee), insert offers the basic guarantee.

- May get a partially shuffled vector, but it will be a valid vector.

**Note:** if you have other iterators pointing at the vector

```C
+---+---+---+---+---+
| 1 | 2 | 3 | 4 |...|
+---+---+---+---+---+
  ^       ^   ^
 it1      h  it2

and you insert at h

+---+---+---+---+---+---+
| 1 | 2 | 5 | 3 | 4 |...|
+---+---+---+---+---+---+
  ^       ^   ^
 it1      h  it2
```

`it2` will now point at a different item.

**Convention:** after a call to insert or erase all iterators pointing after the point of insertion/erasure are considered invalid and should not be used.

- Also, if a reallocation happens, _all_ iterators pointing into the vector become invalid.

Exercises:

- **erase**
  - remove the item pointer to by an iterator
  - return an iterator to the point of erasure
- **emplace**
  - like insert but takes constructor args

BUT - that means there is a problem with `push_back`. If `increaseCap` successfully reallocates and placement new (constructor) throws, the vector is the same but the iterators were invalidated!

To fix:

- allocate new array
- place the new item
- copy or move old items
- delete

FYI: why `void pop_back()` instead of `T pop_back()`?

- `T pop_back()` would call (or move) copy ctor on return
- the destructor on item in vector
- if copy ctor throws, dtor call won't happen
- not exception safe

---

[Is vector exception safe? << ](./problem_15.md) | [**Home**](../README.md) | [>> Abstraction over containers?](./problem_17.md)
