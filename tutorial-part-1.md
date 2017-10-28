---
title: Tutorial Overview
layout: default
---
This tutorial teaches you all you need to know about TreeRegex source code modification.  The tutorial is broken into three parts:

- Part 1 discuses what TreeRegex expressions and replacement strings are and how to use them for simple instrumentation
- Part 2 discusses Transformers and how to use TreeRegex's for more complicated instrumentation
- Part 3 discusses using ANTLR grammars with TreeRegex and using TreeRegex expressions to evaluate a language

Throughout this tutorial we will be using the TreeRegex implementation available here [TreeRegex Download][1].  If you have not downloaded and compiled/installed TreeRegex, please first review [TreeRegex: How to Install][2]

### What is a TreeRegex?
TODO: fix up TreeRegex stands for, serialized expressions
Simply put, a TreeRegex, or a TreeRegex pattern, is a pattern describing a tree of text.  The name TreeRegex stands for Structured Tree Regex - a regular expression-like language over structured text, the structure of which are trees.  A [tree][3], for the purposes of TreeRegex, is either a list of sub-trees or a string.

Similarly, a TreeRegex pattern can be thought of as a tree of [regular expressions][4].  If you are not familiar with regular expressions, I recommend reading the tutorial [here][5] before continuing.

Any regular expression (once escaped) is a valid TreeRegex pattern.  For instance `.*` matches any tree that is just a string.  TreeRegex augments regular expressions by letting you specify the height (or depth) of the string in the tree.  For instance, if you wanted to find a tree that has one child and that child is any string, you would write the TreeRegex `(%.*%)`.  In TreeRegex patterns and trees, we use `(%` and `%)` to deliminate subtrees.

A TreeRegex tree is only made up of plain text and `(%` and `%)`, but a TreeRegex pattern has further metacharacters.  A TreeRegex pattern can also contain a `@` which matches any immediate subtree or `(*` and `*)` which matches a subtree, even if it is not an immediate subtree.  These are explained in *Instrumenting JavaScript Conditions* below.

### What is a Replacement String
While TreeRegex patterns provide a means of matching trees, it is of little use without the ability to modify those trees.  Similar to regular expressions, TreeRegex replacment strings specify what to replace a matched tree with.  These will also be trees.

TreeRegex replacement strings are trees that can have special metacharaters to represent captured values from the TreeRegex pattern.  These are a dollar sign ($) followed by a single digit (similar to some regular expression replacement string formats).

A simple replacement string could just be `Tree`, while a more complicated one may be `(%Tree: $1%)`.  These are further explained in the next section.

