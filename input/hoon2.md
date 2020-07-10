# 14661 Lines of Hoon: Day 2
On day 1, I got sidetracked down a productive path of understanding parsing as I encountered `ream`. The key takeaways were that parsing had `nail`s, `edge`s, and `rule` (which go from `nail -> edge`).

I also noted the convention that `edge` return values were called `vex`, and that parsing them seemed to always consist of checking whether their `q` face as Just or Nothing.

I said yesterday that I wanted to learn `vest` today to finish up our parsing detour, check `cook` since I had seen that somewhere, and then get back to `tree` early in `hoon.hoon`.

When I left off, we were coming up for air as we pieced together `ream` = `(rash txt vest)`. We knew that `rash` just parses the passed cord using a rule, `vest` in this case.

## continuing with `vest`
`vest` is a rule, so it takes a `nail` as input. The return value is `(like hoon)`, so likely some kind of mold wrapping parsed `hoon` (we saw this type yesterday; it's the AST).

### like, line 4014
In the same section as `edge` and `nail`, and is commented as a "generic edge".  The official docs [say](https://urbit.org/docs/reference/library/3g/#like) that it "generates an ++edge with a parsed result set to a specific type."

It's a wet gate that takes a gate, which here is our Hoon mold. We can already guess that it will be a like an `edge`, except that it will probably unwrap the q/(unit {p/* q/nail})` part of `edge` and cast the `p` as our input mold, which is `hoon` here.

Conceptually that is straightforward, but let's see how it works.
