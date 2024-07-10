---
layout: post
title: Creating own programming language. Part 1
date: 2024-07-10
tags: meraki
---

I started this project 7 months ago and in this post I'd like to talk about my
journey and what I've done already. First of all I'd like to let you know that
it's written in thread safe, fearless concurrency, BLAZINGLY FAST 🔥🔥🔥 Rust
programming language, and the source code is available on [GitHub](https://github.com/MilkeeyCat/meraki).
Before I tried to make my own shit, I read an amazing book, called [Interpreter book](https://interpreterbook.com).
It gave me a basic understanding of how programming languages work, who
could've thought that it's not a magic black box made by magicians but it's a
bunch of statements which contain statements and expressions inside of them(but
if you look at C++ internals that shit's definitely made not by humans).
Expressions are units of code which produce a value, wheres statements don't
produce any value. For example `return 5;` is a statement, it doesn't produce
any value, wheres `5` is an expression and produces interger literal value 5.

My compiler consists of 3 phases:
- Lexical analysis
- Syntax analysis
- Code generation

But compilers used in production have many more phases, which can include
generating IR[^1] or code optimization for example.

## Lexical analysis
Lexer was the first thing I wrote and it is the easiest part of the whole
compiler. All its code fits in under 200 loc. Its purpose is to transform raw
dawg text that programmer writes into an array of tokens which will be
processed later by parser. For example when programmer writes a while loop,
keyword `while` will be transformed into a token called `WHILE` and so on, also
lexer skips lines which start with `//`. Full list of tokens can be found
[here](https://github.com/MilkeeyCat/meraki/blob/main/src/lexer/token.rs).
There's no really more to it.

## Semantic analysis
Unlike lexical analysis, semantic analysis is the hardest part in the compiler
xd. That's the place where all the big boi stuff is happening. Firstly, it
checks the semantics of the language, eg. if you use statements correctly and
if you have curlies and semis at the right places. Also it transforms list of
tokens produces by lexer in AST[^2] nodes.

Parser is also the place where symbol tables and type tables are made(next few
sentences can be complete horseraddis but that's how I implemented it
¯\\\_(ツ)_/¯). Symbol table is a structure which holds existing variables and
functions in current scope. So when you declare a new variable or function it
adds the symbol to corresponding symbol table, it's also used to look up the
place of a variable when you need to get a value of it, or return an error if a
variable doesn't exist. And type type does exactly the same but instead of
storing symbols, it stores user defined types. After fucking with these tables,
trying to manage them separately, in the end I just put 'em in a structure
called `Scope`, and there's one global scope which is returned from parser but
also every function and every scope inside these functions contain its
individual scope.

```c
// Global scope
int foo[42];

int main() {
    // Function scope
    {
        // Scope inside function scope
    }
}
```

## Code generation
The last part of the compiler is code generation, it takes AST nodes generated
by parser and produces ready output. My compiler as an output spits out
assembly language code which later can be assembled using GNU assembler[^3] and
linked in an executable. The only remotely hard part of it was to learn
assembly language itself because I never used it before :D. It took me a few
hours to understand how to use `idiv` instruction, all the explaining websites
were using fucking hieroglyphs to explain how that shit works. Though I could
generate 0s and 1s, I thought it wouldn't really teach me anything so I chose
to generate assembly instead.

## Features already added to the compiler
So far I've impemented a few primitive data type like: `u8`, `i8`, `u16`,
`i16`, `bool` and `void`. There's also support for mafs operators like `+`,
`-`, `*`, `/` and comparison operators. Also it supports casting values and
basic functions. For now there's also only 1 control flow statement and it's
`return`, I don't think it will be hard to add if expressions or loops, so
right now I'm trying to add support for extern functions. Since I'm
implementing x86-64 SysV ABI[^4] I will be able to call C functions from my
code and vise versa, it's so cool!

<hr/>

[^1]: [https://en.wikipedia.org/wiki/Intermediate_representation](https://en.wikipedia.org/wiki/Intermediate_representation)
[^2]: [https://en.wikipedia.org/wiki/Abstract_syntax_tree](https://en.wikipedia.org/wiki/Abstract_syntax_tree)
[^3]: [https://ftp.gnu.org/old-gnu/Manuals/gas-2.9.1/html_node/as_toc.html](https://ftp.gnu.org/old-gnu/Manuals/gas-2.9.1/html_node/as_toc.html)
[^4]: If you click on this link it will download .pdf file 😬 [https://gitlab.com/x86-psABIs/x86-64-ABI/-/jobs/artifacts/master/raw/x86-64-ABI/abi.pdf?job=build](https://gitlab.com/x86-psABIs/x86-64-ABI/-/jobs/artifacts/master/raw/x86-64-ABI/abi.pdf?job=build)
