    Title: A Thought on Monad Transformers
    Date: 2016-12-31T12:15:53
    Tags: DRAFT, haskell, types

I remember, back when I was in grad school, two Haskell fans trying to
write a toy [SMTP][] server for the systems class — the instructor, a
well-known PL researcher, allowed *any* language on the assignments —
and having a long discussion about how to order their stack of monad
transformers so they wouldn't get weird interactions between effects,
like exceptions retroactively undoing log messages.  This sounded kind
of weird and overcomplicated, and if memory serves I wasn't mean to
them about it or anything, but I was kind of rolling my eyes.

[SMTP]: https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol

But for the past month I've been doing [Advent of Code][advoc] in
Haskell as a learning exercise.  I first encountered Haskell probably
back in 1999, but hadn't ever had an excuse to use it for anything
nontrivial.  I tried to reframe problems in ways that would make good
use of lazy evaluation and pure functional style when I could — if I'd
wanted ML, I know where to find it — but some of them really are
sequential, and so I wound up exploring monads and monad transformers.
I actually never found myself juggling noncommuting effects, but I was
trying to get an intuition for how they'd work.

[advoc]: http://adventofcode.com/

<!-- more -->

## Clearly this needs a code block.

So I came up with this simple example:

```Haskell
λ> (tell [1] >> fail "nope") <|> tell [2] :: MaybeT (Writer [Int]) ()
MaybeT (WriterT (Identity (Just (),[1,2])))
λ> (tell [1] >> fail "nope") <|> tell [2] :: WriterT [Int] Maybe ()
WriterT (Just ((),[2]))
```

In one case, when the `fail` backtracks to the choice point `<|>`, the
output-like effect of the first `tell` persists; in the other, it's
rolled back.

If you remove the `newtype` constructors from those values, they look
like this (with accompanying types):

```Haskell
(Just (), [1,2]) :: (Maybe (), [Int])
Just ((), [2])   :: Maybe ((), [Int])
```

## Get to the point already.

And here's the thought: Monads closer to the base monad on the inside
of the transformer stack are closer to the outside of the eventual
type, and therefore have power over monads that are farther from the
base.

More concretely, the superior monad can keep data outside the inferior
monad, so its effects can persist even if the other monad discards a
computation and rolls back, and its effects can reach across worlds if
the inferior monad does forking/nondeterminism.  Going the other way,
the superior monad can discard everything about a computation in the
inferior monad, including any evidence of effects, or it can clone the
computation into parallel worlds that can't interact via the inferior
monad.

(I'm borrowing “superior” and “inferior” from the [ITS][]/[GNU][] usage,
because it's a pretty good fit for this… except that it makes [`lift`][]
move conceptually downward, taking a computation from the superior
monads and extending it into inferior ones.)

[ITS]: https://en.wikipedia.org/wiki/Incompatible_Timesharing_System
[GNU]: https://en.wikipedia.org/wiki/GNU_Project
[`lift`]: https://hackage.haskell.org/package/transformers-0.5.2.0/docs/Control-Monad-Trans-Class.html#v:lift

This extends nicely to the primitive monads, `IO` and `ST`, which are
only base monads (or, as I'm describing them here, in the most
superior position) and aren't available as transformers because that
doesn't make sense — once you've done actual I/O or destructive memory
writes, you can't just decide you actually didn't and start over, or
clone the state of the [`RealWorld`][real], like you could if they
were inferior to a nondeterminism monad.

As for exactly *how* the type system stops you from writing `IOT` or
`STT`… that's a topic for another post.

[real]: https://hackage.haskell.org/package/base-4.9.0.0/docs/Control-Monad-ST.html#t:RealWorld
