# 14661 Lines of Hoon: Day 5
When I left off, I had just looked at `rap` to see how it turned bloqs of data into atoms. I said next up was:
* I skipped over `++  stir`; would be fun to take a whole day breaking it down
* resume `wasp` and `;~(gunk sym loaf)`
* `knee` and `tall`
* `stew`

However, I had some good input from `~rovnys-ricfer` re wet gates, so I want to quickly touch on that.

## wetness
Based on his explanation, I looked at [advanced types](https://urbit.org/docs/reference/hoon-expressions/advanced/), in the "wet arms" section.

The first key insight was from this part:
> "Again, will this work? A simple (and not quite right) way to ask this question is to compile all the hoons in the battery for both a payload of p and a payload of q.q, and see if they generate exactly the same Nock. The actual algorithm is a little more interesting, but not much."

Will what work? `this` = "running the original formula with a new type of payload, without worrying about nesting". I hadn't fully realized before that **the challenge of wet cores is to still provide compile-time guarantees**.

So with `turn`, we ask "is the Nock we get from compiling with our current type the same as we get from compiling with the bunt type?"
If you look at `turn`, we see that the same things happen in it as long as `b` is a gate: it knocks out its sample and calls it, etc.
`~rovnys` mentioned that we get a `mull` error if something goes wrong in the call-site Nock compilation. And, in fact, if we do:
```
> (turn ~[1 2] 3)
mull-grow
-find.$.+2
```
i.e. a value for `b` that won't compile, we get a `mull-grow` error because `3` doesn't have a battery. This is different from a dry gate, which would choke immediately with a `nest-fail` because it wouuld just compare the payload and see that `3` isn't a gate.

For the return value type of turn, it does a cast. The cast code expands to the same Nock regardless of `a` or `b`'s value as long as `b` is a gate that works for `a`. Even though `b` is type `gate`, we can pass anything as long as it works with `turn`'s internal Nock.

Why can we pass a dry gate with `b`? Because at the call site, the types of `a` and `b` will be known at compile time, so the typecast will work as long as `a` and `b` are valid, and the compiler will be able to check that `a` nests properly under `b`'s sample (the desired behavior).

### `comp` and `raq`
This is harder, because turn takes a dry gate, while `comp` takes a wet gate (`raq`) as its argument. But a lot of the same stuff applies.

`comp`'s sample is a wet gate, but note that that only matters *if it's used as a gate*. And when it is, we expand its code in place, and so the return value of the bunt wet gate (`raq`) only matters *if it's used in the code*.

In this case, `comp` just calls `raq` on the return values of the parser, but doesn't rely on the structure of `raq`'s return. (`p=(raq p.u.q.vex p.u.q.yit)`). So the expanded Nock will be the same regardless of what `raq` returns.

#### `raq`'s wetness
```
`p=(raq p.u.q.vex p.u.q.yit)`
```
But why does `raq` need to be wet? Let's think what would happen if it were dry. We'd have to know in advance at the call site what types of values would be in `vex` and `yit`. In turn, we know this: we have a given list type that we want to manipulate, so we create or use a pre-existing dry-gate that works on that data type.

But in this case, we don't know that information: `comp` is used by *other* general parser combinators.nn

Example: `pfix`. It does *not* know what kind of data will be in a given `edge`, because it itself is a wet gate. So it needs to pass a wet gate to comp so that the `raq` call can itself be expanded in each place it's used.

`pfix` is one big macro expansion at the end of the day, and the whole expansion's Nock gets compiled and checked to make sure it matches the bunt macro expansion.

### Wetness Conclusion (My Understanding)
Wet gates/cores are just inline functions/macroexpansions, and the compiler expands them fully to test whether they work, rather than using a pre-existing compiled arm+sample as it does for dry gates. As a result, wet gates are slower to compile.

For higher order wet gates that take gates as arguments (like `turn` and `comp`), whether you want to pass a wet or dry gate depends on *how general the call site is*. If the call site knows the types of data it will work with, you can pass a dry gate that matches those. If not, you'll need to use wet.

## Takeaways
* a motivation of wet gates is to provide compile-time guarantees for genericity
* wet gates are like inline function expansions or macro-expansions. Their sample bunt value *does matter*, but only for generatating a "reference" expansion to which other expansions can be compared.

## Possible Writeups
* wet gates and their expansion

## For Next Time
Did only wetness today, so I'll just pick up where I left off; probably will need a quick parsing refresher at that point.
* I skipped over `++  stir`; would be fun to take a whole day breaking it down
* resume `wasp` and `;~(gunk sym loaf)`
* `knee` and `tall`
* `stew`

[Prev: Day 4](hoon4.md) | [Next: Day 6](hoon6.md)
