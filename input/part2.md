# Nock for Everyday Coders, Part 2
# The Rest of Nock and Some Real-World Code

By `~timluc-miptev`

Twitter: [@basile_sportif](https://twitter.com/basile_sportif)

## Series Outline
* [Part 1: Basic Functions and Opcodes](part1.html)
* Part 2: The Rest of Nock and Some Real-World Code
* [Interlude: Loose Ends and FAQ](faq.html)
* [Part 3: Design Patterns and Real Programs](part3.html)

A word of encouragement: we're done with the hard part now. Every Nock function we learn in this section will be built from pieces in Part 1.

None of these new functions are *necessary* to make Nock work. All of them except Nock 10 and 11 are syntatic sugar that is part of the Nock definition, and must be built into every correct Nock implementation.  If you have seen macros or code expansions in other languages, that's another word for what's happening here.

Nock 10 makes it easier to replace a memory value somewhere in a tree, and Nock 11 allows passing hints to the interpreter.

One point of clarity: this syntactic sugar/code-expansion system is ***not*** extensible. This means that you can't invent your own Nock opcodes and still have that language be Nock; you're making a higher-level language on top of Nock at that point.

In fact, that's not a bad way to think of Hoon: it's a higher-level language that adds missing syntatic sugar/macros and human-readable names to Nock. (That's not the whole story, but it's a decent mental peg to initially hang Hoon on.)

***Nock is intentionally very, very small, such that you can always walk through and analyze what is happening in a block of code if you know Nock's opcodes.***

## Table of Contents
1. [An Opening Note about the `1` Function](#an-opening-note-about-the-1-function)
2. [`6`, "If/Else"](#6-ifelse)
3. [`7`, the "Composition" Opcode](#7-the-composition-opcode)
4. [`8`, the "Variable Adder" Opcode](#8-the-variable-adder-opcode)
5. [`9`, Create a Core and Run One of Its Arms](#9-create-a-core-and-run-one-of-its-arms)
6. [Real Nock Code](#real-nock-code)

## An Opening Note about the `1` Function
You're going to see the `1` opcode (the "Quoter") appear in a lot of examples below for a simple reason: the `6`-`9` opcodes expect formulas in a lot of places, not just atoms. And, as we've seen already, formulas have to be cells.

Whenever a formula is required, but you really just want to return a number, you use the quoter function.

### Example 1: Increment the Number 5
```
:: we just want to run 4 on the number 5, but 4 expects a formula after it, so we use [1 5]
~zod:dojo> .*(0 [4 1 5])
6
```

### Example 2: Compare a Memory Slot to a Number
```
:: we grab memory slot 2
:: then it has to be compared to the result of a formula
:: so we just use the formula [1 23] to return 23
~zod:dojo> .*([23 45] [5 [0 2] [1 23]])
0
```

## `6`, "If/Else"
We're into the ***"Sugar"*** part of Nock now. This means that all the functions in Part 2 will use only `*` and Nock code in their pseudocode. For example, here is the code for `6`, the "If/Else" function.
```
*[a 6 b c d]        *[a *[[c d] 0 *[[2 3] 0 *[a 4 4 b]]]]
```
### English Definition (Really Simple)
The English definition of what's happening is simple/boring *(refer to the left side of the definition above)*:
1. evaluate formula `b` against subject `a` (`*[a b]`) to see whether it's `0` or `1`
2. If `b` equals `0` ("true"), run formula `c` against `a`.
3. If `b` equals `1` ("false"), run formula `d` against `a`.
If `b` is not equal to `0` or `1`, the code crashes, for reasons we'll see in the code explanation.

### Code Expansion Explanation (Way More Fun)
OK, so that's the English version. The code explanation (the right side of the definition) is ***really fun*** now that we know the basic Nock opcodes.  

The pseuodocode has 4 nested `[subject formula]`s, so I'm going to unwrap those to the bottom, and then build it up again. The layers are, in order:

1. `*[a ...]`, i.e. subject `a` evaluated against that long formula starting with `*[[c d]...`
2. subject `[c d]` evaluated against the formula starting with `[[2 3...`
3. subject `[2 3]` evaluated against the lowest level formula
4. finally, subject `a` evalauted against formula `[4 4 b]`

#### Step 4
Remember, our English explanation was "see if `*[a b]` is true or false, and do different actions depending on that. So ***(4)*** is that check. 

Let's say we have `a` as `59`, and `b` just the quoted value `0` (true)
```
*[a 4 4 b]
:: a: 59
:: b: [1 0]
*[59 4 4 [1 0]]
:: substitute out the two increment operators
++*[59 [1 0]]
:: ignore subject, return quoted value 0
++(0)
2
```
Summary: because `*[a b]` evaluated to `0` ("true"), we get the number `2`. If `*[a b]` had been "false", we'd get `3` (because we'd evaluate `++(1)`.

What the heck? Why are we getting `2` or `3` back? How does that help us?? Well...

**Nock is going to use that returned 2 or 3 to represent a *memory slot*, and executes the code in that *memory slot***.

#### Step 3
Now we go up to the next level, ***(3)***, with the subject `[2 3]` evaluated against our return value from ***(4)***. Let's imagine that `2` had been returned:
```
remember, "result-of-step-4" was 2
*[[2 3] 0 result-of-step-4]
*[[2 3] 0 2]
:: get memory slot 2
2
```
OK, so now we are getting feel. It grabs memory slot `2` if `*[a b]` was true, and memory slot `3` if `*[a b]` was false.

Wait, isn't this redundant? We are just using our `2` or `3` generated in step ***(4)*** to generate a `2` or a `3`. Seems dumb.

The answer is that we are ***making sure to crash if `*[a b]` yields a non-true/false value***. If `*[a b]` returned `10` (for example), we'd have the following code in step ***(3)***:
```
*[[2 3] 0 10]
:: lol idiot there's no slot 10
CRASH!!
```
This is exactly what we want: the program crashes unless we are doing a boolean test that returns a `0` or `1` (converted to a `2` or `3`) in step ***(4)***.

#### Step 2
We now we have our validated `2` or `3` to plug into step ***(2)***. Let's imagine `c` is the simple formula `[0 1]` and `d` is `[1 203]`
```
:: if step 3 returned "2" (true)
*[[[0 1] [1 203]] 0 2]
[0 1]
```
Not much to see here, we just grab memory slot `2` or `3` depending on whether our initial `b` was true or false.

#### Step 1
And now we're back at the top level, where we just use whichever formula we yoinked in step ***(2)*** and run it against `a`
```
*[a formula-from-step-2]
:: let's say we returned formula [0 1]
:: our original a, from step 4, was 59
*[59 [0 1]
59
```

### Example Code Expansion of `6`
```
~zod:dojo> .*(1 [6 [0 1] [0 1] [4 0 1]])
:: PSEUDOCODE
:: expansion: *[a *[[c d] 0 *[[2 3] 0 *[a 4 4 b]]]]
*[1 *[[[0 1] [4 0 1]] 0 *[[2 3] 0 *[1 4 4 [0 1]]]]]
:: factor out the 4 opcodes
*[1 *[[[0 1] [4 0 1]] 0 *[[2 3] 0 ++*[1 [0 1]]]]]
:: b evaluates to 1 (yank memory slot 1)
*[1 *[[[0 1] [4 0 1]] 0 *[[2 3] 0 ++(1)]]]
:: evaluate the two increments
*[1 *[[[0 1] [4 0 1]] 0 *[[2 3] 0 3]]]
:: get memory slot 3 from [2 3]
*[1 *[[[0 1] [4 0 1]] 0 3]]
:: [0 3] means get memory slot 3 from the subject (formula [4 0 1])
*[1 [4 0 1]]
:: factor out the 4 opcode
+*[1 [0 1]]
+(1)
2
```

### Summary of `6`
Now I'm going to make you a little sad. Most Nock interpreters don't do this whole awesome code expansion. They just see `6`, and implement an if/else check with a crash if `*[a b]` isn't a boolean.

However, the pattern of storing chunks of code in memory and pulling them out when you want them is the most important part of Nock. In fact, those of you who know Hoon are probably already seeing the makings of something that begins with "c" and rhymes with "zor"...

## `7`, the "Composition" Opcode
`7` is so simple we barely need to spend time on it. All it does is create a new subject/environment (using `2`), and immediately runs a formula against that new subject. Let's compare it to `2`:
```
::opcode 7
*[a 7 b c]          *[*[a b] c]

:: opcode 2
*[a 2 b c]           *[*[a b] *[a c]]
```
Literally exactly the same except with `c` instead of `*[a c]`. This means that you can write a "subject-changing" formula in `b`, and then run a simple function against that in `c`.

### Example: Opcode `2` vs Opcode `7` for increment
```
:: using opcode 2
~zod:dojo> .*([23 45] [2 [0 3] [1 4 0 1]])
46

:: using opcode 7 -- we get to remove the quoter opcode "1"
:: wow amazing /sarcasm
~zod:dojo> .*([23 45] [7 [0 3] [4 0 1]])
46
```

`7` is clearly trivial, so why does it exist? It allows clean expression of function composition: this is really `c(b(a))`, where `a` (the subject) is the initial argument, and `b` and `c` are functions. This pattern comes up a ton, and it's nice to not use `1`s everywhere, I guess.

## `8`, the "Variable Adder" Opcode
`8` is what you want if you're ever writing Nock and think "how can I add a new variable to the subject?" The new variable can be based on either the existing subject, or be a new value you add on.
```
*[a 8 b c]          *[[*[a b] a] c]
```
This is saying to run `*[a b]`, and then make that the head of a new subject, with the old subject `a` as the new subject's tail.

### Example 1: Add Variable as a Copied Value from an Existing Subject
In English, the below code first yanks the variable from memory slot 3, copies it to the head of a new subject, and then increments the value in that new head.
```
~zod:dojo> .*([67 39] [8 [0 3] [4 0 2]])
40
:: PSEUDOCODE
*[[*[[67 39] [0 3]] [67 39]] [4 0 2]]
:: yanks mem slot 3 and pins it to the front of the old subject
*[[39 [67 39]] [4 0 2]]
+*[[39 [67 39]] [0 2]]
+(39)
40
```

### Example 2: Add Variable as a New Value
```
~zod:dojo> .*([67 39] [8 [1 0] [4 0 2]])
1
:: PSEUDOCODE
*[[*[[67 39] [1 0]] [67 39]] [4 0 2]]
*[[0 [67 39]] [4 0 2]]
+*[[0 [67 39]] [0 2]]
+(0)
1
```

Why do we want this? In Example 1, we pin a copy of a value so that we can manipulate it without changing the original. In Example 2, we add a `0` to the front; maybe we want to increment it until some condition is met?

The above should be starting to feel **very** Hoon-ish: we're a minor code transform away from pinning new values to the head of a payload.

## `9`, Create a Core and Run a Stored Procedure Arm inside It
We're almost at full Hoon now, although still at a very low/raw level. `9` looks a little complicated...
```
*[a 9 b c]       *[*[a c] 2 [0 1] 0 b]
```
...but in English, this is just saying
1. Use formula `c` to make a new subject from `a` (`*[a c]`)
2. Grab the formula located at memory slot `b` in that new subject
3. Run that formula against the new subject (`*[a c]`)

The fact that the above description has the words "make a new subject" tells us right away that it's syntactic sugar for opcode `2`, and that's indeed what we see in the pseudocode.

### Example: An Incrementer Arm
The initial expression here likely looks *very* cryptic, but if you follow the pseudocode, it's pretty clear.

To stay oriented, remember that, if we're thinking of `9` as `*[a 9 b c]`, then
```
a: 45
b: 2
c: [[1 4 0 3] 0 1]
```
```
~zod:dojo> .*(45 [9 2 [1 4 0 3] 0 1])
46
:: PSEUDOCODE
*[45 [9 2 [1 4 0 3] [0 1]]]
*[*[45 [1 4 0 3] [0 1]]
   2 [0 1] 0 2]
:: new subject is [[4 0 3] 45]
:: that is a formula to increment mem slot 3 in the head; 45 in the tail
*[[[4 0 3] 45] 2 [0 1] 0 2]
:: now we expand opcode 2
*[*[[[4 0 3] 45] 0 1]
  *[[[4 0 3] 45] 0 2]]
:: mem slot 2 of the subject becomes the new formula
*[[[4 0 3] 45] 4 0 3]
+*[[[4 0 3] 45] 0 3]
:: grab mem slot 3
+(45)

46
```
We start above with a subject that is not a core; it's just the atom `45`.  The code for `c` is then:
```
[[1 4 0 3] [0 1]]
```
This formula uses the Cell Maker (Distribution Rule) to insert `[4 0 3]` as the head of the new subject, and puts mem slot `1` of the old subject as the tail, so we get a new subject of `[[4 0 3] 45]`.

Then essentially what we do is use `b` (`2` here) to select mem slot `2` from that new subject. Mem slot `2` is a formula: `[4 0 3]`.We run that formula against the new subject.

### Key Point/Possible Confusion
When I first say `9`, I thought: "why didn't they just use `2` to make the transformed subject? Why is there an extra step to use `[0 1]` to pull the new `*[a c]` subject? Why can't we do `*[a 2 c [0 b]]`?

The answer is that we actually use the `*[a c]` subject *twice*:
1. we extract from it the formula located at memory slot `b`
2. then we run *that* formula against the `*[a c]` subject

If we just did `*[a 2 c [0 b]]`, then `[0 b]` would try to look up the `b` memory slot in `a`, NOT `*[a c]`. So `9` gives us one extra step to set up the core itself.

### How to Think of `9`
I like to think of `9` as having two parts:
1. `c`: our "set up the subject" formula. This makes a new subject
2. `b`: the memory slot in our new subject where an arm is
And then we run the arm located at `b` against the new subject.

You can think of this as setting up a subject, pulling a stored procedure from it, and then running that procedure against the subject.

## `10`, Replace a Memory Slot
Before explaining `10`, we need to introduce a new operator, `#`.
`#` is the "edit" operator. It has the form
```
#[mem-slot new-val target-tree]
```
It replaces the memory slot `mem-slot` in `target-tree` with `new-val`.
```
::  Example
#[2 [4 5] [99 88 77]]
[[4 5] 88 77]
```

Pseudocode for `10`
```
*[a 10 [b c] d]      #[b *[a c] *[a d]]
```
In English, this first calculates `*[a c]` and `*[a d]`, and then replaces memory slot `b` in the latter with the result of the former.

### Example
```
~zod:dojo> .*(50 [10 [2 [0 1]] [1 8 9 10]])
[50 9 10]
::  PSEUDOCODE
*[50 [10 [2 [0 1]] [1 8 9 10]]]
#[2 *[50 0 1] *[50 1 8 9 10]]
#[2 /[1 50] [8 9 10]]
#[2 50 [8 9 10]]
::  expanded as far as we can, now do the edit
::  replace mem slot 2 of [8 9 10] with the value 50
[50 9 10]
```

## `11`: Save for Later
I cover `11` in the [Interlude](faq.html), because it's not part of strict Nock semantics. You can go there to read about it.

### Hoon Time!
If "grab a chunk of code from a subject and then run it against the subject" sounds a lot like getting an arm from a core, that's because it is.

The "new subjects" created by `*[a c]` are cores, and the things selected from memory slots by `b` are arms.

## Real Nock Code
To finish this off and take your new powers for a spin, let's look at some real Nock code from the wild.
I got the below example from the Dojo. It's a mold: a function that takes a noun and returns it if it's the correct type, and crashes if not. The mold here checks whether the input noun is a boolean (`0` or `1`).

(To see more examples of basic molds, go to the [Hoon Tutorial 2.3](https://urbit.org/docs/tutorials/hoon/structures-and-complex-types/)). 

### A Boolean Mold
```
~zod:dojo> =boolean-mold ?

:: below output shortened for our purposes here
~zod:dojo> boolean-mold
< 1.toz ... >

::  grab the code for the battery
~zod:dojo> -.boolean-mold
[6 [5 [1 0] 0 6] [1 0] 6 [5 [1 1] 0 6] [1 1] 0 0]
```
So we
1. assign the mold-gate `?` to the face `boolean-mold`
2. take a look at what's inside it...looks like the head is an arm...let's print out its source code!
3. print out that source code by calling the head

#### Code Walkthrough
So, we know the below code is a gate that evaluates its sample, and returns it if it's a boolean, and crashes otherwise. Let's see how that works.
```
[6 [5 [1 0] 0 6] [1 0] 6 [5 [1 1] 0 6] [1 1] 0 0]
```
We start with `6`, which means this is an if-else. The true/false test is the next element:
```
[5 [1 0] 0 6] 
```
This compares the quoted value `0` (from `[1 0]`) with the value at memory slot `6` in the subject (`[0 6]`).

What is the subject and what is at mem slot `6`? This code is the arm of a gate, so the subject is that gate/core: `[battery sample payload]`. We are looking at the battery right now, and so `[0 6]` yanks the head of the tail...the `sample`!

We know the sample will be a noun that we're testing for booleanness, so this code starts by seeing whether the sample is the value 0. If it is, the next element is `[1 0]`, so we return `0` if the sample is `0`.

Otherwise, we run the 2nd branch of the if-else:
```
[6 [5 [1 1] 0 6] [1 1] 0 0]
```
This is also an opcode `6` if-else. once again, it compares something to the value at mem slot `6` (the sample), but this time it checks whether that is the value `1`.  If it is, it runs the formula `[1 1]` to return `1`.

If not, it means our input was neither `0` nor `1` and is not a boolean, so we run the formula `[0 0]`, which always crashes (there is no  memory slot `0`). This is exactly what we want--crash if the sample is not a boolean!

## Summary
In Part 2, we've seen how to use all the remaining opcodes, which build Nock up to a slightly more expressive level. We also saw how Hoon cores start to arise pretty naturally out of the Nock primitives, especially `8` and `9`. Then we walked through real production Nock code to show that everything we've learned so far works exactly as expected in the wild.

In Part 3, you'll learn how to write real programs in Nock, and compose those programs to make new ones.

Finally, hopefully you now see that, whatever its other limitations, Nock is not particularly obscurantist, and is fairly straightforward to parse, once you understand its syntax and idioms.
