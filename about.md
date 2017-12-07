---
title: About
layout: page
---
<!--
# Regular Expressions

Regular expressions are a special
formalism that is used to describe text patterns. They are widely used
by programmers and computer scientists to concisely and elegantly
describe search patterns over texts. Most popular programming
languages support regular expressions; some popular text-processing
languages, such as `awk`, `sed`, and `perl`, were
designed around regular expressions. A key reason behind the
popularity of regular expressions is that they are compact and
concise. Moreover, regular expressions can extract substrings from
texts. This is particularly useful in extracting information and in
modifying texts.

However, formal regular expressions are not expressive enough to describe
patterns over texts that have tree-like structures.  Tree-like structures
are common in text - from bullet points to programming source code.  For example, it
is impossible to write a regular expression that matches a block of
statements in a C-like language because a statement block can have
nested blocks.


# TreeRegex-->
Researchers at UC Berkeley created TreeRegex to make quick source code analysis easy.  It was created to use syntax that most developers already know - regular expressions - and to be applicable to artifacts that developers create - source code (which is a tree-structured text).
<!--The goal is to create a language that is as easy to write as regular expressions, but powerful enough for tasks like matching and transforming source code.-->
TreeRegexLib is a prototype towards this end.  This prototype was initially developed as a means of easily instrumenting source code.  During the development of TreeRegex, we realized that the tool could be used for other tasks and did preliminary exploration for using TreeRegex for linting and compilation.  We hope to further strengthen the expressive power of TreeRegex through additions to the language and apply TreeRegex to a series of applications.
