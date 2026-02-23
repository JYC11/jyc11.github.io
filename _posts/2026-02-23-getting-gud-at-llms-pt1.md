---
layout: post
title: "Getting Gud at LLMs Pt1"
date: 2026-02-23
categories: blog
---

## Background

Around this time last year, I was quite skeptical of LLM usage for programming. I do remember having a false sense of superiority that I programmed "the old way". However, I kept my eye on the progress of LLMs and used them more at work while I was employed.

I originally used the web interfaces (ChatGPT, Gemini, Claude, Qwen, Deepseek) to one-shot bash scripts, Golang scripts or SQL that I couldn't be bothered to write. Then I used the Canvas feature in Gemini to actually code/refactor certain features. My main programming use was providing samples of test code that I was happy with and getting the LLM to copy that format for other tests. Then eventually I caved and got the AI subscription from Jetbrains to use the LLMs within the IDE because copy-pasting to the browser was getting annoying.

After a while of deliberation, I decided to give this LLM thing an actual shot. Far too many people around me as well as people I've seen on Youtube have said that the industry is changing. From the 2 possible futures (software engineering industry changes fundamentally OR it doesn't), I just decided to give in to what seems like changing tides. I set out to challenge my own assumptions — mainly that LLMs are bad at dealing with large complex codebases and are only good at basic programming tasks.

Another sentiment I kept hearing was from staff-level and above engineers (via Reddit and YouTube) that LLMs boosted their productivity by a lot and they almost no longer code by hand. I imagine that very experienced engineers deal with far more difficult, larger and nebulous problems that require a lot of thought, planning and task subdivision — which are exactly what you need to do to use LLMs effectively from what I've heard. Considering that I am quite early career and don't deal with such difficult issues, I may not have such a large productivity boost with LLMs. So I figured the best way to understand why all these more experienced engineers are lauding LLMs was to try something larger and more difficult myself.

---

## The experiment

I wanted to build something simple first to get in the hang of things and I also wanted to use Rust for the heck of it. That became [Workout Util](https://github.com/JYC11/workout-util) — I already knew that LLMs were good at SQL and basic forms/pagination stuff due to the likely abundance of its training data so this project was a breeze. My main thoughts/reflections of using LLMs are in the readme of that project.

Now that I was done with a simple example, I decided to do something more complicated: [Koupang](https://github.com/JYC11/koupang/tree/main), an ecommerce backend. I am reasonably familiar with the ecommerce domain, it's a well established problem and I have worked on ecommerce backends so I decided to tackle this with Rust as well.

To facilitate a much larger use of LLMs, I decided to just pay money and use the **Claude Code Max** plan and work through the CLI. I originally scaffolded the entire project with Qwen (web interface), shoved that in a markdown file and then got Claude and started working on it.

---

## What surprised me

And so far... Claude seems to be doing really well. The first thing I worked on is the identity microservice. Arguably, the project is still small and identity/auth is a well established topic as well so the LLM would be good at it. But I am incredibly optimistic about this project because the use of Claude Code allowed me to make ridiculously fast progress despite only using Claude Code in a basic way. I am excited to see where this project goes and how the LLM can handle larger codebases.

### Specific points where I was surprised

- I assumed LLMs would be poor at using Rust due to there not being a large amount of Rust training data... I was wrong?
  - To be fair, I am using quite basic Rust and nothing too crazy
- The code produced by LLMs is not crazy spaghetti
  - It's not perfect code and I have to suggest better abstractions and pulling code out into the common shared package often but that's fine
- It's surprisingly good at handling niche crates like testcontainers and tonic/prost, which I'm pretty sure don't have a huge amount of usage data. My assumption about lack of training data is being slowly chipped away.
- I did barely any coding by hand.

---

## Claude Code setup/usage notes

Nothing fancy yet and no multiple sub-agents:

- Just basic prompting through the CLI after using `/plan` mode
- Often restarting sessions to clear context
- Trying to get the LLM to use beads rust but it doesn't really seem to be listening
- Being conservative with Bash script permissions just in case
- Being quite strict about human in the loop and reviewing the code it generates
- Scoping tasks quite tightly so context doesn't bloat

**Plugins:**

- [rust-skills](https://github.com/actionbook/rust-skills)
  - Kinda freaky how easy it was from the Claude Code CLI to add this considering prompt injection risks
  - Made sure to read through the skills to ensure they're not malicious but the more I use Claude Code, it's likely the more lax I will become

**Skills:**

- Create Skills meta skill from Anthropic
- Beads Rust skill that I made Claude create

---

## Why write about it

Even as I heard good things about using LLMs for projects, I was never really able to see the outcomes. Most people work on closed source codebases at their jobs, so I couldn't see the actual code being produced or how they had their tools set up — the prompting strategies, the configurations, the actual workflow. So this blog post is my attempt at showing the process for others who may be skeptical or curious. If there are transparent examples (actual complex codebases and well documented workflows), please let me know. I am eager to learn.

---

## Progress Log

I've also started keeping ADRs (Architecture Decision Records) in the repo to capture the why behind technical choices, and tagging milestones like `v0.1-identity-auth` so I can easily diff what changed between blog posts.

---

## What's next

In Part 2, I want to push Koupang further, I plan to work on the catalog service next, and see if the LLM can keep up as the codebase grows. Stay tuned.
