---
title: "Building Twenty Apps to Find the One That Works"
date: 2025-07-08T18:28:36-04:00
draft: false
toc: false
images:
tags: 
  - entrepreneurship
---

### Introduction

What if you knew that only 5% of your startup ideas would succeed—but you had the grit to try 20 times? Pieter Levels famously said that only 1 in 20 of his ideas succeeds [link](https://x.com/levelsio/status/1457315274466594817?lang=en). This blog series documents my journey to create twenty web applications, one after the other, with the goal of hitting profitability by app number twenty—or earlier. [Using Coding Skills to Make Passive Income](https://www.youtube.com/watch?v=2VD9CWJ9HPM) is a fantastic intro around tactics to generating a successful app.

In this new era with AI tools, I believe I can build 20 apps with [Cursor](https://cursor.com/) and its agent interface doing most of the grunt work. In fact, I will hopefully be giving a talk at the [ExMex](https://exmexconf.com/) Elixir conference about how I leverage Cursor to refactor repetitive tasks at my day job.

This is not a passive investment strategy. It’s a deliberate, compounding effort to ship small, useful tools into the world.

Here are some other useful resources I have collected and will keep in mind while building.

My notes from [10 Things I’ve Learned | Jason Fried at BIG Omaha](https://www.youtube.com/watch?v=wC14JM94BIs):

1. Bootstrapped company
  - we need to make money--something someone can get good at with practice
  - funded company: we need to spend money--rent time
  - own your own schedule

2. Price
  - you need to be good in order to charge money for something
    - an intimate relationship--will give honest feedback
  - are we $50 good enough?

3. Useful > Innovative
  - innovation is overrated
  - usefullness never wears off
  - ask yourself, is this useful?

4. Focus on What Won't Change (in your business)
  - speed, reliability, secure, simplicity
  - quality is an iteritive process

5. DIY
  - how can you evaluate something if you don't know what it does?
    - try to solve the problem internally
    - forces you to learn stuff

6. I'm Sorry 
  - HR/lawyer apologize
  - a real apology: more trusting customer base
  - do your customers believe/trust your apologies?

7. Draw a Line in the Sand
  - have a clear point of view
  - what would you say no to?

8. Specs, Features and Tech
  - can I afford it?
  - is it comfortable?
    - customers don't care about the features
  - nail the basics
 
9. Less
  - finish things
  - say no
  - focus on a few things really well

Description of the Mom Test from the [blog post](https://www.coryzue.com/writing/solopreneur/) associated with "Using Coding Skills to Make Passive Income" video:

> The key point from this book is when you’re trying to pitch a startup idea to anybody, they’re going to talk to you like your Mom would talk to you. You’re going to pitch your thing and they’ll say “sweetie that’s such a good idea, I love it, you’re going to be so successful.”

> People do this because they don’t want to burst your bubble. You’re coming to your friends, you’re coming to your co-workers saying “I got this cool idea for an app”—no one’s going to tell you that idea sucks, because they want you to not have your confidence destroyed. This is a really good book that hammers that concept home for you and then gives you a framework for trying to get around the fact that everybody’s kind of lying to you when you pitch your ideas.

---

### The Rules

To keep this challenge focused and measurable, I’ve set a few constraints:

1. **One app at a time.** I will finish and ship each app before starting the next one.
2. **Solve real problems.** Every app will target a niche pain point, preferably one I understand personally or professionally.
3. **No ads.**
4. **Transparent metrics.** I'll publish metrics such as number of emails obtained from Splash page and Plausible analytics.

---

### Why I'm Doing This

I’m a software engineer with a background in both trading and product development. I’ve built for others for over 12 years, but I’ve always wanted to apply those skills toward creating something of my own. The goal of this body of work is to develop an income stream. If an app gets a reasonable amount of users, I plan to blog about the process of setting up an LLC.

---

### What to Expect

Each entry in the *Twenty Apps* series will include:

* The idea and target user
* Research and competitors
* Issues refining the LLM agent prompt
* Traffic metrics from [Plausible Analytics](https://plausible.io/)
* Post-mortem: what worked, what didn’t
* Next steps (pivot, abandon, improve?)
* All projects will be posted on my personal website [here](https://iacut.one/software.html)

---

### App #1 – Email Collector

The first app is already underway. It turned out pretty well in terms of code produced by Claude Sonnet from Cursor. This is sort of a meta project. I have tried many of the email collector services such as [MailChimp](https://mailchimp.com/), [ConvertKit](https://kit.com/) and [MailerLite](https://www.mailerlite.com/). These apps have lots of features I do not need. They also embed Javascript onto your website which I do not want. Furthermore, there are draconian limits to the amount of campaigns you can create--MailChimp only allows two campaigns.

My app is dead simple. Provide an endpoint to users for POST actions to insert email records into a database. Users can embed a simple POST into a form on their website. Additionally, the app provides a GET endpoint to return a list of email records. Simple.

Here is the Splash website and GitHub [repo](https://github.com/iacutone/email_collector/). I plan to keep this app running permanently in a Docker container because I often reach for an email signup as confirmation of an app idea in a splash page. And I don't want to use the existing email solutions.

---

### Follow Along

I’ll be posting each update here. You can subscribe to the RSS feed to follow the journey.
