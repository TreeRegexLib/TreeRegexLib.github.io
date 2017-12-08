---
title: Home
layout: default
---

# Introduction

Tree-structured text is widely used in many different software engineering and programming tasks. Examples of tree-structured text include various programming languages, domain-specific languages, and data formats such as XML and JSON. Users of tree-structured text often need to query patterns over such text and modify text.  For example, a user may want to query if the `eval` function has been called in the body of any function in a JavaScript program or rapidly prototype a compiler for a domain-specific language by modifying the abstract-syntax tree of a program.

There are several languages and associated tools that users often use to search patterns and to modify tree-structured text. Regular expressions are one such special formalism that is used to describe text patterns. They are widely used by programmers and computer scientists to concisely and elegantly describe search patterns over text. Most popular programming languages support regular expressions; some popular text-processing languages, such as `awk`, `sed`, and `perl`, were designed around regular expressions. A key reason behind the popularity of regular expressions is that they are compact and concise. Moreover, regular expressions can extract substrings from texts. This is particularly useful in extracting information and in modifying texts.

However, formal regular expressions are not expressive enough to describe patterns over text having a tree-like structure. For example, it is impossible to write a regular expression that matches a block of statements in a C-like language because a statement block can have nested blocks.

<!--*Context-free grammars (CFGs)* overcome the limitations of regular expressions by providing a more expressive formalism for describing patterns over tree-structured text. Although CFGs are strictly more powerful than regular expressions, they are not as compact and concise as regular expressions-pattern matching and replacing with CFGs requires the user to write an explicit program.-->

<!--Term-rewriting systems simplify the use of CFGs for text search and modification. These systems allow programmers to describe tree rewriting declaratively as a set of rules. Though term-rewriting systems have been found to be more convenient to use compared to traditional parsers and abstract-syntax tree (AST) visitors, they require a complete porting of the CFG of the given text to the term-rewriting system, which could be non-trivial for complex languages.-->

We  propose  a  natural  and  intuitive  extension  to  regular expressions, called `TreeRegex`, which can specify patterns over tree-structured text. A key insight behind the design of `TreeRegex` is that if we annotate a string with special markers to expose information about the string’s tree structure, then a simple extension to regular expressions can be used to describe patterns over the annotated string.  Based on this insight, we propose a two-step process for specifying and matching `TreeRegex` expressions against a tree-structured string. In the first step, we annotate the string by inserting parenthesis meta-characters `(%` and `%)` in the string. This annotated string, which resembles an S-expression in LISP, has *balanced* occurrences of `(%` and `%)` and is called a *serialized tree*. For example, `(%2+(%3∗4%)%)` is the annotated string for `2+3∗4`. The parenthesis meta-characters make the tree-structure of the string explicit. An existing parser and source-code generator (i.e., a tool that serializes an abstract-syntax tree to the original string) could easily be modified to generate an annotated string.

In the second step, we write patterns over the serialized tree as `TreeRegex` expressions. A `TreeRegex` expression is a regular expression extended with balanced `(%`, `%)`, balanced `(*`,`*)`, and the wildcard meta-character `@`. The exact motivation and semantics of these extra meta-characters are described in the following two sections. After describing a pattern in `TreeRegex`, we match the pattern against the serialized tree using an efficient algorithm, called `TreeRegexLib`.

`TreeRegex` has several key advantages.

1. It nicely decouples the CFG and parsing aspect of a string from the pattern expression and matching aspect. One can easily modify an existing parser in the first step to create a serialized tree—there is no need to port the CFG to our system.  We provide frontends that convert C/C++, Python, and JavaScript to serialized trees [here](#download)

2. `TreeRegex` is a natural extension to regular expressions, which we believe would be easy to learn if one is familiar with regular expressions.

3. Since `TreeRegex` is independent of the underlying CFG used to generate the serialized trees, it can be used to replace a sub-tree in a serialized tree with a sub-tree or string that does not conform to the original CFG.   We have taken advantage of this flexibility in a case study that generates `MIPS` assembly code from a simple language, based on the BC calculator language. For this compilation, our approach requires no information about the grammar of `MIPS`.

4. One can restore the original string from a serialized tree by dropping the parenthesis meta-characters `(%` and `%)`. This becomes useful while debugging `TreeRegex`.   We provide the command line utility `tstrip` to do this in our C++ `TreeRegexLib` implementation.
<!--5)`TreeRegex` matching and replacement algorithms can be implemented easily by using the API of an existing regular expression library. Thus, `TreeRegex` can be easily ported to many languages and this will allow programmers to use `TreeRegex` with their favorite languages to deal with tree-structured texts.-->

