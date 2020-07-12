# 14661 Lines of Hoon: Day 3
When we left off yesterday, we said we'd pick up with:
* walk through parsing `comp`
* keep diving into tall-form parsing
* finish `vest`?
* Go back to `tree` if I have no better ideas

## `comp`
For reference, `pfix` had:
```
  |*  sam={vex/edge sab/rule}
  %.  sam
  (comp |*({a/* b/*} b))
```
`comp` is a "wet core" (`|@`), which I don't see a ton, so time to look it up. It says "A |@ expression produces a 'wet' core whose payload is the expression's subject."  This isn't super helpful, since I don't know which expression they're referring to.

So to make headway, we go read about `comp` in the official docs. Interestingly, the source given in the official docs is different: it uses a wet gate instead of a wet core with one arm, `$`. It also puts `raq` at the head. The docs say that it produces a `rule`.

Ah I see now: we use `|@` instead of `|*` so that `raq` doesn't have to take a bunt value. The `|@` operator packages up arms and puts them in the battery, and then places the current subject as the payload. So this whole thing is just a cute way to put a non-bunt value in the sample.

Oh, ok. So `comp` takes a gate with params a/b and produces a new gate that takes [edge rule]. That is then run on `sam` to throw out the first parsing result.

### Important: How It Works
This makes sense: **we injected `raq` using `comp`**. So now, every time we have an edge and a rule, we will just throw out the parsed result of the edge, but our continuation parser can move on.

So `comp` just inserts some inner magic into the parsing monad: instead of manually defining in (eg) `pfix` how it will combine the new parsed result with the existing `p`, you pass a function that does that for you.

### How to Combine `edge` and `rule`
You can either manually write logic to tell your combinator how to parse and combine the edge's `p` with the `p` from running the `rule`, **or** you can use `comp` and just pass the function that you want to use.

## on to `vast`, line 12133
`(ifix [gay gay] tall:vast)`
So now we understand *everything* in `vest` except for `tall:vast`. Time to dive in; I'm expecting to be here awhile. My primary initial question is how we will parse for wide form when it seems that we're just doing for tall, but I expect we'll find out.  I know for a fact that `ream` and `vest` must be parsing wide form too, because:
```
> (ream '|=(a=@ a)')
[%brts p=[%bsts p=term=%a q=[%base p=[%atom p=~.]]] q=[%wing p=~[%a]]]
```

`vast` looks like you should be able to call it as a door, so it's weird that doesn't work when I try it at the Dojo. Anyway. Oh wait, it's a door, not a gate, so: `~(. vast [%.n /])` works. Sweet. Of course `[bug=? wer=*path]` isn't documented, because Urbit.

### `tall`, line 14306
```
%+  knee  *hoon
|.(~+((wart ;~(pose expression:(norm &) long lute apex:(sail &)))))
```
OK so we're gonna do something called `knee` to some blank Hoon, and the final arg is a trap (i.e. a thunk). Kinda weird, but let's check `knee`.

Wait, before that...what about `~+`? Turns out that that "Caches the formula and subject of p in a local cache (generally transient in the current event)." This sounds like memoization, right? 

### `knee`
Ohhh ok, so `knee` is 
```
 =|  {gar/* sef/_|.(*rule)}
  |@  ++  $
```
which is just a wet gate.

## Takewaways
* `comp` extracts the general pattern of using a function to combine `p` in a parser combinator
* I think it's OK to think of `edge`s as monads and `;~`+combinators as a `bind` function for them?
* got fully comfortable with `=+` and `=|` as ways of both pinning variables and making payloads

## To Explore
* `knee`
* `~+`
* `tall`
* wetness, the essence of beauty

[Prev: Day 2](hoon2.md) | [Next: Day 4]()
