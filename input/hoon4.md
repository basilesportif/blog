# 14661 Lines of Hoon: Day 4
At the end of yesterday, I said I'd move to:
* `knee`
* `~+`
* `tall`
* wetness, the essence of beauty

I ended up starting with wetness, just because the behavior of `comp`'s return type was bugging me so much.  Read the official docs in the Hoon tutorial 2.5, and it seems clear that wet gates figure out the product type at the call site, and then make sure that that works throughout the code. Simple enough, but seems like you may as well just cast all outputs to `*`.

## back to `tall`
```
%+  knee  *hoon
|.(~+((wart ;~(pose expression:(norm &) long lute apex:(sail &)))))
```
We got `~+` yesterday: it just caches a computation with subject+formula as the key. `|.` is a thunk that produces a given type. In this case, we see in `knee` that it will be a `rule`.

### `wart` and `here`
`wart:vast`
```
|*  zor/rule
%+  here
  |=  {a/pint b/hoon}
  ?:(bug [%dbug [wer a] b] b)
zor
```
`pint` is line-column range, and We also see that `wart` has some `dbug` stuff in it and seems to "wrap" our computation above, so it's probably just doing some stuff for debugging. In fact, `bug` and `wer` are the door sample of `vast`, so this is probably wrapping ebug info like path and line/col.

#### `here`
Is a parser combinator that says it's "place-based apply". It's a wet gate that takes:
`=+  [hez=|=({a/pint b/*} [a b]) sef=*rule]`
So `hez` is a gate that takes `pint` and data and associates them in a custom way.

The rest is straightforward: we wrap `sef` such that `hez` gets run on the `p` value of `vex` and associates it with:
- start point of the nail
- start point of the continuation parser
- `p` result of `vex`
So it attaches metadata of the line-col range of this individual parse.

## back up to `tall`
`;~(pose expression:(norm &) long lute apex:(sail &))`
`pose` gives us the first parse that works, and we are gonna try 4 here.

### `norm` (13647) inside `vast`
Takes a flag, `tol` and returns a core.  It says "rune regular form", so probably we'll be using the other parsers here when we don't have that.

#### `expression:norm`
Long and boilerplate. Just need to check `stew` and `stet` after digging in. First list here all has elements of form:
```
['_' (rune cab %brcb exqr)]
```

#### `exqr`
```
++  exqr  |.(;~(gunk loan ;~(plug wasp wisp)))
```
Quick `plug` check shows that it parses first then second, and fails if either fails.

##### `wasp` 13970
Tries to find `(most muck ;~(gunk sym loaf))` inside an `ifix` with outside `[;~(plug lus tar muck) muck]`.

##### `muck` 14049
`++  muck  ?:(tol gap ace) ` OK *now* we're getting somewhere and bottoming out. `tol` seems to be a flag for whether we're parsing tall form. If we are, `muck` parses `gap`, otherwise `ace`.

##### `most` 5389 parser
```
|*  {bus/rule fel/rule}
;~(plug fel (star ;~(pfix bus fel)))
```
Keeps a `fel` on the front and gets rid of all other `bus`es between other `fel`s. In the concreate example above in `wasp`, we are dropping all the `muck` between multiple `;~(gunk sym loaf)`, ie parsing terms without whitespace.

##### wasp and `;~(gunk sym loaf)`
`gunk` runs `(glue muck)`

###### glue
```
|*  bus/rule
|*  {vex/edge sab/rule}
(plug vex ;~(pfix bus sab))
```
This is just dropping any `bus` that get created in an `edge`.  I.e. our `bind` function now throws out separators created in `vex` and living in its continuation parser.

Why is this necessary when we already have `most`?  Probably because `most` runs *after* this inner parser. So first we get `sym` and `loaf` as a unit without separators, and *then* we remove the rest of the `muck`. These names are quuite evocative and fun.

##### `sym`
```
%+  cook
  |=(a/tape (rap 3 ^-((list @) a)))
;~(plug low (star ;~(pose nud low hep)))
```
Cooks a parsed tape with `(rap 3 ...)`. `rap` takes a `bloq` (2^n bits, so an `oct` here), and a list. It turns the whole list into one atom using `cat`.  Now go pretty deep into that stuff...

####### `lsh` and `cat`
`lsh`: `|=  {a/bloq b/@u c/@}`
- `a`: bloq size, ie `2^a` total bits. If it's 3, we are doing bytes.
- `b`: how many chunks of `2^a` bits to shift left, ie multiply. E.g. if `a=3`, then we have bloqs of 8, so we multiply by 8 in binary `b` times.
- `c`: original number to shift

`cat`: 
```
|=  {a/bloq b/@ c/@}
(add (lsh a (met a b) c) b)
```
So we figure out how big `b` is in bytes, then shift `c` left that many bytes. Then add `b` at the end.

If you translate the binary for the tape `hi` and run it through `rap`, it works for ASCII (1-byte) values. `rap` for some reason reverses the order though: `i` is first, `h` is second.

##### up to `sym`
OK so `sym` cooks everything with the atomizing `rap` gate.

#### `stet`
Official docs say that it "Add faces [p q] to range-parser pairs in a list." So now we need to find range-parsers, which seem to be type `(list {?(@ {@ @}) rule})`.  But the gate is wet, and most of our "range-parsers" seem to be passing single-character `cord`s.

Not much to spend time on here: it's just adding `p` and `q` faces to the head and tail of each element, probably because `stew` expects that format.

#### `rune`


## Takeaways
* `here` to attach parse location metadata
* general pattern: either transform the `bind` function for `;~` (like `glue` does), or transform two rules into a new one (like `most`) does.
* will be really good for writing up parsing to go back and do so in terms of where things bottom out at various points.
* feel comfortable with `rap`'s process for turning tapes into single atoms

## Next Up
* I skipped over `++  stir`; would be fun to take a whole day breaking it down
* resume `wasp` and `;~(gunk sym loaf)`
* `knee` and `tall`
* `stew`

[Prev: Day 3](hoon3.md) | [Next: Day 5](hoon5.md)