So far, we have implemented `TreeRegex` for C++, Java, and JavaScript programs as `TreeRegexLib`, we have released the C++, the most mature version [here](#download)

<!--We apply `TreeRegexLib` to five case studies: a tool for instrumenting JavaScript programs for branch coverage, a tool to prevent SQL injection vulnerabilities, a linter for JavaScript, a tool for finding errors in C programs, and a compiler from a BC-like language to `MIPS` assembly code. In our case studies, we found `TreeRegex` to be powerful enough for our tasks. We also found that we wrote significantly fewer lines of code while using `TreeRegexLib` compared to conventional AST-based techniques. Our experiments on the compiler for the BC-like language show that our `TreeRegexLib` implementation runs fast enough for practical usage—we can compile a 160kB file in less than 1 second.-->

# Overview of `TreeRegex`

We gently introduce `TreeRegex` through a series of motivating examples that we often encounter in program analysis and compiler construction.

*Motivating Example.* Suppose we want to check if the function `eval` has been called inside the body of any function in a JavaScript program. We may try to find such a usage of `eval` using a regular expression.  The following expression comes to mind:

    function .*(.*){.*eval(.*).*}

where `.` matches any character and `*` is the Kleene star operator (we do not treat parentheses as a meta-character here). Unfortunately,this regular expression does not work-it will match the following code, for instance:

    function  f1(x){bar ()}  eval(s);  function  f2() {}

This is because `.*` will match too much. Making the following slight modification to the regular expression prevents this problem:

    function .*(.*){[ˆ}]*eval(.*).*}

Here, we use `[ˆ}]`, a character class excluding curly braces, instead of the `.` meta-character. Unfortunately, this regular expression does not match

    function  f1(v){ { bar ()} eval(s)}

which should be matched—it has a call to `eval` in its body, preceded by a block with a call to `bar`. Both of these example regular expressions fail because they ignore the structure of the target expressions.  To find the pattern we are looking for, we must specify that `eval` can appear either at the top-level block or in some nested block of a function body.

<!--Patterns over structured text could be expressed using context-free grammars (CFGs). A standard technique to search for such patterns is to write a full-fledged CFG of the JavaScript language and then use a parser to convert a JavaScript program into an abstract-syntax tree (AST). The code pattern can then be searched by performing a programmatic traversal of the AST. This is the de-facto technique that various linters use to discover problematic code snippets. Unfortunately, this technique has a few disadvantages. First, we need to understand the structure of the AST enough to know where to look for the definition of a function and for the invocation of the `eval` function. Second, searching for a particular code pattern requires us to write a program that visits over the AST and explicitly looks for the identifier `eval` in a function definition sub-AST. Such code will span several lines and will not be as compact as a simple, single-line regular expression.-->

## `TreeRegex` and serialized trees

We propose a two-step technique to represent strings having tree-like structure and to express search patterns over them. In the first step, we convert a string into an annotated string which makes the tree structures of the string explicit. Specifically, we convert a string into an annotated string, similar to S-expressions in LISP, where the recursive structures are surrounded by the special parenthesis meta-characters `(%` and `%)`. For example, we convert the string `3∗4+5∗6`, denoting an arithmetic expression, into the annotated string `(%(%3∗4%)+(%5∗6%)%)`.  Such an annotated string has two important properties:
- An annotated string has balanced parentheses `(%` and `%)`. That is, each opening parenthesis `(%` has a later corresponding closed parenthesis `%)`, and the string between the pair of parentheses is again balanced.
- In an annotated string, if we remove the parenthesis meta-characters, we get back the original string.

