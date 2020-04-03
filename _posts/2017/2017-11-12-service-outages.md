---
layout: post
title: Service outages and good practices around handling them
excerpt: "My learnings on what to do when there's a service outage"
share: false
comments: true
tags:
  - outages
  - postmortem
  - microservices
---

As humans, we are inherently flawed, and therefore the software we write is also inherently flawed, no matter how careful we are, or how many tests we write.
It is prudent what we admit that so that we can plan for when things go wrong.

One of the many things I appreciated very much in my time at SoundCloud was their rigorous process around "What to do when there's an outage?" and their dedication to continuously improve it so that the company as a whole was better equipped to handle these unwanted situations when they happened. Here are some of my learnings:

* **Plan for things to go wrong**

  _Hope for the best and prepare for the worst_. This might seem obvious to you (or not), but the most important thing to do to prepare for when things go wrong, is to do just that: prepare. This could take the form of:
    - Runbooks: Document your systems and what to do when they go down. Store the runbooks in a place that is easy for your team to access and not in the same place as the rest of your services.
    - Training: Run regular sessions to make sure your team is well versed on what to do. This will also ensure that your runbooks are up-to-date.

* **Communicate when things go wrong**

  One of the first things that you should do when you find out that something is wrong is notify people. Depending on the size of the problem, you might just be notifying your team, or the whole company and subsequently your public users.

  Use any and all of the following means:
    - If you have a public facing service, notify your users, be it through Twitter, a static status page or banner on your site.
    - Send out severity emails to your entire company, and most importantly your executives, sales and customer support folks.

* **Use a centralized channel to troubleshoot the issue**

  If you have a distributed team (which is more often than not a common occurrence these days), agree upon a means by which you can troubleshoot the issue. In the case of SoundCloud, everyone knew to go one specific Slack channel when an incident occurred. Since there was also a pretty large engineering team, the first thing we did was to establish who would take 'point' and 'comms'. The 'point' person would be the one directing the troubleshooting, even if they weren't the ones actually doing the work. The 'comms' person would be the one taking care of communicating the outage to the rest of the company, and to keep them up to date as things change.

* **Find out how and what went wrong and how to prevent it from happening in the future**

  - Set up a time to talk through what happened as a team. One of things that SoundCloud did which I appreciated was to open up those postmortem meetings to any engineer who was interested in them. Engineers were actually encouraged to attend postmortem meetings once in a while just to be informed on what happened.
  - If you have different teams managing different parts of your system, ask that team to prepare the postmortem report.
  - As part of that report, also include a section for the work that needs to be done in order to prevent this specific occurrence in the future.
  - And of course, schedule that work as soon as you can.


Outages happen. We need to be ready when it does.
