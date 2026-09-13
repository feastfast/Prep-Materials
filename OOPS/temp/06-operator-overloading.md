# Operator Overloading

> **Note:** Operator overloading is a **C++-specific** feature (Java deliberately doesn't support user-defined operator overloading — only `+` for `String` concatenation is built-in and not user-extensible). This entire topic is C++-only.

---

## 1. What Operator Overloading Does

> Lets you redefine what an operator (`+`, `==`, `<<`, `[]`, ...) means when applied to objects of a **user-defined class** — making custom types behave with the same natural syntax as built-in types.

```cpp
Complex c1(2, 3), c2(4, -5), c3;
c3 = c1 + c2;              // reads naturally, instead of: c3 = c1.add(c2);
cout << "c1 + c2: " << c3; // works with cout, just like a built-in type
```

### Syntax
```cpp
ReturnType ClassName::operator Symbol(argument list) {
    // function body
}
```

---

## 2. Overloadable vs. Non-Overloadable Operators

**Most operators can be overloaded** — arithmetic (`+ - * /`), comparison (`== != < > <= >=`), assignment (`= += -= ...`), subscript (`[]`), function-call (`()`), increment/decrement (`++ --`), stream (`<< >>`), dereference/pointer (`* ->`), and more.

**A small, fixed set cannot be overloaded**, for structural reasons:
| Operator | Why not |
|---|---|
| `::` (scope resolution) | Operates on names/scopes, not values — there's no "object" to overload it for |
| `.` (member access) | Must always mean "access this exact member," or basic language guarantees break |
| `.*` (pointer-to-member access) | Same reasoning as `.` |
| `?:` (ternary) | Requires special short-circuit evaluation the language controls directly |
| `sizeof` | A compile-time query, not a runtime operation on values |

> **A crucial rule that's frequently tested:** **you cannot change an operator's precedence, associativity, or number of operands.** Overloading `+` still means it's binary and still respects normal precedence relative to `*`; you're only changing **what it computes**, never its grammatical behavior.

---

## 3. Member Function vs. Friend (Non-Member) Function Overloading

This is the single most commonly tested design decision in this topic.

### Member function overloading
```cpp
class Complex {
    float x, y;
public:
    Complex operator+(Complex& c) {   // MEMBER — implicit left operand is *this*
        Complex result;
        result.x = x + c.x; result.y = y + c.y;
        return result;
    }
};
c3 = c1 + c2;   // equivalent to: c3 = c1.operator+(c2);
```
The **left operand** of the operator is implicitly `*this` — this works fine as long as the left operand is **always** an object of the class itself.

### Friend function overloading — needed for symmetry
```cpp
friend Complex operator+(Complex& c1, Complex& c2);   // FRIEND — both operands explicit
Complex operator+(Complex& c1, Complex& c2) {
    Complex c3;
    c3.x = c1.x + c2.x; c3.y = c1.y + c2.y;
    return c3;
}
```

**Why this matters specifically:** if `+` were only a *member* function, `c1 + c2` would work (left operand is a `Complex`), but **`2 + c1`** would not — there's no way for the compiler to call a member function on the literal `2`, since `2` isn't a `Complex` object with an `operator+` member at all. Making it a **friend** (non-member, but still granted access to private data) lets the operator be written to accept **either** order of operands, or specifically an operand type on the left that isn't the class itself.

> **Interview soundbite:** "The deciding question is: 'does the left-hand operand always have to be my class?' If yes, a member function is simpler and idiomatic. If the operator needs to work with the class on either side — or, even more clearly, if the left operand is never your class at all — you need a friend/non-member function instead."

### The clearest case where a member function is *impossible*: `<<`
```cpp
friend ostream& operator<<(ostream& os, const Complex& c) {
    os << c.x;
    if (c.y < 0) os << c.y << "i";
    else os << "+" << c.y << "i";
    return os;
}
cout << c1;   // equivalent to: operator<<(cout, c1);
```
`cout << c1` has `cout` (an `ostream`) as its **left** operand — you cannot add an `operator<<` member function to the *class itself* to handle this, since a member function's left operand is always `*this` (a `Complex`), never an `ostream`. `operator<<` for custom output **must** be a non-member (friend) function for this exact structural reason — this is one of the most reliable "why must this be a friend function" interview questions.

---

## 4. Worked Examples — a Complete `Complex` Number Class

The following are all verified, working overloads from a single consistent `Complex` class (`x` = real part, `y` = imaginary part):

### Unary minus (negation) — a **member** function, one operand (`*this`)
```cpp
Complex& Complex::operator-() {
    x = -x; y = -y;
    return *this;
}
// usage: -c1
```

### Prefix vs. postfix increment — distinguished by a dummy `int` parameter
```cpp
Complex& Complex::operator++() {          // PREFIX: ++c1
    ++x; ++y;
    return *this;                          // returns the MODIFIED object itself
}
Complex Complex::operator++(int) {        // POSTFIX: c1++  (the `int` here is a dummy marker, unused)
    Complex t = *this;                     // save the ORIGINAL value first
    x++; y++;
    return t;                              // returns the ORIGINAL (pre-increment) value
}
```
> **Why the dummy `int` parameter exists:** C++ has no other way to give the compiler two different overloads for the textually-identical `++c1` vs. `c1++` — the language designers picked an unused `int` parameter purely as a **marker** to distinguish "this is the postfix version" at the overload-resolution level. It's never actually used inside the function body.
> **Why prefix returns a reference but postfix returns a value:** prefix modifies and then returns *the same, now-modified* object — a reference is natural and avoids an unnecessary copy. Postfix **must** return the *original* (pre-increment) value, which requires saving a copy *before* modifying `*this` — so it can't return a reference to `*this` (which has already changed by the time it returns).

### Assignment operator
```cpp
Complex Complex::operator=(const Complex& c) {
    x = c.x; y = c.y;
    return *this;
}
```
> **A detail worth knowing for interviews (not shown in the basic example above, but a common follow-up):** a robust `operator=` should check for **self-assignment** (`if (this == &c) return *this;`) before proceeding — this matters enormously once a class manages dynamically allocated resources (recall the deep-copy discussion in Constructors & Destructors): naively freeing old memory and *then* reading from `c` when `c` and `*this` are actually the **same object** would free memory you're about to read from.

### Subscript operator `[]`
```cpp
float Complex::operator[](int i) {
    if (i) return y;    // index 1 → imaginary part
    else return x;      // index 0 → real part
}
// usage: c1[0], c1[1]
```
Lets a custom type support array-style indexing syntax — extremely common for custom container/vector-like classes.

### Function-call operator `()` — enables making an object "callable"
```cpp
float Complex::operator()() {                  // no-arg version: magnitude
    return pow(x*x + y*y, 0.5);
}
void Complex::operator()(float x) {            // one-arg version: reset both parts to x
    this->x = x; this->y = x;
}
// usage: c1()   (magnitude)     c1(100)   (reset)
```
Overloaded like any other function — `()` can have multiple overloads distinguished by parameter list, exactly like ordinary function overloading. This is the mechanism behind C++ **functors** (function objects) — objects that can be "called" like a function while still carrying their own state.

### Equality/inequality — `bool`-returning comparisons
```cpp
bool Complex::operator==(Complex& c) { return x == c.x && y == c.y; }
bool Complex::operator!=(Complex& c) { return x != c.x || y != c.y; }
```

### Binary arithmetic — friend functions (both operands explicit)
```cpp
friend Complex operator+(Complex& c1, Complex& c2);
friend Complex operator-(Complex& c1, Complex& c2);
friend Complex operator*(Complex& c1, Complex& c2);   // (a+bi)(c+di) = (ac-bd) + (ad+bc)i
friend Complex operator/(Complex& c1, Complex& c2);   // complex division via the conjugate
```

### Compound assignment — implemented in terms of the simple operators (good practice!)
```cpp
void Complex::operator+=(Complex& c) { (*this) = (*this) + c; }
void Complex::operator-=(Complex& c) { (*this) = (*this) - c; }
```
> **Design principle worth noting:** implementing `+=` in terms of the already-written `+` and `=` avoids duplicating the actual arithmetic logic — a small but genuinely good practice (don't repeat the math twice, once in `+` and again in `+=`).

### Dereference operator `*` (unary, distinct from binary multiplication)
```cpp
float Complex::operator*() { return x; }   // unary * — e.g., mimicking pointer-like dereference
// usage: *c2
```
> **Note:** `*` is overloaded **twice** in this class — once as a unary operator (dereference-style, taking no explicit parameter beyond the implicit `*this`) and once as a binary friend function (multiplication, taking two `Complex&` parameters). **C++ tells these apart purely by their parameter count** — this is exactly why a symbol like `*` or `-` can serve double duty as both a unary and binary operator.

---

## 5. Interview Soundbite Summary

> "Operator overloading doesn't add new capability to C++ — everything you can write with an overloaded operator, you could also write as a plainly-named member function. What it adds is **readability and consistency with built-in types** — code that manipulates a `Complex` or `Matrix` class reads exactly like code manipulating an `int` or `double`. The main things interviewers actually probe are: (1) member vs. friend, and *why* — usually via the `<<` example, since that's the case where a member function is structurally impossible; (2) the prefix/postfix dummy-`int` trick; and (3) the self-assignment check in `operator=`, once resource ownership is involved."

---

## Interview Questions With Answers

### Q1. Why can't operator precedence or the number of operands be changed when overloading an operator?
**Answer:** Operator overloading only changes *what* an operator computes for a given type — it never changes the operator's fundamental grammatical role in the language. `+` remains a binary operator with the same precedence relative to `*` and other operators regardless of how it's overloaded, because the compiler's parser resolves expression structure (precedence, associativity, arity) at a stage entirely separate from — and prior to — deciding which specific `operator+` implementation to actually call based on operand types. Allowing this to change would make code unparseable without knowing the types involved, breaking the language's grammar.

### Q2. Why must `operator<<` for custom stream output be a non-member (friend) function, and not a member of the class being printed?
**Answer:** A member function's left operand is always implicitly `*this` — the object of the class the member function belongs to. But in `cout << myObject`, the left operand is `cout` (an `ostream`), not `myObject` — there's no way to write a member function on the custom class that could ever be invoked with `ostream` as the implicit left operand, since member function calls are always dispatched based on the object appearing on the left of the dot (or here, structurally, the operator). The only way to define this operator to accept `ostream` on the left and the custom type on the right is as a free (non-member) function — declared as a `friend` specifically so it can still access the class's private members directly.

### Q3. Why does postfix increment take a dummy `int` parameter that's never actually used in the function body?
**Answer:** C++ needs some way to distinguish two overloads of the same operator symbol (`++`) used in two textually different but symbol-identical forms: `++c1` (prefix) and `c1++` (postfix). Since there's no other syntactic distinction available at the point of overload declaration, the language designers introduced a convention: the postfix version takes an extra, unused `int` parameter purely as a compile-time marker telling the compiler "this overload corresponds to the postfix form" — the parameter carries no semantic meaning and is never read inside the function.

### Q4. Why does prefix increment typically return a reference, while postfix increment must return a value (not a reference)?
**Answer:** Prefix increment (`++c1`) modifies the object and then represents *that same, already-modified* object as its result — returning a reference to `*this` is both correct and avoids an unnecessary copy. Postfix increment (`c1++`) must represent the object's value *before* the increment happened as its result, which requires saving a separate copy of the original state before any modification occurs — since `*this` itself has already been modified by the time the function returns, the function cannot return a reference to `*this` and expect it to reflect the pre-increment value; it must return the separately-saved copy by value instead.

### Q5. Why is checking for self-assignment (`if (this == &other) return *this;`) considered important in `operator=` for a class managing dynamically allocated resources, even though the example Complex class doesn't strictly need it?
**Answer:** For a class holding, say, a raw pointer to heap-allocated memory, a naive `operator=` implementation might first free the current object's existing memory and then copy data from the source object's memory. If the source and destination are actually the *same* object (`c1 = c1;`, or more subtly, through aliased references), freeing "the current object's memory" is the exact same memory the code is about to read from as "the source" — resulting in reading from (or copying from) memory that's already been freed, which is undefined behavior. Checking `this == &other` at the very start and returning early if they're the same object avoids ever reaching that dangerous free-then-read sequence in the self-assignment case. The example `Complex` class doesn't strictly need this because it has no dynamically allocated resources to free before copying — but any class doing manual resource management should always include this check.

### Q6. How does C++ distinguish between the same operator symbol being used as both a unary and a binary operator (e.g., `*` as dereference vs. multiplication, or `-` as negation vs. subtraction)?
**Answer:** C++ disambiguates purely based on the number of parameters (arity) the overloaded operator function declares. A unary operator overload (like dereference `*` or negation `-`) takes no explicit parameters beyond the implicit `*this` (for a member function) — it operates on a single operand. A binary overload (like multiplication `*` or subtraction `-`) takes one explicit parameter (as a member function) or two explicit parameters (as a friend function) — it operates on two operands. The compiler picks the correct overload based on how many operands actually appear at the call site (`-c1` has one operand → unary; `c1 - c2` has two → binary), matching against the declared parameter counts.

### Q7. Why is it good practice to implement compound assignment operators (like `+=`) in terms of the simple binary operator (`+`) and the assignment operator (`=`), rather than reimplementing the underlying arithmetic separately?
**Answer:** Reimplementing the same arithmetic logic in both `operator+` and `operator+=` duplicates code that computes the exact same mathematical result — any bug fix or logic change to how addition works would then need to be made in two separate places, and it's easy to update one and forget the other, causing the two operators to silently drift out of consistency with each other. Implementing `+=` as `(*this) = (*this) + c` reuses the already-correct `+` and `=` implementations, guaranteeing consistency by construction and reducing the actual arithmetic logic to a single source of truth.

### Q8. Scenario: A developer overloads `operator+` for a `Money` class as a member function only: `Money operator+(const Money& other) const`. Later, they want to support `100 + myMoneyObject` (an `int` on the left, `Money` on the right) in addition to the already-working `myMoneyObject + 100`. Explain why the member-function version doesn't support this, and what change is needed.
**Answer:** As a member function, `operator+` is always invoked as `leftOperand.operator+(rightOperand)` — the left operand must be an actual `Money` object for the member function call syntax to even apply, since member functions are called *on* an object. `100 + myMoneyObject` has an `int` literal as the left operand, and `int` has no `operator+` member function that knows how to combine itself with a `Money` object — there's no member function to dispatch to on the left side at all. The fix is to also provide (or replace it with) a free/friend function version of `operator+` that takes both operands explicitly as parameters (e.g., `friend Money operator+(int amount, const Money& m)`, possibly alongside an implicit conversion from `int` to `Money`, or simply an overload accepting that exact parameter order) — this is exactly the same underlying reasoning as needing a friend function for `2 + complexNumber`, or for `operator<<`: whenever the class you're overloading for isn't guaranteed to be the left operand, a member function alone cannot cover that case.
