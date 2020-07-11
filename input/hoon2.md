# 14661 Lines of Hoon: Day 2
On day 1, I got sidetracked down a productive path of understanding parsing as I encountered `ream`. The key takeaways were that parsing had `nail`s, `edge`s, and `rule` (which go from `nail -> edge`).

I also noted the convention that `edge` return values were called `vex`, and that parsing them seemed to always consist of checking whether their `q` face as Just or Nothing.

I said yesterday that I wanted to learn `vest` today to finish up our parsing detour, check `cook` since I had seen that somewhere, and then get back to `tree` early in `hoon.hoon`.

When I left off, we were coming up for air as we pieced together `ream` = `(rash txt vest)`. We knew that `rash` just parses the passed cord using a rule, `vest` in this case.

## continuing with `vest`
`vest` is a rule, so it takes a `nail` as input. The return value is `(like hoon)`, so likely some kind of mold wrapping parsed `hoon` (we saw this type yesterday; it's the AST).

### `like`, line 4014
In the same section as `edge` and `nail`, and is commented as a "generic edge".  The official docs [say](https://urbit.org/docs/reference/library/3g/#like) that it "generates an ++edge with a parsed result set to a specific type."

It's a wet gate that takes a gate, which here is our Hoon mold. We can already guess that it will be a like an `edge`, except that it will probably unwrap the q/(unit {p/* q/nail})` part of `edge` and cast the `p` as our input mold, which is `hoon` here.

Conceptually that is straightforward, but let's see how it works by commenting its code:
```
::  take a gate, probably to transform p of the edge
++  like  |*  a/$-(* *)

::  our like mold is a gate that takes an edge. Should use |=(b=edge ...) but w/e
|:  b=`*`[(hair) ~]

::  head of cell is the hair--ok we're reconstructing the edge
:-  p=(hair -.b)

::  bind q to the result
^=  q

::  Check whether our edge even has a value
?@  +.b  ~

::  if it does, apply gate a to its parsed p result value
:-  ~
u=[p=(a +>-.b) q=[p=(hair -.b) q=(tape +.b)]]
```

### `plug` and `;~`
I looked at official docs, and somehow found the line `+<:;~(plug cab cab)`
This made me want to understand `plug` and also `;~`. 

#### `;~`: takes gates, produces a gate
It lets you combine previous results into a new structure. The key thing is that each of its elements along the way don't have to understand what type of structure you're building.
`;~`'s `p` gate just needs to have type:
```
;~(p (list q))
p = (b, (a -> b)) -> b
q = (a -> b)
```
Key intuition is that p's job is to use each subsequent output of q and know how to combine it with the buildup of type `b` it already has. `b` can be thought of as a monad: it wraps up some state and we have to use our (a->b) to get back to it.

The key thing is that p knows how to find things of type `a` inside type `b`.
Imagine `b` is an edge, and `a` is a nail. Then `p` knows how to find the nail inside an edge, try to apply a new rule to it, and return a new edge. If there is a further step in the pipeline, it will "unwrap" that new edge, find its nail, and so on.

`;~` is basically monadic `bind`, where `a` below is a nail and `M` is an edge that wraps nails:
```
bind :: M a -> (a -> M b) -> M b
```
Final intuition is that the output of `;~(p ~[x y z])` is a gate.

### back to `vest`
OK so now we see some simple ways to create parser pipelines, and we know `vest` returns an `edge` of `(like hoon)`, which we also now get.

We now have `%.  tub`, which just inverts the sample and gate, so we should pay attention to what type of gate the below forms:
```
%-  full
  (ifix [gay gay] tall:vast)
```

We saw `full` yesterday: it takes a `rule` and makes it a rule that has to parse the full incoming nail, or else fail. So `vest` has to parse its whole input to `hoon` or else fail.  The rule here is `(ifix [gay gay] tall:vast)`. Please Lord let `ifix` me small surface area...

#### `ifix`, line 5335
part of `4e parsing combinators`. Looks like our foray into `;~` is paying off already based on its body:
```
::  fel = [gay gay]
::  hof = tall:vast
|*  {fel/{rule rule} hof/rule}
  ;~(pfix -.fel ;~(sfix hof +.fel))
  
```
So we know this bad boy takes two rules as its first argument and another rule at the end. Seems weird, especially since both rules are the same `gay`. Official docs say that `ifix` is a "parser modifier: surround with pair of ++rules, the output of which is discarded."  Ahhhh ok. So in the examples we see things like `(scan "-40-" (ifix [hep hep] dem))` yielding `q=40`. So it's using the two rules in `fel` to "trim" the input around the middle expression.

When we look at `gay`, that seems confirmed:
```
++  gay  ;~(pose gap (easy ~))
```
`pose` says "first or second", so basically it's matching gap or doing nothing but NOT failing. 

`easy`'s source (line 5280) is...easy. It's a no-op that puts whatever value you pass as the "parsed value". In this case we just pass `~`. It also returns the entire unparsed nail as its continuation, which makes sense.

`++  vast` is labeled as "main parsing core", and `tall:vast` is "full tall form". It's a little weird that we don't seem to have anything here for wide form but maybe that will become clear in a sec.

#### `pose`
Return `vex` if it parsed a value. If not, run the `rule` with no sample (i.e. default sample) and return that edge. For the current `hair` (parsing location), use whichever of `vex` and the default are further along.

We run across something weird, however: `=+  roq=(sab)`. I thought at first this was the bunt value, but that wouldn't allow things to work, because `q.roq` wouldn't contain the continuation parser (`nail` that we still need to parse).

Then I looked back at `;~`, and it's defined as:
```
;~(a b c) expands to:
(a (b arg) c(+6 arg))
```
So we knock out the sample with our incoming argument all the way down the line. That means that `easy` has access to the same initial `nail` as `gap`.

#### `pfix` and `sfix`
`pfix` says that it "discards first rule" and `sfix` "discards last rule". It's pretty clear *what* they do in `ifix` (discard `gap`s), but let's get into their code a bit. `pfix`:
```
  |*  sam={vex/edge sab/rule}
  %.  sam
  (comp |*({a/* b/*} b))
```
This takes us into the `comp` combinator. We'll leave that for later I think.

#### `ifix`/`pfix`/`sfix`
These all parse part of input and discard the rest:
* `ifix` keeps `i`, "inner"
* `pfix` keeps `p`, "post"
* `sfix` keeps `s`, "start"

## `cook` and `sear`
I said I'd hit it today, and now it's almost too easy, conceptually: `cook` just turns a rule into a rule that slams a gate on the `p` in the edge it produces.

`sear` is the same as cook, except that its input gate (`pyq`) outputs a `unit` value. If `(pyq parse-result)` is `~`, then the whole parse fails.

So `cook` needs just the parse to succeed, whereas `sear` needs *both* the parse and subsequent `turn` to succeed.

## Takeaways
* `;~` gets heavily used in parsing
* `;~` knocks out the sample in *all* its gates
* `:~` is easiest to understand inductively
* parsing is pretty mechanical once you "get" `;~`
* a lot of the weird words like `vex` and `sab` appear very very frequently and have an idiom to them.

## Possible Writeups
It's becoming clear that there's a *lot* to be written about `;~` and parsing in general. It's not super-complicated, but if it were well-explained it would speed up grokking of that part by 10x+.

I'd focus a lot on how `;~` and parsing are easiest to understand inductively: lock down the "only one gate passed" base case, and then show how the `n+1` stage builds up when you add a 2nd gate.

## Up Next
* walk through parsing `comp`
* keep diving into tall-form parsing
* finish `vest`?
* Go back to `tree` if I have no better ideas

[Prev: Day 1](hoon1.md) | [Next: Day 3](hoon3.md)
