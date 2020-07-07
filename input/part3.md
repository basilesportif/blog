# Nock for Everyday Coders, Part 3
# Nock Design Patterns
## Intro
After parts 1 and 2, you now know everything there is to know about Nock syntax and basic code manipulation.

However, if I asked you to go write a simple program in Nock, you'd probably have a hard time getting started. It's really helpful to see some examples of how Nock programs are designed and patterns that tend to recur in them.

## A Quick Reminder
Recall that from the [Interlude](faq.html) that we can set variables to formulas in the Dojo. We will use that in this lesson to make code easy to follow, and to show how Nock programs are made out of discrete chunks.
```
~zod:dojo> =increment-formula [4 0 1]
~zod:dojo> .*(50 increment-formula)
51
~zod:dojo> =increment-formula
```

## The Basic Nock Pattern
## Set up a Core to Do Everything
Every Nock program is just a formula that takes in a subject (argument). In order to run full programs against a subject, we follow the following pattern:

1. first pass: set up the dominos inside an opcode like `2` (or `7`/`8`/`9`). This will create a core, and evaluate it on the second pass.
2. second pass, evaluate that core to get our result.

The pseudocode for opcode `2`, this looks like:
```
*[subject 2 formula-to-set-up-core formula-to-set-up-to-run-against-that-core]
```

Almost always, however, we'll use the `9` opcode (which is sugar on top of`2`) to actually start our programs running, since it supports the concept of "cores" and "arms" very naturally.

For those who know Hoon, this “set up a core and run it” pattern should feel similar to |^ or =>.

## Example Program #1: a Simple Increment Gate
Let's take code to increment the subject and turn it into a Hoon-style gate (a core with one arm and a sample).
The code itself is simple: `[4 0 1]`

### Making the Arm
Our code above expects the subject to be an atom, but Hoon gates expect their subject to be the gate itself, which has the form `[battery payload]`. So we need to adjust our formula to accept this subject format.
```
> =inc [7 [0 3] [4 0 1]]
::  verify that it increments value in the subject's tail (payload)
> .*([1 74] inc)
75
```

### Assembling the Core
We put our formula in the head, and put a default sample of `0` as the payload.
```
> =inc-gate [inc 0]
> inc-gate
[[7 [0 3] 4 0 1] 0]
```

### Invoking the Gate (Running Our Core)
Arms are best invoked using the `9` opcode. Here we use the "standard" version of it: `[9 2 0 1]`. Recall that `9` is `*[a 9 b c]`, where `c` is the "new subject" formula, and `b` is the memory slot of that new subject to take an arm from. Here we use `[0 1]` to keep our subject (the gate) unchanged

#### Using the Default Sample of `0`
```
> .*(inc-gate [9 2 0 1])
1
```

#### Replacing the Sample
To "pass a parameter" to our gate, we use the `10` opcode to edit the subject (the `inc-gate` core) and replace the payload with a value.

We make our subject `[inc-gate val-to-decrement]`.
```
:: notice how opcode 10 can replace the last element of inc-gate
:: the tail of inc-gate was 0; now it's 36
> .*([inc-gate 36] [10 [3 [0 3]] 0 2])
[[7 [0 3] 4 0 1] 36]

::  now use that to set up the subject with a new tail (sample)

::  and call arm in memory slot 2 (our increment formula)
> .*([inc-gate 36] [9 2 10 [3 [0 3]] 0 2])
37
> .*([inc-gate 562] [9 2 10 [3 [0 3]] 0 2])
563
```

#### Summary
Wait, what was the point of this? We took a perfectly good increment formula that worked on atoms, and made it more complicated for the same result. Lame!

In fact, however, this setup will come in really handy when we have multiple arms in our core, not just one. We've made it easy to set up a clean environment for increment to find its sample in, no matter what other data is present. This will come in *very* handy in example program #3.

First, however, let's look at putting a more complicated formula inside a gate. (Hint: the process is the same).

