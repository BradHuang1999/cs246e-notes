[The copier is broken << ](./problem_21.md) | [**Home**](../README.md) | [>> A big unit on Object Oriented Design](./object_oriented_design.md)

# Problem 22: I want to know what kind of Book I have

> **2018-11-01**

For simplicity, let's assume the old book hierarchy

- `Book`

  - `Text`
  - `Comic`

**C-style Casting**

- `(type) expr`
- Forces `expr` to be treated as type `type`
- Ex. `int addr = ...; int* p = (int*) addr;`
- Very easy to be a source of error
- Very difficult to search for

The `C++` casting operators - _4 operators_

- **`static_cast`:** for conversion with well-defined semantics

  - Ex.
    ```C++
    void f(int a);
    void f(double d);
    int x;
    f(static_cast<double>(x));
    ```
  - Ex.
    ```C++
    // superclass pointer to a subclass pointer
    Book *b = new Text { ... };
    Text *t = static_cast<Text *>(b);
    ```
  - Compilers be like: this is your program, you are in charge, but don't blame me if this becomes a problem!
  - You do this if you're 100% sure that pointer is pointing to a `Text`, otherwise it is undefined behavior

- **`reinterpret_cast`:** for casts without well-defined semantics

  - For casts without a well-defined semantics
  - Unsafe, implementation-dependent (therefore unportable)
  - Ex.
    ```C++
    Book *b = new Book { ... };
    int *p = reinterpret_cast<int *>(b);
    ```

- **`const_cast`:** for adding/removing const

  - The only C++ cast that can "cast away `const`"
  - If item is still in read-only memory, it's still read only
  - Ex.
    ```C++
    void g(Book &b);    // Assuming we know g won't change b
    void f(const Book &b) {
        g(const_cast<Book &>(b));
    }
    ```

- **`dynamic_cast`:** `Book *pb = ...`

  - What if we _don't_ know whether `pb` points to a `Text`?
  - `static_cast` is not safe
    ```C++
    Text *pt = dynamic_cast<Text*>(pb);
    ```
  - If `*pb` is a `Text` or a subclass of `Text`, cast succeeds. `pt` points at the object, else `pt = nullptr`.
    ```C++
    if (pt) { ... pt->getTopic() ... }
    else ... // Not a Text
    ```
  - Ex.
    ```C++
    void whatIsIt(Book *pb) {
        if (dynamic_cast<Text*>(pb)) cout << "Text";
        else if (dynamic_cast<Comic*>(pb)) cout << "Comic";
        else cout << "Book";
    }
    ```
    - **Not good style**, what happens when you create a new `Book` type?
  
  - Dynamic reference casting
    ```C++
    Book *pb = ...;
    Text &t = dynamic_cast<Text &>(*pb);
    if (*pb is a Text) // ok
    else // raise std::bad_cast
    ```

  - **Note:** dynamic casting works by accessing an objects **Run-Time Type Information (RTTI)**, this is stored in the vtable for the class
    - This means we can only `dynamic_cast` on objects that have at least one virtual method
  - Dynamic reference casting offers a possible solution to take polymorphic assignment problem:
    ```C++
    Text &Text::operator=(Book other) { // Virtual
        Text &textother = dynamic_cast<Text &>(other);
        // Throws if other is not a Text
        if (this == &textOther) return *this;
        Book::operator=(std::move(textother));
        topic = std::move(textother.topic);
        return this;
    }
    ```
  - However, this solution would pass on the problem to the client - they would know better what to do if the `Book` is not a `Text`.

---

[The copier is broken << ](./problem_21.md) | [**Home**](../README.md) | [>> A big unit on Object Oriented Design](./object_oriented_design.md)
