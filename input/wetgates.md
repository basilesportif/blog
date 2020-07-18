# Wet Gates, Explained at Last

Wet gates in Hoon are not mystical or particularly complicated to understand. However, they are so fundamentally different from dry gates that the shared word "gate" probably just confuses.

Wet gates are simple macros that are expanded in place and checked for type safety at compilation time. They have strict, predictable rules that can be observed by playing around with them, and are **not** an "anything goes" free-for-all, although they initially may seem like that.

## Dry Gate Compilation and Calling
We'll be contrasting with dry gates throughout, so let's just review how those work.

### Dry Gate Compilation
When a dry gate declaration is compiled, the following happens:
1. The body of the dry gate is compiled into Nock (call this "formula")
2. The bunt of the sample is compiled into Nock (call this "payload")
3. A core as created with the form `[formula payload current-subject]`. The formula is given a face named `$`

As an aside, you can also create dry gates by using the structure
```
=+  a=9
|%
++  $
  some-code here
```
Here we do things manually. Step 1 above still happens with our formula in `$`. We skip step 2. Then we bundle up the core as `[formula current-subject]`, but it just so happens that the head of `current-subject` is `a=9`, so we actually have the structure `[formula a=9 rest-of-subject]`. This is simply done to make a payload with a non-bunt for the sample. Also notice that there's nothing magical about `$`--it's just the name of an arm.

So the dry gate rune `|=` is just a code expansion: it "rewrites" code in place:
```
|=  a=@ud
  (add a 2)
::  expands in place to ...
::  =| pins a value to the head of the subject like =+, 
::  but its product is the bunt value of the type passed
=|  a=@ud
|%
++  $
  (add a 2)
```

### Dry Gate Calling
To use our dry gate in other parts of code, the compiler simply gets its core, replaces its sample with a user-provided sample if one is provided, and runs its formula.  For those who are interested, [Part 3 of My Nock Primer](part3.html) gives simple examples how the Nock 9 opcode is used to do this.

The key here is that *dry gates are only compiled once*, and then are called potentially many times in programs by replacing the sample and running the formula.

## What Are Macros?
Besides dry gates, macros are the other prerequisite for understanding wet gates.

Macro is just a general word for code expansion, as opposed to calling a function/subroutine. In our dry gate example above:
* the compilation of a dry gate uses a macro, `|=`, to expand code and save typing
* the actual running of a dry gate *after* compilation uses the gate as a subroutine: it calls the precompiled code while changing something in that code's environment (the sample).

Of course, "saving typing" isn't the only benefit of macros: we have bundled `|=` up into an abstraction that we can cleanly use without having to think about its innards every time, and in a way that makes it easier for us and other programmers to grok when we read it in code.

Some programming languages use macros in extremely complex ways. LISP is the obvious example here; its double-pass evaluation lets you build up fully custom programming languages within it. Much like medieval theology, macros can absorb arbitrary amounts of mental energy: [Paul Graham](https://twitter.com/paulg/status/1260138502974570497), in particular, has made a career of mentally masturbating to macros while ignoring nearly every other facet of computer science.

Hoon's macros, however, are fairly limited. Runes are one form, and wet gates provide another for handling type issues. We'll see exactly how this works when we get to Wet Gate Calling.

## Wet Gate Compilation
The initial compilation of a wet gate is identical to that of a dry gate. The `|*` rune even expands in the same way as `|=` does:
```
|*  a=@ud
  (add a 2)
::  expands to
=|  a=@ud
|@
  ++  $
  (add a 2)
```
This looks the same process as the compilation of a dry core...and in fact, it *is* the same process. The difference is that the gate gets marked as "wet", and the compiler treats it differently when called. Let's look at how that works now.

## Wet Gate Calling
Imagine we have a gate called `listify` that takes an argument, and makes it the head of a one-element list. We'll make both a dry and wet version of it, and examine how the compiler treats it when it's called.

### dry `listify`
```
> =dry-listify |=(a=* [a ~])
> (dry-listify 3)
~[3]
> (dry-listify 'timluc')
~[109.355.981.433.204]
> `(list cord)`(dry-listify 'timluc')
```
When `(dry-listify 3)` is called, the compiler just takes the `dry-listify` core, replaces its sample with `3`, and runs the already-compiled formula. Easy.

The limitation, however, is that because the formula was already compiled, it can only deal with `'timluc'` as a noun. If we try to cast the return as `(list cord)` or even `(list @)`, we'll get a `nest-fail`.

### wet `listify`
```
> =wet-listify |*(a=* [a ~])
> (wet-listify 3)
~[3]
> (wet-listify 'timluc')
['timluc' ~]
> `(list @tas)`(wet-listify 'timluc')
~[%timluc]
```
Ok, so it seems that our wet gate retained all the type information of the sample, even though it was declared as `a=*`.  How did it do that?

### Wet Gate Calling Steps
Whenever a wet gate is called anywhere in a program, the compiler does *not* call the previously compiled Nock for the wet gate.  Instead, it does the following:
1. Replace the sample of the gate with the caller's sample
2. *re-compile* the formula into Nock using that new sample
3. Check whether the original compiled Nock and this new compiled Nock are the same
4. If the same, insert the newly compiled formula+sample into the code at this call site: this is a macro expansion

The key difference between dry gate calls and wet gate calls is that dry gate calls use the Nock `9` opcode to call a previously created core with a new sample, while wet gate

### Nock Compilation Example
This might be easier for some to understand with a concrete example. What does it look like when newly compiled Nock is the same as the original wet gate Nock? Thanks to the magic of Urbit's "Nock all the way down" data structures, we can quickly check!

#### Example 1
```

```


## Wet Gate Sample Constraints
Does the declared sample even matter for a wet gate? That depends on how the sample is used inside the gate.

### Example 1: Sample Is Used Specifically
```
> =foo |*(a=$-(* *) (a 9))
> (foo |=(x=@ud +(x)))
10
> (foo 3)
-find.$.+2
```
Here we use the sample internally as a gate. So it works when we pass an actual gate, but chokes when we give it the value `3`, and tells us that `3` doesn't have a `$` face in its `+2` memory slot (in fact, it has no memory slot `2` at all).

### Example 2: Sample Could Be Anything
Now let's modify `foo` from the prior example to just return its sample:
```
> =bar |*(a=$-(* *) a)
> (bar 3)
3
> (bar |=(a=@ud a))
::  returns whole gate
```
In this case, when the initial Nock is compiled using the bunt value for the sample, the generated Nock is simple: itjust returns the sample's value. In fact, we can investigate this in the Dojo:
```
> -.bar
[0 6]
```
This is just Nock for "return the value located at memory slot `6`", which for a gate `[formula payload context]` is simply the value of `payload`.  Contrast with `-.foo`:
```
> -.foo
[8 [0 6] 9 2 10 [6 7 [0 3] 1 9] 0 2]
```
Here, the formula is doing a lot more. (Exactly what it's doing will be clear to those who [follow the path of Nock](part1.html)).

### So, Does the Wet Gate's Sample Type Matter?
As we've seen, the wet gate's sample type matters inasmuch as the gate's formula does operations that depend on that type. As a result, when you see wet gates declared with types, the type usually indicates to you how generic the operations performed inside are, although we don't know for sure until we compile the Nock at a given call site.

## Wet Gates as Higher-Order Functions

## Passing Wet Gates as Arguments