## Example Program #2: a Decrement Gate
Here we take a piece of code that decrements the subject when the subject is an atom, and turn it into a gate.

### Decrement Formula
This decrement formula comes from the [Official Nock Tutorial Example](https://urbit.org/docs/tutorials/nock/example/). It will probably make more sense to read that *after* this lesson--you'll be able to build the decrement function yourself then!

For now, we will take it as a given that the decrement function given below works. (I've altered the code from that tutorial to crash when the input is a cell or `0`)
```
::  the code: set a Dojo face =dec
> =dec [6 [5 [0 1] [1 0]] [0 0] [6 [3 0 1] [0 0] [8 [1 0] 8 [1 6 [5 [0 7] 4 0 6] [0 6] 9 2 [0 2] [4 0 6] 0 7] 9 2 0 1]]]

::  testing in the Dojo
> .*(100 dec)
99

:: with non-atom input we crash
.*([100 101] dec)
ford: %ride failed to execute:
```

### Making the Arm
Let's re-jigger the decrement formula to accept the subject in format `[battery payload]`:
```
::  gets value in mem slot 3 and applies the decrement formula to it
> =dec-arm [7 [0 3] dec]

::  verify that dec-arm decrements the value in the tail
> .*([1 88] dec-arm)
87

::  output the full dec-arm formula code
> dec-arm
[7 [0 3] 6 [5 [0 1] 1 0] [0 0] 6 [3 0 1] [0 0] 8 [1 0] 8 [1 6 [5 [0 7] 4 0 6] [0 6] 9 2 [0 2] [4 0 6] 0 7] 9 2 0 1]
```

### Assembling the Core
```
:: all we need to do is add the sample! (we give it a default value of 0)
> =dec-gate [dec-arm 0]
> dec-gate
[[7 [0 3] 6 [5 [0 1] 1 0] [0 0] 6 [3 0 1] [0 0] 8 [1 0] 8 [1 6 [5 [0 7] 4 0 6] [0 6] 9 2 [0 2] [4 0 6] 0 7] 9 2 0 1] 0]
```

### Invoking the Gate

As in Example #1, we use the `10` opcode to edit the subject (the `dec-gate` core) and replace the payload with a value.

We make our subject `[dec-gate val-to-decrement]`.
```
::  confirm that our 10 code places 36 in the tail
> .*([dec-gate 36] [10 [3 [0 3]] 0 2])
[[7 [0 3] 6 [5 [0 1] 1 0] [0 0] 6 [3 0 1] [0 0] 8 [1 0] 8 [1 6 [5 [0 7] 4 0 6] [0 6] 9 2 [0 2] [4 0 6] 0 7] 9 2 0 1] 36]

::  now use that to set up the subject with a new tail (sample)
::  and call arm in memory slot 2 (our decrement formula)
> .*([dec-gate 36] [9 2 10 [3 [0 3]] 0 2])
35
> .*([dec-gate 562] [9 2 10 [3 [0 3]] 0 2])
561
```

## Example Program #3: Compare 2 Numbers
In this example we will build a Nock core that compares two numbers and returns whether they are greater than, less than, or equal to each other.

### Program Specification
* takes two numbers `a` and `b` as input
* `0` if `a == b`
* `1` if `a > b`
* `2` if `a < b`

The trick is that we only mathematical operators we have to do this with are the `4` opcode for increment, `5` for equals, and a decrement Nock function that I'll supply.

If you want to give yourself a little challenge, try and figure out an algorithm for comparison before going to the next section.

### Comparison Algorithm
* If `a == b`, return `0`
* If `a == 0`, return `2` (i.e. `a < b`)
* If `b == 0`, return `1` (i.e. `a > b`)
* Else recur with both `a` and `b` decremented.

The above works because if a is smaller than b, it hits 0 first. If b is smaller than a, it hits 0 first. If they are equal, we never decrement in the first place, so there are no issues with numbers going negative.

### Implementation Plan
We will construct a core that has two arms: one arm with the algorithm logic, and a second arm with the decrement gate from Example #2. These arms will live in the head of the core (think "battery").

We will put our variables `a` and `b` in the tail of the core (think "payload"). They will be updated before the core is called each time. (This is similar to a trap or door in Hoon).

The final core we will create will have the structure `[battery payload]`, which can be broken down as:
```
[[main-logic dec-gate] a b]
```

Once the core is set up, we'll invoke the algorithm logic arm to set it running!

### Main Loop Logic
We will build the main logic "inside-out", by first defining the "recur" code, and then inserting it into our if/else tests that implement the algorithm.

#### Recur Logic
This assumes the decrement gate lives in the tail of the battery (the battery is the head), i.e. `[0 5]`. We are using `[10 [3 [0 memory-slot]] 0 5]` to extract the decrement gate from our core, and then we invoke it with `[9 2...]`. 

This is identical to what we did in example 2--the only difference is that we now do it twice: once for memory slot `6` (`a`) and again for memory slot `7` (`b`).
```
> =dec-a [9 2 10 [3 [0 6]] 0 5]
> =dec-b [9 2 10 [3 [0 7]] 0 5]

::  we can test our formulas by making a dummy core (a = 33, b = 77)
::  first formula in battery just returns the current subject
> =dummy-core [[[0 1] dec-gate] 33 77]
> .*(dummy-core dec-a)
32
> .*(dummy-core dec-b)
76
```

**describe the recur formula**
use opcode `9`
new subject setup: replace payload with new a and b
run the current core
```
> =recur [9 4 [[0 2] dec-a dec-b]]

:: test it -- we want to see same battery, with payload being replaced with 32 and 76
> .*(dummy-core recur)
[[[0 1] [7 [0 3] 6 [5 [0 1] 1 0] [0 0] 6 [3 0 1] [0 0] 8 [1 0] 8 [1 6 [5 [0 7] 4 0 6] [0 6] 9 2 [0 2] [4 0 6] 0 7] 9 2 0 1] 0] 32 76]

::  clean up the dummy-core
> =dummy-core
```

#### Main Logic

```
::  check whether a is 0; return 2 if true
::  else check whether b is 0; return 1 if true
::  else recur
> =main-logic [6 [5 [0 6] [1 0]] [1 2] [6 [5 [0 7] [1 0]] [1 1] recur]]

::  test (we aren't doing the equality case here)
::  a < b
> .*([[main-logic dec-gate] 9 10] [9 4 0 1])
2
::  a > b
> .*([[main-logic dec-gate] 10 9] [9 4 0 1])
1
```

### Constructing the Core
We build an outer test for whether `a == b`, and then if it's not, we use the `1` opcode to return the core.
```
> =battery [main-logic dec-gate]

::  payload for our core is memory slots 2 and 3 (a and b)
> =comparison [6 [5 [0 2] [0 3]] [1 0] [9 4 [[1 battery] [0 2] 0 3]]]

:: test our 3 cases
> .*([0 8] comparison)
2
> .*([0 0] comparison)
0
> .*([8 0] comparison)
1
```

### Final Code in One Line
We can take this code and use it *anywhere* that our subject is a cell of two numbers, and it will "just work".
```
[6 [5 [0 2] 0 3] [1 0] 9 4 [1 [6 [5 [0 6] 1 0] [1 2] 6 [5 [0 7] 1 0] [1 1] 9 4 [0 2] [9 2 10 [3 0 6] 0 5] 9 2 10 [3 0 7] 0 5] [7 [0 3] 6 [5 [0 1] 1 0] [0 0] 6 [3 0 1] [0 0] 8 [1 0] 8 [1 6 [5 [0 7] 4 0 6] [0 6] 9 2 [0 2] [4 0 6] 0 7] 9 2 0 1] 0] [0 2] 0 3]
```
