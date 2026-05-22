---
layout: post
title:  "[Learning Notes] Rust Ownership and Expressions"
categories: learning
---

From reading chapters 3 and 4 of [The Rust Programming Language](https://doc.rust-lang.org/book/).

**Q: Why is immutability desired on computer programs?**

Because it's easier to reason about (mutable objects can change in different parts of the codebase) and because immutable objects are inherently thread-safe.

**Q: What is the difference between statements and expressions in Rust?**

*Statements* are instructions that perform some action and do not return a value; *expressions* evaluate to a resultant value.

**Q: What is the semicolon's role when it comes to expressions and statements in Rust?**

Semicolons turn expressions into statements:

{%highlight rust%}
// y gets 4
let y = {
    let x = 3;
    x + 1
};

// y gets (), the unit type
let y = {
    let x = 3;
    x + 1;
}
{%endhighlight%}

**Q: What does it mean to say that Rust is an expression-based language?**

It means that "most forms of value-producing or effect-causing evaluation are directed by the uniform syntax category of *expressions*."

Furthermore, "each kind of expression can typically *nest* within each other kind of expression, and rules for evaluation of expressions involve specifying both the valuer produced by the expression and the order in which its sub-expressions are themselves evaluated."

From [Statements and expressions - The Rust Reference](https://doc.rust-lang.org/reference/statements-and-expressions.html).

**Q: What is the `Copy` trait in Rust?**

`Copy` indicates types whose values can be duplicated simply by copying bits.

**Q: What is the relationship between the `Copy` and `Drop` traits in Rust?**

If a type has implemented the `Drop` trait, Rust won't let us annotate it with `Copy`.

**Q: What is ownership in Rust?**

Rules that govern how a Rust program manages memory:
1. Each value has an owner.
2. There can be only one owner at a time.
3. When the owner goes out of scope, the value will be dropped.

**Q: What happens when values in the heap are passed as arguments in Rust?**

{%highlight rust%}
fn main() {
    let s = String::from("hello");
    takes_ownership(s);
}

fn takes_ownership(some_string: String) {
    println!("{some_string}");
}
{%endhighlight%}

They are moved, as if in an assignment. In this program, if we were to use `s` after `take_ownership(s)`, Rust would throw a compile-time error.
