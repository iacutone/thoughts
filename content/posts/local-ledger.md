---
title: "Local Ledger"
date: 2025-11-16T10:46:08-05:00
draft: false
toc: false
images:
tags: 
  - entrepreneurship
  - elixir
---

# local\_ledger: A csv parser to convert csv data to the plain text accounting ledger format 

---

## 1. 🧠 The Idea

I to use an LLM to output the (Plain Text Accounting)[https://plaintextaccounting.org/] (ledger) format. However, I also don't want to upload my personal financial data to ChatGPT.

---

## 2. 👥 Who It’s For

**local\_ledger** is designed for:

* People that value their financial privacy
* People that don't want to methodically go through every credit card transaction and transform that record into the ledger format

---

## 3. 🔍 Research & Competitors

### Existing tools:

I don't see any existing tools to convert csv files into the ledger format that take advantage of LLM tech. Though, I have used tools like [reckon](https://github.com/cantino/reckon) in the past which are amazing. But using a tool such as reckon involves a lot of manual work.

### What I wanted:

* To upload a csv file in a form and have the output be a ledger txt file

---

## 4. ⚙️ How It Works

The tools I am using to create this application are the Elixir [Plug](https://hex.pm/packages/plug) library to talk over HTTP. I am using [Ollama](https://ollama.com/) running on my laptop with the `qwen2.5:7b` model to parse a csv file. Finally, I am using [ngrok](https://ngrok.com/) to tunnel from my laptop to a remote server in order to communicate with the LLM.

---

## 6. Learnings

TBD

---

## 7. Next Steps

I would like to implement another classifier to have more specific credit card categories represented in the legder output.

---

## 8. 🌐 Follow My Projects

This is my first of many experiments I’m building under a “20-app challenge.” I want to create and release 20 small tools to learn what developers want, what sticks, and what solves real problems.

- [iacut.one/software.html](https://iacut.one/software.html)

---

## 9. 💬 Try It / Give Feedback

Check out the repo:
- https://github.com/iacutone/local_ledger/
- Beta: https://local-ledger.com/
