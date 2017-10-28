---
title: Tutorial Part 1
layout: default
---

# Simple Instrumentation
## Instrumenting JavaScript Conditions
To understand TreeRegex expressions, let us work through a quick example - instrumenting the conditions in a JavaScript test file.  This tutorial should take about 5-10 minutes.

If you have not downloaded and compiled/installed TreeRegex, please do so now via [TreeRegex: How to Install][2].

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

The first step of using TreeRegex is adding the structure to the text - we can use an existing parser for this.  Run the command:

    node JavaScriptToSText.js test.js

This will print out the structured text.  Here we can see the form of the if-statements we want to instrument.  For example, here is the part of the first if-statement that we want to instrument (with the body ellided for brevity):

    (%if...

Any TreeRegex tree is a valid pattern that matches itself; if we used this as the pattern we could match this if-statement.  In this case, though, that is not enough.  We want to match any if-statement.  Let us look at  the other if-statement to see if we can determine the pattern we need.

The second if-statement looks like:

    (%if...

Both if-statements look something like this:

    (%if((%...%))(%...%)%)

Where the `(%...%)` is some subtree.

### Writing and matching Conditions with TreeRegex

In TreeRegex we can either specify the entire subtree with `(%` and `%)`, we can specify some (non-immediate) subtree with `(*` and `*)`, or we can just state that a subtree is there, without requiring any particular text in it with `@`.  Because we are not looking for a particular condition or particular body, we can use `@` here to get the following pattern:

    (%if(@)@%)

If we only wanted conditions with `Math.random()` in them, we could use:

    (%if((*Math.random()*))@%)

Let us keep both of these patterns in mind while doing the next step - adding in our instrumentation.

### Replacing Conditions with our instrumentation function



## Moving on to Better Things...

[1]: https://github.com
[2]: https://github.com
[3]: https://en.wikipedia.org/wiki/Tree_(data_structure)
[4]: https://en.wikipedia.org/wiki/Regular_expression
[5]: https://example.com
