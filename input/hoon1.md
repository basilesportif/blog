# 14661 Lines of Hoon: Day 1
I'm going to proceed through all of `hoon.hoon` and document the entire journey. I won't let any part remain un-understood. Using the [June 29, 2020 commit](https://github.com/urbit/urbit/blob/7266b3f5c3ddde60b15427a8aa35e0fe97bfae18/pkg/arvo/sys/hoon.hoon) to keep line numbering standard.

Goals:
* develop an intuitive understanding of how pieces fit together through repetition
* burn commonly-used features into memory through usage
* discover a pedagogical approach so that no one has to do this again

I don't want to "cheat" much by referring to official docuumentation, but I probably will on occasion. It's a preference, not a rule.

## Lines 1-181: Basic Arithmetic and Core Registration
Nothing really to see here. I did have to look up [~%](https://urbit.org/docs/reference/hoon-expressions/rune/sig/#sigcen) which turns out to be core jet registration.

The first `~%` seems to just be registering the version number. The second one, `~%  %one  +  ~` confuses me a bit. I don't get what it means to "register the parent at leg `+`".


The rest is basic arithmetic and comparison functions.

## `+|`
Line 187 brings some tree addressing functions, and I look up `+|`. I modify the example given in the docs, and find that:
```
> (ream '|%  +|  %numbers  ++  two  2  +|  %letters  ++  a  "a"  --')
[p=~ q={[p=%numbers q=[p=~ q={[p=%two q=[%sand p=%ud q=2]]}]] [p=%letters q=[p=~ q={[p=%a q=[%knit p=~[97]]]}]]}]
```
...which means we can use `+|` to label sections of a core for the parser. It also shows that the parser turns `|%` into maps internally that can be named using `+|`.

### detour: `ream`
Since the example used `ream`, I detour to it in line 14360. It parses `cord` to `hoon` using `(rash txt vest)`

#### `+$  hoon` (line 6670)
- default value of `[%zpzp ~]`
- `$^` to interpret head and tail of cell both as Hoon
This is followed by a long list of tagged unions for different hoon "leaves" and runes (as 4-letter terms).

#### `rash`
Takes an `@` and `sab=rule`. The `rule` type comes up in lots of code, so lets dig into it.

##### `rule`
` _|:($:nail $:edge) ` -- interesting, I had to look up `|:`, seems it lets you define a sample that's not the default value. Then we cast by example, so the rule is just a gate that takes a `nail` and returns and `edge`.

##### `nail` and `edge`
```
++  edge  {p/hair q/(unit {p/* q/nail})}  ::  parsing output
++  nail  {p/hair q/tape}   ::  parsing input
++  hair  {p/@ud q/@ud}     ::  parsing trace (line and column number)
```
So a `nail` is just `[[@ud @ud] tape]` -- a parsing trace and some tape.

An `edge` is `[[@ud @ud] (unit [* nail])]` -- a parsing trace and some noun plus input (`nail`).

##### up to rash again
Backing all the way up to `rule` -- a `rule` is a mold for a parsing gate (input -> output). Why are they called `nail`s and `edge`s? Who knows, but maybe it will stick mentally.

And that takes us back to `rash`, which takes `naf=@` and `sab=rule` and runs `(scan (trip naf) sab)`. `trip` -- seems to just be `cord -> tape`. There's some interesting stuff in there, but let's just assume it for the moment.

So with `(trip naf)` we turn a cord into a tape. That takes us down to `scan` right below. We can already guess now that scan is going to work through the tape until it hits some `rule`.

##### scan
Shonough, `scan` takes `{los/tape sab/rule}`. It runs `(full
sab)`, which takes a `rule` and returns a gate expecting a `nail` (parsing input). 

###### full
`full` is interesting and helps us understand `edge`. The gate it returns runs the closured rule on the passed nail (i.e. parses it, returning an `edge`)., but ONLY returns a value for the edge if there is no more input. This is line 5296:
```
?~(q.vex vex ?:(=(~ q.q.u.q.vex) vex [p=p.vex q=~]))
```
The key is where it tests `q.q.u.q.vex`, which is the `nail` inside the `edge`. It returns a `q=~` if the `nail` isn't null, because that means that there is still is probably unparsed data left over, which makes sense: `full` does the full thing or fails.

##### back to scan, then rash
We see that `scan` produces `vex` by running the `(full sab)` gate on `[[1 1] los]`, i.e. line 1 column 1 of the input tape. So now we know that `nail` probably uses `hair` to represent its starting point in the text.

Next, `scan` crashes with a stack trace if `q.vex` is ~, printing `p.vex`. So `p.vex` is the location of the parser error most likely.  Otherwise returns `p.u.q.vex` -- the parsed noun.

So `rash` is a thin wrapper: make the input atom into a tape and `scan` it fully with the input rule, crashing on failure.

## Takeaways From Day 1b
Starting to build intutions for what a `rule` is, since it comes up so often in parsing, even though we haven't looked at one yet. Edge and nail make total sense now. We saw the whole Hoon-AST-defining data structure as well and it was very straightforward. Also realized that it parses into those tagged cells for each rune's part of the AST.

### Parsing
- nail is input starting at [line col]
- edge is output of an arbitrary noun, with an error stacktrace given if necessary, and an "undone" input (continuation yes I cheated with docs) also passed back if it exists.
- we haven't gone through the internals of a `rule` yet

### Codebase Covered
I wrote this for today, but I think I'll stop it tomorrow.
* 1-187
* 4012-4021
* 5545-5562
* 6670-6800
* 14361-14364

## Day 2 Plan
* vest to understand ream
* tree to continue in order (line 187 ish)
* cook just because it seems relevant

[Next: Day 2](hoon2.md)
