
# Interlude: Loose Ends and FAQ

By `~timluc-miptev`

Twitter: [@basile_sportif](https://twitter.com/basile_sportif)

## Series Outline
* [Part 1: Basic Functions and Opcodes](part1.html)
* [Part 2: The Rest of Nock and Some Real-World Code](part2.html)
* Interlude: Loose Ends and FAQ
* [Part 3: Design Patterns and Real Programs](part3.html)

## Convenience Trick: Variables in the Dojo
You can assign Nock code to Dojo variable faces as seen below. Here, we store the Nock formulas for increment and decrement, and run them against the subject `50`.
```
~zod:dojo> =inc-formula [4 0 1]
~zod:dojo> .*(50 [4 0 1])
51
~zod:dojo> =dec-formula [8 [1 0] 8 [1 6 [5 [0 7] 4 0 6] [0 6] 9 2 [0 2] [4 0 6] 0 7] 9 2 0 1]
~zod:dojo> .*(50 dec-formula)
49

:: remove the faces when done
~zod:dojo> =inc-formula
~zod:dojo> =dec-formula
```

## Order of Operations/Evaluation
One question I've seen come up after Parts 1 and 2 is "in what order does Nock evaluate expressions?"

### Explanation and Walkthrough
The answer is straightforward: it starts with code that has only one pseudocode operator and all Nock code, as below: 
```
*[50 4 4 [5 [0 1] [1 50]]]
```

The interpreter can then be thought of as moving left-to-right, expanding according to its rules, until it can't expand any further into pseudocode:
```
*[50 4 4 [5 [0 1] [1 50]]]
+*[50 4 [5 [0 1] [1 50]]]
++*[50 5 [0 1] [1 50]]
++=[*[50 [0 1]] *[50 [1 50]]]
++=[/[1 50] 50]
```
Once we get to that last line, `++=[/[1 50] *[50 [1 50]]]`, no further expansion into pseudocode is possible.

At this point, the interpreter expands "inside-out", starting from the furthest inside pseudocode operators. In this case, those are `/[1 50]` and `50`, so it does those:
```
++=[50 50]
```

Then it moves to the next-furthest-inside operator: `=`, and then to the `+` operators:
```
++=[50 50]
++(0)
+(1)
2
```

### Summary of Order of Operations
1. First pass, when we have `*(nock-code): expand left-to-right into pseudocode
2. Second pass, when we've fully expanded pseudocode: evaluate operators from the inside-out

## Nock `11` -- Hints/Messages and Side Effects for the Interpreter
I put this off until now since it's not STRICTLY needed in expanding Nock code.

`11` lets you compute side effects and pass information to Nock's interpreter, for that interpreter to do what it wants. The most common uses are things like 
jets and debugging prints. In theory, this opcode is fairly dangerous, since it tells the interpreter that it's free to do anything.

### Pseudocode
There are two different versions of `11`, depending on whether the head following `11` is an atom or a cell
```
::  head is an atom (b)
*[a 11 b c]  *[a c]

::  head is a cell ([b c])
*[a 11 [b c] d]     *[[*[a c] *[a d]] 0 3]
```

This pseudocode lets `11` handle two separate use cases:
1. We just want the interpreter to process a message in its own language/environment
2. We want the interpreter to compute some Nock code with Nock semantics before processing the hint.

### Option 1: Interpreter Processes a Message
Imagine the following Nock:
```
.*([50 51] [11 369 0 2])
```
This passes the value `369` to the interpreter, where maybe it would be the key in a hashmap. The value associated with the could be the function to debug print the whole operation.  Or the value could be a jet function that specifically knows how to compute the `0` opcode faster when it's followed by a `2`.

After the interpreter does whatever, it needs to then discard the value `292.989`, and compute
```
~zod:dojo> .*([50 51] [0 2])
50
```

### Option 2: Run Nock Code and then Process a Message
In this case, instead of passing a static value to the interpreter, we pass it a Nock computation against the subject. The result of that computation will then be available to the interpreter to use as a message.

### Practical Use of `11`: Jet Registration
Jet registration in Hoon usually looks like
```
++  my-function
    ~/  %my-function
    |=  var=@
    ...
```
`~/` uses Nock `11` to pass the value `%my-function` to the interpreter, where it is associated with the C code that produces the product of `my-function`.