The string between a pair of balanced parentheses denotes a structure that can have other nested structures. We call such annotated strings serialized trees. To convert a string into a serialized tree, one can use an existing parser and an AST-to-source code generator.  Programming languages and various structured data formats, such as XML, usually come with an implementation of a parser and an AST-to-source code generator, and they could be easily modified to annotate a string with `(%` and `%)`. <!--For example, we modified 108 out of 2298 lines of code in `esotope` code generator to generate serialized trees for JavaScript programs. We have also implemented a generic serialized tree generator for ANTLR grammars using 128 lines of Java code.--> A key advantage of converting a string into a serialized tree is that a simple extension to regular expressions can now be used to describe patterns over serialized trees.

The second step of our technique will be to check whether an annotated input string matches a desired pattern. To do so, we propose a simple, yet powerful, extension to regular expressions, called `TreeRegex`, to specify patterns over serialized trees. We next introduce `TreeRegex` gradually through a series of simple examples to demonstrate its intuitiveness.

In our examples, we assume that the inputs are strings denoting arithmetic expressions constructed using positive integers and arithmetic operators +,−,∗,/,(,). We assume that the strings have no space or newline characters. We also assume that the strings have been parsed and converted into serialized trees using a parser with standard precedence declaration for arithmetic operators. For simplicity of exposition and to reduce clutter, we assume that integer literals are not enclosed within `(%` and `%)`. For example, the arithmetic expression string `3∗4+5∗(6−2)` has been converted to the serialized tree `(%(%3∗4%)+(%5∗(%((%6−2%))%)%)%)`.

