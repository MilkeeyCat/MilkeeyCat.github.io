---
layout: post
title: Meraki, 3 months later
date: 2024-10-14
tags: meraki
---

It's been 3 months since I've written something here, and I've added a lot of
new features to the language. In this time it transformed from being a fancy
calculator to a real programming language, not a very good or robust one, but
hey, nobody's perfect.

### Syntax changes
Language's syntax was completely changed because it was impossible to parse
after I decided let the user use structs/types which are declared after the
place they are used in.

To better understand the problem, here's a question: What is `a`?<br/>
a) A type which is declared after `main` function<br/>
b) A global variable declared after `main` function<br/>
c) 69
```c
int main() {
    a * c;
}
```
Welp, there was no a correct answer, this code can be interpereted in 2 ways:
either it's a multiplication expression between `a` and `b` or declaration
of variable `c` with type `a*`. The easiest way to solve this problem is to
change the language's syntax. I like the look and feels of rust's, so I yoinked
it, and voíla, new syntax:
```rust
fn main() -> u32 {
    let c: *a;
    // you can see the different between the two, right?
    a * c;
}
```

### Functions
```rust
fn bar() -> u8; // You can also make function declarations
fn foo(arg: *usize) -> u32 {
    return 69;
}

fn main() -> u8 {
    let num: usize = 10;
    let bar: u32 = foo(&num);

    return bar as u8;
}
```
That looks like Rust with pointers instead of borrows.

### Structs
```rust
struct Bar {
    value: u8;
}

struct Foo {
    bar: Bar;

    fn gimme_bar() -> *Bar {
        return &this->bar;
    }
}

fn main() -> u8 {
    let foo: Foo = Foo {
        bar: Bar {
            value: 42
        }
    };
    let bar_ptr: *Bar = foo.gimme_bar();

    return bar_ptr->value;
}
```
My idea with structs was take them as they are in C and add methods to them.
Currently there're only non static methods, which means they have access to
instance of the struct using `this` keyword, Im planning to also add support for
static methods which can be used for creating instances of structs for example.

### Tree-sitter grammar
To make my life easier I also wrote a tree-sitter grammar for the language which
is available [here](https://github.com/MilkeeyCat/tree-sitter-meraki). After
randomly pasting `prec.left/right` to resolve parsing issues I have a syntax
highlightning for the code:

<img src="/assets/images/code_highlighting.png"/>

### Assembly skill issue story
It was my first time trying to make somewhat real program in the language, I
wrote a few functions and everything was working perfectly, but then for some
god unknown reason the program was crashing with such a message:
```
[1]    174626 floating point exception  ./a.out
```
After few minutes of removing random lines. I found the line which was causing
the creash, there was such an expression `10 + 30 / 15 - 20`. It doesn't look
like a problem at all I thought to myself. I took a look at assembly version
of this code:
```
...
mov rax, 30
mov rdx, 15
cqo
idiv rdx
...
```
After 20 mins staring at this I realized that when [`cqo`](https://www.felixcloutier.com/x86/cwd:cdq:cqo)
instruction is called, `rax` will be sign-extended to `RDX:RAX`. So it was
basically set to 0, and that's why it was crashing :\ Lesson learnt.
