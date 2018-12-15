[Insert/Remove in the middle <<](./problem_16.md) | [**Home**](../README.md) | [>> Heterogeneous Data](./problem_18.md)

# Problem 17 - Abstraction over Containers

> **2018-10-23**

Recall: `map` from Racket

```scheme
(map f (list a b c)) -> (list (f a) (f b) (f c))
```

May want to do the same with vectors:

```C
+---+---+---+---+      +----+----+---+-----+
| a | b | c | d |  ->  |f(a)|f(b)|f(c)|f(d)|
+---+---+---+---+      +----+----+---+-----+
source                target
```

Assume target has enough space to hold as much of source as we want to send.

```C++
template<typename T1, typename T2>

void transform(const vector<T1> &source, vector<T2> &target, T2 (*f)(T1)) {
    auto it = target.begin();
    for (auto &x: source) {
        *it = f(x);
        ++it;
    }
}
```

Ok but:

- What if we want only part of the source?
- What if we want to send the source to the middle of the target, not the beginning?

More general: use iterators

```C++
template <typename T1, typename T2> void transform(
    vector<T1>::iterator start,
    vector<T1>::iterator finish,
    vector <T2>::iterator target,
    T2 (*f)(T1)) {
    while (start != finish) {
        *target = f(*start);
        ++start;
        ++target;
    }
}
```

This piece of code won't compile for some reason we will explore later, but nevertheless:

- What if I want to transform a list, I'll have to write the same code again - not good
- What if I want to transform a list to a vector, or vice versa?

Solution: Make the types stand for the iterators, not the container elements.

But then how do we indicate the type for `f`? Here it is:

```C++
template<typename InIter, typename OutIter, typename Fn>
void transform(InIter start, InIter finish, OutIter target, Fn f) {
    while (start != finish) {
        *target = f(*start);
        ++target;
        ++start;
    }
}
```

Works over vector iterators, list iterators, or any kinds of iterators.

`InIter`/`OutIter` can be any types that support `++`, `*`, `!=`, including ordinary pointers.

C++ will instantiate a template function with any type that has the operations being used by the function.

> **2018-10-24**

`Fn` can be any type that supports **function application**. Example:

```C++
class Plus {
        int n;
    public:
        Plus(int n): n{n} {}
        int operator() (int m) { return n + m; }
};

Plus p{5};

std::cout << p(7);  // 12
```

But more interesting

```C++
transform(v.begin(), v.end(), w.begin(), Plus{1});
```

Now we have an arbitrary plus one function for any types;

OR

```C++
transform(v.begin(), v.end(), w.begin(), [](int n) { return n + 1 });
                                      // ^ "lambda"
```

```C
            param list
               |
lambda: [...](...) mutable? noexcept? { ... }
          |                              |
        capture list                    body
```

- capture list - provides access to selected vars in the enclosing scope.
- param list - parameters of the lambda function
- If the lambda is declared mutable, then `operator()` is not const.

Semantics:

```C++
void f(T1 a, T2 b) {
    [a, &b](int x) { body }
}
```

Translated to:

```C++
void f (T1 a, T2 b) {
    class ??? {
        // ??? - anonymous class we can't access the name
            T1 a;
            T2 &b;
        public:
            ???(T1 a, T2 &b): a{a}, b{b} {}
            auto operator()(int x) const {
                body;
            }
    }
};
```

## Sidenote: Mutable in Class

``` C++
class A {
    mutable int n;
    int m;
};

int main() {
    A const a = A();
    a.n = 1;  // OK
    a.m = 2;  // Error: can't modify const
}
```

Note that one can only use `auto` to store a lambda function.

[Insert/Remove in the middle <<](./problem_16.md) | [**Home**](../README.md) | [>> Heterogeneous Data](./problem_18.md)
