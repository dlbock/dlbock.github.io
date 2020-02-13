---
title: Beyond Just Feature Development
share: true
tags:
  - software engineering
---

Software engineering isn't just about pushing out features, IMHO. There is so much that needs to happen between when an engineer is "done" with a feature and when it is actually available to be used by customers. I find that some engineers tend to glaze over these things with the excuse that it is not their job to ensure the software they write gets into their customers hands and works as intended. I disagree. Here are some of the questions I ~~worry about~~ ask myself when writing software.

### Testing
  - How do I create automatic test harnesses for these features?
  - How do I ensure that this will work in production? Is there a pre-production environment that's a carbon copy of production? Is there a way that I can simulate the load/traffic of production?

### Deploying changes
  - How do I get these changes to pre-production? And then to production?
  - How quickly can I get bug fixes into production, if needed?
  - Is there an automatic deployment pipeline? Is it a manual process? Is it a black box controlled by someone else?

### Releasing features
  - How do I enable these features for our customers? Or a subset of our customers? Is it something I can do myself, or do I have to go searching for the right person for it?
  - Do we even have a feature toggle system in place? Is it somewhere that I can find easily or do I have go on a digging expedition
  - How are these feature toggles managed across environments?

### Observability of features
  - How do I know how these features are performing in production?
  - How do I know if they are even being used in production?
  - How do I get notified when things go wrong in production?

I'll update this list over time as I learn about other things that I should ~~worry~~ care about.
