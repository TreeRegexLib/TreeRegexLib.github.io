---
title: Tutorial Part 1
layout: default
---

# Simple Instrumentation
## Instrumenting JavaScript Conditions
To understand TreeRegex expressions, let us work through a quick example - instrumenting the conditions in a JavaScript test file.  This tutorial should take about 5-10 minutes.

If you have not downloaded and compiled TreeRegex, please do so now via the directions [here](download.md).

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

To run this code, we can use `node`:

    node test.js

This should print out the probability of the path taken.  Otherwise, either `node` is not installed (which is necessary for this tutorial) or there is an error in the JavaScript file.

### Changing a test file into a Tree

The first step of using TreeRegex is adding the structure to the text - we can use an existing parser for this.  After following the install commands in the `frontend/js` folder, run the command:

    node <path_to_treeregex_frontends>/js_to_sexp.js test.js

This will print out the structured text into `test.js.sexp`.  Here we can see the form of the if-statements we want to instrument.  For example, here is the part of the first if-statement that we want to instrument (with the body ellided for brevity):

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
<path_to_treeregexlib_build>/tsearch '(%if(@)@ else @%)'  < test.js.sexp
~~~~

If we only wanted conditions with `Math.random` in them, we could use:

    (%if((*(%Math%).(%random%)*))@ else @%)

Let us keep both of these patterns in mind while doing the next step - adding in our instrumentation.

### Replacing Conditions with our instrumentation function

Now that we are able to match each of the if-statements, we might want to log what path is taken.  To do this, we are going to use the `logBranch` function.  This function takes a single argument - whether or not the branch is taken - and prints it to the screen.

We can define it as follows:

~~~~
var n = 0;
function logBranch(v){
	console.log("Branch " + n + " is taken?: " + v);
	n++;
}
~~~~

Now, we want to insert a call to this function into each if-statement.  To do this, we take our matching pattern to find each of the if statements and then we replace them with a replacement string.  A replacement string in TreeRegex works similar to a replacement string in regular expressions - `$n` values are replaced with the `n`th capture.  On top of using parentheses to capture values, TreeRegex `@` expressions also capture the strings they match.  This makes the necessary replacement string simple to use:

We want to match with `(%if(@)@ else @%)` and replace with `(%if(logBranch($1))$2 else $3%)`.

To test this, we can use the `treplace` program like follows:

~~~~
<path_to_treeregexlib_build>/treplace '(%if(@)@ else @%)' '(%if(logBranch($1))$2 else $3%)' < test.js.sexp
~~~~

This produces the instrumented serialized tree.  We then only need to remove the ending markers.  We can do this with the `tstrip` command:

~~~~
<path_to_treeregexlib_build>/treplace '(%if(@)@ else @%)' '(%if(logBranch($1))$2 else $3%)' < test.js.sexp | <path_to_treeregexlib_build>/tstrip
~~~~

This completes the tutorial of instrumenting JavaScript if-statements.


<!--## Moving on to Better Things...
~~[Part 2](tutorial-part-2.md) describes how to use the *Transformer* API to combine different TreeRegex expressions for easy rewriting.~~ *Coming Soon!*-->
