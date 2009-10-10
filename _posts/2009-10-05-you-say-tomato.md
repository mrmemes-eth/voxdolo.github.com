---
title: You Say Tomato
layout: post
excerpt: |-
  Introducing <a href="http://tomatoi.st">tomatoi.st</a>, a
  <a href="http://www.pomodorotechnique.com/">Pomodoro</a> timer.

---

## I say Pomodoro!

After a significant amount of time and effort and the gracious help of several
contributors, I'm proud to officially announce [tomatoi.st][t].

### For those in a hurry

[tomatoi.st][t] is a simple timer application that helps you implement the
[Pomodoro Technique][p]. You should totally use it.

### The Story

While pairing with [Corey Grusden][cg] a few months ago, we decided to experiment
with the [Pomodoro Technique][p]. We immediately found that it helped improve
our productivity and fight off project fatigue.  Plus it gave Corey some
officially sanctioned time in which to deal with his digital logorrhea (SMS
much?). Needless to say, we were hooked!

The immediate hurdle was that the software available for doing the timing was
lackluster (fwiw, [decent software][r] has since been developed).  In light of
this, we spent the first few days working with a simple `sleep 1500 &&
growlnotify` type solutioni which was inelegant at best. More importantly, it
was missing an utterly essential piece of functionality: the ability to start a
timer at my desk, go to the john, and check up on the time left in a break
whilst on the throne!

And so, in pursuit of this lofty goal, [a tomato][t] was born.

### What it does

[tomatoi.st][t] does very little, but we like to think it does it very well.
*ahem*.

1. Visit [http://tomatoi.st][t]
1. Click the "Pomodoro" button
1. Start a focused unit of work
1. When you hear the "ding!", click "Short Break"
1. Take a break, go to the john, whatever
1. Rinse and repeat

It is suggested that you do 4 sets of Pomodoros. The first three are followed by
a short break, the last by a long one. [tomatoi.st][t] makes the assumption that
you'll be doing them in that order and handily suggests the next timer you
should do by highlighting it's button in green when the previous timer expires.
Your current timer is highlighted in yellow while it's underway and then red
once it's expired. Your timer history is tracked just below the big ticker.

Timers out of sync with reality (say, after a long lunch)? Click the "Reset"
button to start fresh.

Want to come back to this set of timers tomorrow? Customize the name of the
timer and then "rename" it so it's easy to remember.

### What it doesn't do

It is not intended to be a full implementation of the [Pomodoro Technique][p].
Notably absent are things like task planning and daily review.

### What's next

Coming up promptly is integration with [Fluid][f]'s javascript API to provide
neat things like Growl integration.

Want to help shape the face of what [tomatoi.st][t] will become? There are a
number of ways you can help do so:

* scratch your own itch and [fork it on github][d]
* file a [github issue][i]
* peruse and/or add to our [tracker project][tr]
* or make a request below in the comments

### Shout Outs

A number of people have contributed to making this application what it is and
I'd like take a second to thank them all.

* [Sandro Turriate][st]: for adding server side polling to keep multiple people
  watching the same timer in sync as new timers are started.
* [Wes Gibbs][wg] and [Les Hill][lh]: for adding timer countdown in the title
  bar (one of my favorite features).
* [Corey Haines][ch]: for his general good will toward the app and for helping
  to spread the word about it.
* [Andrew Maier][am]: for taking the app from plain to fantastic! His design is
  what made the official release a possiblity.

[t]: http://tomatoi.st/
[p]: http://www.pomodorotechnique.com/
[d]: http://github.com/voxdolo/ding
[r]: http://github.com/rubyist/tomato
[i]: http://github.com/voxdolo/ding/issues
[f]: http://fluidapp.com/
[h]: http://www.hashrocket.com/
[tr]: http://www.pivotaltracker.com/projects/11740
[cg]: http://www.coreygrusden.com/
[st]: http://turriate.com/
[wg]: http://twitter.com/wesgibbs
[lh]: http://blog.leshill.org/
[ch]: http://www.coreyhaines.com/
[am]: http://www.uxbooth.com/author/andrewmaier/
<!-- #hashrocket -->
