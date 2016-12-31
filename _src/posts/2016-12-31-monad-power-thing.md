    Title: A Thought on Monad Transformers
    Date: 2016-12-31T12:15:53
    Tags: DRAFT, haskell, types

I remember, back when I was in grad school, two Haskell fans trying to
write a toy SMTP server for the systems class — the instructor, a
well-known PL researcher, allowed *any* language on the assignments —
and having a long discussion about how to order their stack of monad
transformers so they wouldn't get weird interactions between effects,
like exceptions retroactively rolling back logging.  This sounded kind
of weird and overcomplicated, and if memory serves I wasn't mean to
them about it or anything, but I was kind of rolling my eyes.

But for the past month I've been doing [Advent of Code][advoc] in
Haskell as a learning exercise -- I first encountered Haskell probably
back in 1999, but had never had an excuse to use it for anything
nontrivial.  I tried to reframe problems in pure functional terms when
I could, but some of them really are sequential, and so I wound up
exploring monads and monad transformers.  I actually never had
noncommuting effects, but I was trying to get an intuition for how
they'd work.

[advoc]: http://adventofcode.com/

<!-- more -->

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

The extreme case of this are the primitive monads, `IO` and `ST`,
representing actual real effects.  They are always in the most
superior position, at the base of the transformer stack; there's
actual I/O and/or actual memory overwriting happening, and that can't
(for example) be rolled back by the likes of a `Maybe`.

<!-- FIXME: break out the rest of this into its own post, and redo
that last paragraph as a teaser or something. -->

Certainly you can have `Maybe (IO a)`, but (and, yes, I just spent
some time scribbling to convince myself of this) it can't be a monad,
because there is no well-typed term (ignoring `undefined` (etc.) and
unsafe code, which doesn't actually do what you want) that can be
`(>>=)`.  Given a `Maybe (IO a)` and an `a -> Maybe (IO b)`, the only
way to get the `a` you need in order to apply that function and get
the result is to perform the effects in the `IO a`, and there's no way
to use that `a` to get a value that isn't `IO t` for some `t`.

It can be an `Applicative`, though:

```Haskell
newtype IOT f a = IOT { unIOT :: f (IO a) }
instance Functor f => Functor (IOT f) where
   fmap f (IOT x) = IOT (fmap (fmap f) x)
instance Applicative f => Applicative (IOT f) where
   pure = IOT . pure . pure
   (IOT f) <*> (IOT x) = IOT (pure (<*>) <*> f <*> x)
```