*Matching an exact serialized tree.* Let us first write a pattern that checks if an arithmetic expression is the addition of two positive integers. For example, `2+3+1` does not match this pattern, but `2+3` matches the pattern. Such a pattern can be easily written using the regular expression: `\d+\+\d+`. Here `\d` denotes the digit character class and `\d+` denotes one or more digits. Since `+` is a meta-character in regular expressions, we escape `+` with `\` to denote the actual `+` arithmetic operator. When matching against the serialized tree corresponding to an arithmetic expression, we need to use a `TreeRegex` expression. In `TreeRegex`, we extend regular expressions by allowing the usage of parenthesis meta-characters `(%` and `%)` in a balanced fashion. For example,

    (%\d+\+\d+%)

is a `TreeRegex` expression and it matches the serialized tree (e.g.`(%2+3%)`) corresponding to an arithmetic expression where two positive integers are being added. It is important to note that a regular expression in a `TreeRegex` expression cannot match the meta-characters `(%` and `%)`. Another example of a `TreeRegex` expression  that  matches  an  arithmetic  expression  is  one  that  denotes  the  addition  of  two  expressions,  where  each  expression is  the  multiplication  of  two  positive  integers.  This expression is: `(%(%\d+\*\d+%)\+(%\d+\*\d+%)%)`.  This `TreeRegex` expression matches the serialized tree `(%(%31∗4%)+(%5∗62%)%)` (i.e. serialized tree for `31∗4+5∗62`).

*Matching an arbitrary serialized tree.* So far, we have extended regular expressions with parenthesis meta-characters `(%` and `%)`- a `TreeRegex` expression with this extension has the form of a serialized tree. However, this extension is not enough if we want to match more complex patterns. For example, suppose we want to write a pattern that matches an arithmetic expression that is the addition of two arbitrary arithmetic expressions. We need a (sub-)`TreeRegex` expression that matches an arbitrary arithmetic expression. In the general case, we want a pattern that matches an arbitrary serialized tree beginning and ending with `(%` and `%)`, respectively. We add the meta-character `@` to `TreeRegex` to match such arbitrary serialized trees. A `TreeRegex` expression that matches the addition of two arbitrary arithmetic expressions could then be written as

    (%@\+@%)

Here `@` matches any serialized tree that starts with a `(%` and ends with a `%)`. Note that `@` cannot match any arbitrary string. This `TreeRegex` expression will now match `(%(%31∗4%)+(%5∗62%)%)`(i.e. serialized tree for `31∗4+5∗62`), and `(%(%2+3%)+(%1∗4%)%)`(i.e. serialized tree for `2+3+1∗4`). It will not match `(%2+3%)`,because `2` and `3` do not start with `(%` and end with `%)`. Note that we did not enclose an integer literal in `(%` and `%)` to illustrate this subtlety; our implementation does enclose integers in `(%` and `%)`.

*Matching a serialized tree nested in another serialized tree.* Now suppose we want to match any arithmetic expression that contains a specific form of nested sub-expression. The form we are looking for is an addition of two integers, and it can be nested arbitrarily deep inside the top-level expression. For example, the pattern should match both `(%(%2∗(%((%3+11%))%)%)∗1%)` (i.e. serialized tree for `2∗(3+11)∗1`) and `(%2+3%)`(i.e. serialized tree for `2+3`). We now need the ability to specify a pattern that matches a serialized tree that contains a serialized tree at an arbitrary depth. To do so, we add two more parentheses meta-characters `(*` and `*)`(inspired by Kleene star in regular expressions) to `TreeRegex`. A `TreeRegex` expression can use these meta-characters as long as the expression is balanced with respect to both `(%`,`%)`, and `(*`,`*)`. A pattern `(*t*)`, where `t` is some other `TreeRegex` expression, matches any serialized tree that contains a nested serialized tree matching `t`. With this new extension, the `TreeRegex` expression `(*\d+\+\d+*)` matches an arithmetic expression that has a nested arithmetic sub-expression that is the addition of two positive integers.

*Revisiting the motivating example.* We are now ready to write a `TreeRegex` expression that checks if a JavaScript program contains an `eval`-calling function body. First note that a function definition in a JavaScript program can be arbitrarily nested inside the program.  A call to `eval` can be arbitrarily nested within the body of a function as well. Therefore, we need two sets of `(*`,`*)`: one pair to match a function definition and another pair to match a call to `eval`. The `TreeRegex` expression for the pattern is

    (*function .*(@){(*eval(@)*)}*)

This pattern matches the serialized tree of a JavaScript program if the program has a function whose body calls `eval`. The first `@` matches the serialized tree for the list of parameters, and the second `@` matches the serialized tree for the argument to `eval`. Note that while `.*` matches an arbitrary string, it cannot match a serialized tree. Similarly, `@` matches an arbitrary serialized tree, but it cannot match any string.

## Capture Group Replacement
Most regular expression libraries provide support for search-and-replace via replacement strings. We provide support for similar search-and-replace operations in `TreeRegex`.

Let us consider the motivating example again. Now we want to replace the `eval` call with a `safe_eval` call. To do this, we make slight modifications on the `TreeRegex` expression as follows:

    (%function((.*))(@){(*eval(@)*)}%)

We simplify the example by replacing the outermost `*`-parentheses with `%`-parentheses, and add `((` and `))` to capture the name of the function. We use `((` and `))` to denote regular expression parenthesis meta-characters that capture. Now we need to capture four pieces of information: the name of the function, the formal parameters, the argument of the `eval` function call, and the text that surrounds the `eval` function call. The function name is matched and captured by the `((.*))`. In `TreeRegex`, we specify that the wildcard `@` captures the serialized tree it matches. This means the formal parameters and the argument passed to the `eval` function are captured. We can now build our desired replacement string

    (%function $1($2){... safe_eval($4)... }%)

where `$1` and `$2` refers to the first and second captured values, `$4` refers to the value captured by `@` (which is the argument passed to the `eval` function), and `...` are the missing strings that we are yet to specify.

The strings that surround the `eval` function call are more difficult to manipulate because there is no replacement string syntax in conventional regular expressions for inserting a string in the middle of a captured value.

We need to insert our new `safe_eval` function call between the strings that are to the left and right of the `eval` function call. Before that, though, we need to capture the strings to the left and right of the `eval` function call. In `TreeRegex`, we specify that an expression of the form `(*t*)` creates a capture group that captures a string with a hole. For example, if `(*eval(@)*)` matches the string `bar();foo(eval(s),2);`, then the `(*`,`*)` pair will capture the string `bar();foo(•,2);`, which has a hole `•`. This captured string will be referred by `$1` in this case. With this new definition of a capture group, we can specify our desired replacement string as

        (%function $1($2){$3(%safe_eval($4)%)}%)

In this replacement string, `$3` represents the strings surrounding the `eval` function call. This string has a hole. The question is what string do we use to fill the hole. In our approach, we fill the hole with the serialized tree that follows `$3` in the replacement string, which in our case is the string `(%safe_eval($4)%)` where `$4` is suitably replaced.

In summary, in `TreeRegex` a `@` captures a serialized tree string, and `(*t*)` captures a string with a hole. If in a replacement string, `$n` refers to a string with hole, then the hole is filled with the serialized tree string that follows `$n` in the replacement string.

<!--# Case Studies-->
<!--## Measuring JavaScript Test Coverage-->
# Tutorial
In this <!--case study-->tutorial, we use `TreeRegexLib` to instrument JavaScript programs  for  tracking  branch<!--and  statement --> coverage.  The  instrumentor has two parts: a JavaScript program to serialized tree converter, and a <!--list of transformers--> a `TreeRegex` expression and replacement string implementing the instrumentation. We built our converter on top of the `acorn` parser. <!-- and `esotope` JavaScript code generator by adding 108 lines of modification.-->

Our instrumentation program wraps the conditional expressions in various statements and expressions, such as `if-else`, `for`, `while`, and `switch`. For example, the code `if (x>0){x = 0;}` gets instrumented into `if (Cond(x >0)){x = 0;}`. <!--A unique static `id` is passed as the first argument to `Cond` and the conditional expression is passed as the second argument.--> An implementation of `Cond` records the branch being taken and returns the value of the conditional expression unmodified. <!--Similarly, we add a call to `Stmt` before every statement to track statement coverage.-->  This tutorial will walk through implementing this instrumentor.

<!--The  instrumentation program has  13 transformers implemented in 37 lines of JavaScript code. A simplified version of the `TreeRegex` expression and replacement string used to instrument an if-statement is shown below.

    (%if (@)@%)        (%if (Cond($3, $1)) $2%)

Here `$3` represents a static id which is generated and appended to the list of captures in the modifier of the `transformer`.

The instrumentation program was quite straight-forward to write. The total lines of code of the instrumentation program, which is 145 including the serialized tree converter, is significantly fewer than the 968 lines of code of the instrumentor in `istanbul`, a popular JavaScript coverage tool. Istanbul uses a similar parser, called `esprima`, and the `esotope` code generator. It programmatically visits the AST of a JavaScript program to perform the instrumentation.  We believe that such traversal code is tedious to write, debug, and maintain. Another important aspect of our instrumentation tool is that we did not use a specialized term-rewriting tool to perform instrumentation. Such a tool would simplify the task of writing an instrumentor; however, it would require one to define a grammar for JavaScript. We simply reused an existing parser and code generator. The ability to exploit existing tools for generation of serialized trees makes `TreeRegexLib` practical for real-world usage.--->

If you have not downloaded and compiled TreeRegex, please do so now via the directions [here](#download).

### Creating a test file
In order to instrument a JavaScript test file, we first need a JavaScript test file.  Open up your favorite text editor and create a `test.js` file with the following content:

~~~~
if(Math.random() > 0.5){
    if(Math.random() > 0.8){
        console.log("0.5*0.2 chance!");
    } else {
        console.log("0.5*0.8 chance!");
    }
} else {
    console.log("0.5 chance!");
}
~~~~

This file contains code that picks some random numbers and takes branches depending on those numbers.

To run this code, we can use `node`.  `node` is available [here](https://nodejs.org/en/) and must be installed before proceeding.  Once `node` is installed, the following should work on the command line (all commands meant to be run on the command line are prefixed with a `$ ` in this tutorial):

~~~~
$ node test.js
~~~~

This should print out the probability of the path taken.

### Changing a test file into a Tree

The first step of using TreeRegex is adding the structure to the text - we can use an existing parser for this.  After following the install commands in the `frontend/js` folder, run the command:

~~~~
$ node <path_to_treeregex_frontends>/js_to_sexp.js test.js
~~~~

This will print out the structured text into `test.js.sexp`.  Here we can see the form of the if-statements we want to instrument.  For example, here is the part of the first if-statement that we want to instrument (with the body elided for brevity):

    (%if((%(%(%(%Math%).(%random%)%)()%) > (%0.5%)%))(%{ ... }%) else (%{ ... }%)%)

Any TreeRegex tree is a valid pattern that matches itself; if we used this (with the bodies filled-in) as the pattern we could match this if-statement.  In this case, though, that is not enough.  We want to match any if-statement.  Let us look at  the other if-statement to see if we can determine the pattern we need.

The second if-statement looks like:

    (%if((%(%(%(%Math%).(%random%)%)()%) > (%0.8%)%))(%{ ... }%) else (%{ ... }%)%)

Both if-statements look something like this:

    (%if((%...%))(%...%) else (%...%)%)

Where the `(%...%)` is some subtree.

### Writing and matching Conditions with TreeRegex

In TreeRegex we can either specify the entire subtree with `(%` and `%)`, we can specify some (possibly non-immediate) subtree with `(*` and `*)`, or we can just state that a subtree is there, without requiring any particular text in it with `@`.  Because we are not looking for a particular condition or particular body, we can use `@` here to get the following pattern:

    (%if(@)@ else @%)

We can test this pattern using the `tsearch` binary like so:

~~~~
$ <path_to_treeregexlib_build>/tsearch '(%if(@)@ else @%)'  < test.js.sexp
~~~~

If we only wanted conditions with `Math.random` in them, we could use:

    (%if((*(%Math%).(%random%)*))@ else @%)

Let us keep both of these patterns in mind while doing the next step - adding in our instrumentation.

### Replacing Conditions with our instrumentation function

Now that we are able to match each of the if-statements, we might want to log what path is taken.  To do this, we are going to use the `Cond` function.  This function takes a single argument - whether or not the branch is taken - and prints it to the screen.

We can define it as follows:

~~~~
var n = 0;
function Cond(v){
	console.log("Branch " + n + " is taken?: " + v);
	n++;
	return v;
}
~~~~

Now, we want to insert a call to this function into each if-statement.  To do this, we take our matching pattern to find each of the if statements and then we replace them with a replacement string.  A replacement string in TreeRegex works similar to a replacement string in regular expressions - `$n` values are replaced with the `n`th capture.  On top of using parentheses to capture values, TreeRegex `@` expressions also capture the strings they match.  This makes the necessary replacement string simple to use:

We want to match with `(%if(@)@ else @%)` and replace with `(%if(Cond($1))$2 else $3%)`.

To test this, we can use the `treplace` program like follows:

~~~~
$ <path_to_treeregexlib_build>/treplace '(%if(@)@ else @%)' '(%if(Cond($1))$2 else $3%)' < test.js.sexp
~~~~

This produces the instrumented serialized tree.  We then only need to remove the ending markers.  We can do this with the `tstrip` command:

~~~~
$ <path_to_treeregexlib_build>/treplace '(%if(@)@ else @%)' '(%if(Cond($1))$2 else $3%)' < test.js.sexp | <path_to_treeregexlib_build>/tstrip
~~~~

The output of this command is the JavaScript with instrumentation.  Save this to a file and add in the definition of `Cond` at the top.  The resulting file looks like this:

~~~~
var n = 0;
function Cond(v){
	console.log("Branch " + n + " is taken?: " + v);
	n++;
	return v;
}

if(Cond(Math.random() > 0.5)){
    if(Cond(Math.random() > 0.8)){
        console.log("0.5*0.2 chance!");
    } else {
        console.log("0.5*0.8 chance!");
    }
} else {
    console.log("0.5 chance!");
}
~~~~

If we run this file, with `node` (e.g. `$ node instrumented_test.js`) we will get output like:

	Branch 0 is taken?: false
	0.5 chance!

This completes the tutorial of instrumenting JavaScript if-statements.


# Download
To use TreeRegex you need two applications: a frontend to convert source code (or other tree-structured text) into an annotated form and a TreeRegex implementation.

[To download a frontend, see our repo here.](https://github.com/TreeRegexLib/treeregex_frontends), requires Python or JavaScript, depending on the frontend.  Directions to install are available in the [README.md](https://github.com/TreeRegexLib/treeregex_frontends/blob/master/README.md)

To download an implementation, see our repos here:
* [C++ Library + CLI Tools](https://github.com/TreeRegexLib/treeregexlib_cpp), requires C++11 compatible compiler.  Directions to compile are in the [README.md](https://github.com/TreeRegexLib/treeregexlib_cpp/blob/master/README.md)
* Other versions coming soon!
<!--* ~~Java Library~~ *Coming Soon!*-->
!--* ~~JS Library~~ *Coming Soon!*-->
