---
title: "Email Collector"
date: 2025-07-13T20:35:45-04:00
draft: false
toc: false
images:
tags: 
  - entrepreneurship, elixir, phoenix
---

# 📬 email\_collector: A Privacy-First Email Signup Backend for Developers

**Posted on [iacut.one/software.html](https://iacut.one/software.html)**

---

## 1. 🧠 The Idea

Most email collection tools—like MailChimp or ConvertKit—require:

* Embedding third-party JavaScript
* Accepting branding or limited free tiers
* Surrendering user data to centralized platforms

My app is a dead-simple REST API for collecting and retrieving email signups, built with privacy and developer control in mind.

---

## 2. 👥 Who It’s For

**email\_collector** is designed for:

* Indie hackers launching landing pages
* Developers tired of heavy integrations
* Privacy-conscious teams who want to own their data

No JavaScript. No trackers. Just POST to an endpoint and GET back email signups later.

---

## 3. 🔍 Research & Competitors

### Existing tools:

| Platform               | Free-Tier Limits                                                                            |  Signup Requirements                | Key Limitations / Developer Notes                                                     |
| ---------------------- | ------------------------------------------------------------------------------------------- |  ---------------------------------- | ------------------------------------------------------------------------------------- |
| **Mailchimp**          | Up to **500 contacts**, **2,500 emails/month**, basic forms, templates, and CRM             |  Email, name, company & password    | Mailchimp branding, limited automation, daily send cap (\~500), forms with JS embeds  |
| **MailerLite**         | Up to **1,000 subs**, **12,000 emails/mo**, forms, automations, pop-ups, landing pages      |  Email, name, address               | Logo in emails, limited A/B tests & templates, time-limited support                   |
| **ConvertKit**         | Up to **10,000 subs**, unlimited emails, forms & pages, 1 automation sequence               |  Email, name                        | Branded emails, limited automations, recommendation ads slot                          |
| **Brevo (Sendinblue)** | Unlimited contacts, **300 emails/day**, forms, landing pages, automations, SMS              |  Email, org name                    | Email quota: 300/day (\~9k/month), Brevo branding, some features limited on free plan |
| **Klaviyo**            | Up to **250 profiles**, **500 emails/mo**, **150 SMS credits**, forms & flows               |  Email, org info                    | Limited send volume, branded content, limited support window                          |
| **Moosend**            | **1 form**, **1 landing page**, **1 automation** + trial features                           |  Email & contact list               | Very limited free features; devs should test paid plan                                |
| **EmailOctopus**       | Up to **2,500 contacts**, **10,000 emails/mo**, 3 automations (5 steps each), 3 forms/pages |  Email, name; no CC                 | Basic features only, branding link in emails, reporting only 30 days per campaign     |
| **Formspree**          | **50 submissions/month**, unlimited forms & projects, 1 team member, spam filtering         |  Email login; confirm email         | Supports plain HTML forms, domain restriction; no storage beyond submissions          |
| **Getform**            | **5 endpoints**, **1,000 submissions/month**, **1 GB file storage**, API & webhooks         |  Email, password                    | Limited submissions & endpoints; branding on thank-you page, limited retention        |


### What I wanted:

* No dependencies
* Generous campaign/email limits
* Limited data collection
* No branding on my website

---

## 4. ⚙️ How It Works

### POST endpoint for collecting emails:

```http
POST /api/collect
Content-Type: application/json

{
  "email": "user@example.com",
  "campaign": "launch"
}
```

### GET endpoint to retrieve signups:

```http
GET /api/emails?campaign=launch
```

This makes it easy to:

* Drop a form into your site with a simple HTML form
* Collect emails per campaign (e.g. "launch", "waitlist", "beta")
* Pull signups as a CSV or from the aforementioned GET endpoint

---

## 5. 🤖 Building It with an LLM-Powered Phoenix Workflow

While this app is relatively simple, I wanted to try something different: **vibe coding with [Cursor](https://www.cursor.so/)**, an AI-first code editor.

The result was surprisingly productive:

* The LLM scaffolded a basic **Phoenix API** with routing and plug-based validation
* It helped with edge cases, like ensuring emails were valid before insertion
* Even small touches—like timestamped logs or JSON responses—were quickly refined through iterative prompting

This made building the API **feel like collaborating with an assistant**, not just writing code solo.

---

## 6. Learnings

I used the Claude Sonnet 4.0 context window to vibe code the application. The following are brief issues I had while vibe coding:

- Could not figure out how to refactor the flash message in a desirable manner
- Generated css stylings in the HTML templates
- Made all CSS tailwind specific
- Made very weird decision with the Phoenix form helpers

---

## 7. Next Steps

Now I have an operational email collection app, I can easily collect emails on splash pages without the worries of using the current email collections providers. I plan on starting my next app using my email collections service, [Eola Dog Walkers](https://eoladogwalkers.com).

---

## 8. 🌐 Follow My Projects

This is my first of many experiments I’m building under a “20-app challenge.” I want to create and release 20 small tools to learn what developers want, what sticks, and what solves real problems.

- [iacut.one/software.html](https://iacut.one/software.html)
- https://collection.email or https://email-collection.com -- I am unsure which domain will be best for SEO

---

## 9. 💬 Try It / Give Feedback

Check out the repo:
📦 [github.com/iacutone/email\_collector](https://github.com/iacutone/email_collector)

If you use it on your landing page, I’d love to hear about it. PRs, stars, and suggestions are always welcome.
