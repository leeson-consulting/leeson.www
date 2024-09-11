---
date: 2021-02-08
categories:
  - Code Bashing
  - Experiments
---

# Genesis: _A place to experiment and grow_

[Genesis](https://github.com/culyun/genesis)
hosts my enthusiastic but often ill-fated code experiments.
It's meant for my personal consumption, but is open for nosing around.

The code is patchy and will seem a bit random to the casual observer.
It's certainly NOT meant for anything other than journalling my thoughts around ideas.

The process in my own mind is throw everything in one place.
Use any method that gets the job done.
Document facts and opinions in commits.
If _anything decent_ begins to emerge from the [SOUP](https://en.wikipedia.org/wiki/Software_of_unknown_pedigree),
create a fresh, focused repo and continue development there.

I was scared about putting this up for quite a long time.
The general quality of published open source is pretty good, and this is somewhat at the other end.
*BUT* the thing is that I find repositories a great way to capture thoughts.
I treat them as journals that you can come back to and review.

I was inspired to give this approach a go after listening to David Epstein who draws on research to advocate for experimentation for advancement.
He pulled out a figure of something like, "... you need to be failing 15-20 percent of the time... "
So I took it to heart and decided to start failing a little to acheive more!

Some seed thoughts (code) currently in the repo are:

- OtherEndian is a UDT meant to transparently define wire endianess for comms / storage protocols.
  This class implements the normal arithmetic operators to allow transparent conversion and interaction with native types.
  Native endianess is determined at build time, with the intention of allowing no-cost abstraction of the dominant little endian encoding without penalty.
- FixedPoint is a UDT meant to bridge the gap between floating point types and q-types (fixed precision integrals).
  The hope was that Q arithmetic could calculated at build time to ensure the underlying integrals used for the calculations were of sufficient width.
  This was to intended to precalculating arithmetic shifts and masks at build time based on the underlying Q M.N type.
  It quickly became bigger than Ben Hur and was TBH a little beyond my ability.
  Nevertheless, the need for such a UDT remains since the current best practice is a highly error prone method of
  capturing the Q M.N structure in parameter names and MANUALLY coding required shifts and masks.
