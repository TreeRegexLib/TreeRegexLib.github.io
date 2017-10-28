---
title: TreeRegex
layout: default
---

# What is TreeRegex?

TreeRegex is an extension of regular expressions to handle tree-like structures in text.  Regular expressions are already a standard tool in the computer programmer's toolbox - they are a required technology to know in modern software.  Yet, they can't be used effectively with a huge class of texts: tree-structured texts.  Tree structured text includes a lot of different cases, the most prominent among them are *programming languages*, JSON, and XML-languages.  TreeRegex is designed to be used with any tree structured text and we provide tooling to get started using it in a variety of cases.  In short, we made regular expressions work on source code.  Now if you want to do a complicated refactor or find a set of instructions, you can do so using a small adjustment to a tool you already know.  *TreeRegexLib* is our reference implementation.

# A Taste of TreeRegex

TreeRegex operates in four steps:

1. Run a frontend to annotate the text to make the structure explicit (we provide a variety already - including for C/C++ and Java)
2. Write and run a TreeRegex on the annotated text.  The TreeRegex can perform find (and replace) tasks.
3. De-structure it into text
4. Profit...

TODO: wide image that goes from code to sexp, with matching TreeRegex expression

# Benefits

* Fast: TreeRegexLib (our reference implementation) performs hundreds of times faster than Perl regular expressions for comparable tasks!

* Easy to use: TreeRegex doesn't require (re)writing a grammar or porting your entire build system to an esoteric build environment.

* Quick to get started: Because TreeRegex is based heavily on regular expressions, we believe that anyone who is familiar with regular expressions can easily get started in using it.

* Powerful: We are able to quickly implement instrumentation of if-statements or simple code transformations (as shown in the [[Tutorial]]).

* Reliable: One of the fundamental difficulties with making tooling for a language is that the language changes - new features or added or internal components are rearranges.  By focuses on the outer-most layers of a tool chain (the syntax itself), we are able to build tools that work for different versions of the language and different versions of the underlying tool chain (e.g., different versions of clang, even as the API is adjusted).

If you want to learn more, continue to the [Tutorial](tutorial-overview.md)
