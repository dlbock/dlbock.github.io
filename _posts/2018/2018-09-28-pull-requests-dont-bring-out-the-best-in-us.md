---
title: Pull Requests And Why They Don't Bring Out The Best In Us
share: false
tags:
  - rant
  - software engineering
---

The pull request (PR) system has existed on GitHub (and other similar products) from day one, and has been revamped a few times to allow for more options of collaboration around (all types of) code. As evidenced by [this GitHub blog post](https://blog.github.com/2010-08-31-pull-requests-2-0/) from 2010.

In the recent years however, I've realized that the pull request model causes more problems and creates more inefficiencies than they solve.


#### **Intent is hard to capture and communicate**

When authoring a pull request, it is important to provide some context as to why you're making the changes that you're making, the thought processes behind the code that was written and what were the tradeoffs. This helps a reviewer understand the what, why and how behind the code change so that they can take that into account when providing feedback. Capturing all that context is hard and time-consuming. And even if you were able to capture all of that in the description of your pull request, it's not a guarantee that your reviewers will have the same background and experience to completely understand the intent behind what you wrote.

The same goes when providing feedback on a pull request. It isn't easy to communicate nuances via the written word and we have to work extra hard to get our point across effectively.

#### **Pull requests discourage emergent design**

Emergent design is the notion that engineers focus on delivering small pieces of working functionality and allow for the design to emerge as the code evolves, as opposed to having long running feature branches and anticipating design in advance.

This way of working allows for more frequent commits to master/trunk, and changes the focus from "I need to deliver this feature" to "How can I deliver business value in small chunks?".

The end result is usually just enough design and code to enable the feature that is being delivered and less opportunity for over-engineering.

#### **Pull requests end up being the conversation conduit as opposed to just conversation starter**

When a team is in the same time-zone and co-located, there is no reason to use pull requests as the tool to facilitate conversation as opposed to just having the conversation in real life. In addition to the difficulty of effectively communicating intent, all sorts of other communication challenges arise when we hide behind the veneer of a pull request: bike shedding, nitpicking, etc.

---
It is a constant reminder for me that the pull request system sometimes does not allow me to work in the most efficient way possible.
I prefer sitting and and having a conversation with my colleague about code in real life, and allow for the back-and-forth and nuances to be communicated that way. In an ideal world, we would be pairing and designing a solution together, committing to master/trunk as frequently as possible while keeping the tests passing and allowing the design to emerge from each iteration.
