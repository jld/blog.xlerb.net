    Title: 2015 ICFP Contest, Contemporaneous Rambling Edition
    Date: 2015-08-10T17:09:35
    Tags: icfpc

Someday I'll write about something other than the ICFP programming
contest here.  Someday sooner I might even write a more coherent
writeup about this year's.  But until then, here's the text that
poured from my fingers into my `README` file at 3:30 AM Monday
morning.

<!-- more -->

# Rambling

This is not the algorithmic breakthrough you're looking for.  I wasn't
feeling hugely inspired about writing game AI, so I didn't, and I
wanted an excuse to spend more time re-learning Rust, so I did.

This is me trying to reuse my idea from 2012 (when I used the
just-released Rust 0.3) (also sort of my idea from 2008 and,
eventually, 2009) and just try a bunch of random paths, looking for
ones that score well.

That worked okay in past years when there was a specific goal to aim
towards, or especially a *series* of goals so I could do some kind of
evolutionary-ish hill-climbing thing to the ones that made it.  This
problem has a bunch of fiddly positional stuff that I wasn't feeling
up to trying to come up with a utility heuristic thing for, and it
didn't really go so well.

(I also meant to write a routine to actually display the game state,
so I could visualize what was going on, and... kind of never did.
Oh well.)

But I did manage to adapt it to find paths with chosen strings, after
spending a bunch of time thinking about the right way to use
Brzozowski derivatives to build a minimal DFA and considering the
nondeterminism of choosing a letter for a command and then ignoring
all of that and just hacking up something that more or less worked.
And it did kind of okay at that.

Also, typing this, I realize that just top-down imposing a specific
power phrase to try would have been a lot simpler, but I'm not going
to try a frenzied last-minute implementation, because it won't help
much.

I'm sure everything I *should* have done will make perfect sense when
I get up tomorrow, relaxed and having taken the mental step back from
everything that I can never manage during the weekend, because that's
always how it works.
