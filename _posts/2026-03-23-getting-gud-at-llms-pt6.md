---
layout: post
title: "Getting Gud at LLMs Pt6"
date: 2026-03-23
categories: blog
---

In [Part 5](/blog/2026/03/04/getting-gud-at-llms-pt5.html), I went fast and broke things building Filament. This time, I bounced between 5 projects over 18 days — shipped Filament v1.0, built and shipped Jujo v1.0 from scratch, pushed Koupang's order saga to completion, attempted a C++ to Rust port, and started learning Haskell. I also overhauled my entire skill and workflow setup. Here's a snapshot of my current setup which is now quite evolved: [Current LLM Workflow Setup (March 2026)](/blog/2026/03/23/current-LLM-workflow-setup.html).

---

## The Numbers

- **~35 sessions**, **~300+ prompts** over ~18 days (Mar 4 – Mar 22)
- **5 projects** touched: Filament, Koupang, Jujo, RLedger, Haskell learning
- **87 git commits** across all projects (36 Koupang, 31 Filament, 20 Jujo)
- Filament: v1.0.0 released, 31 commits, extensive QA
- Koupang: full order saga implemented, STYLE.md adopted, 36 commits
- Jujo: built from scratch to v1.0.0 in ~3 days
- Haskell: levels 1–9 completed (exercises generate by Claude) and also did additional Exercism exercises

---

## Filament

Finishing this took longer than I thought. When I last left off, the TUI was nearly done but not quite and then I got distracted by my perfectionism. I did some extensive QA work which I thought was good to experience and managed to release v1.0.0. By the end of this 5 day sprint, I was absolutely exhausted because of how much focus I put in. It was a very different kind of tiredness which I am finding hard to describe with words. As I was in the frenzy of prompting, it felt extremely exhilarating to just Get Shit Done but my need for full knowledge and perfect verification also led me to do lots of reading of the code and constant re-examining of priorities. I was just "On" for so many hours of the day and obsessed with getting it done. I don't particularly wanna get back into this state again.

### Git History (Mar 4–22)

The filament sessions weren't captured by the prompt logger since it only ran in the Koupang directory at the time. But the git history tells the story — 31 commits from Mar 4 to Mar 22:

- TUI enhancements: message detail pane, keyset pagination, reply-to-message, 95 TUI tests
- Pre-v1 code review: 18 fixes across all 4 crates, typed entity DTOs, Clearable<T> enum
- QA: 7 QA sessions (22/22 tests passing, 0 bugs), roleplay simulations for multi-agent coordination
- Bug fixes: flaky daemon test race condition, error exit codes, circular dependency prevention
- v1.0.0: CI/CD pipeline, curl install script, distributable skills

> **My take:** The filament git log is dense. 31 commits in ~18 days but the actual work was front-loaded into maybe 5-6 days of focused sessions. The roleplay simulations (where Claude pretended to be multiple agents using filament concurrently) were genuinely quite useful and I think using LLMs for QA has a lot of potential.

---

## Koupang

I went back to Koupang after some rest where I did nothing. Instead of going more headfirst super-fast like Filament, I decided to take things a bit slower and more deliberate. I went over the generated code and revised plans multiple times. I eventually iterated enough on the plans and read enough of the code that I got a good understanding of what is going on and what to do and began implementing. At this point, I also started doing LLM-assisted code reviews. I was still kinda recovering from the mad-sprint as I was doing this.

I think the code that was output was pretty decent and well tested. I learned a fair bit (mostly big picture) about outbox and kafka setup from this which was good. The very detailed implementation details elude me to be honest but considering these are kinda one-off things and not repetitive like basic CRUD work, I think it would take me far longer to internalize. The full order flow is now complete and I just need to fully verify it works as expected.

### Kafka Consumer/DLQ Planning & Outbox Review (Mar 5)

Reviewed and critiqued the Kafka consumer/DLQ plan for edge cases and production quality. Then turned the same rigor on the existing outbox implementation. Created improvement plans for both. Also got book recommendations for practical Kafka knowledge.

<details markdown="1">
<summary>Prompts</summary>

```
let's go with bd-3sv
```

```
I would like for you to critique it more for edge cases and production quality
```

```
could you also critique the current outbox implementation with the same rigor (edge cases and production readiness)?
```

```
the improved kafka plan and this outbox improvement plan, we should track them in beads
```

</details>

> **My take:** This was the deliberate pace I wanted after my Filament burnout. Plan critique → outbox critique → track in beads. No rushing to implement. I was still using beads at this point. Idk how other people go full yolo for long periods of time. I don't have it in me.

### Migrate from Beads to Filament (Mar 10)

Migrated all beads tasks and project documentation into filament. Verified migrated data, filled in service entities, and updated global skills to reference filament instead of br.

<details markdown="1">
<summary>Prompts</summary>

```
/filament I recently completed the filament project and I want to migrate the beads in /br into filament along with project documentation. I want to actively use this tool in this project.
```

```
each plan needs a doc so the minimal mvp-milestone.md could be filled in in the future so leave it. the other stuff I think we can leave for record purposes
otherwise, just do a double check of ALL the migrated data
```

```
update the project_context and handoff skills to reference filament instead of br. they are located in the global .claude directory
```

</details>

> **My take:** Eating my own dog food. I built filament for this exact purpose and it was satisfying to actually use it. The migration from beads was straightforward because I designed filament to handle the same concepts. Was slightly worried things may not work well but as I kept on using it, it wasn't that bad.

### Kafka Implementation Sprint (Mar 13–14)

Added gstack review skills and made them generic. Switched Kafka to `apache/kafka-native` image for smaller footprint. Implemented Kafka event consumer with DLQ support. Did outbox improvements. Worked through shared infrastructure tasks (caching, tracing). Ended with a shared code cleanup sweep.

<details markdown="1">
<summary>Prompts</summary>

```
before we go further, I want to add these skills: https://github.com/garrytan/gstack
only the ones relevant to us
```

```
/filament we will continue with the kafka implementation as we have planned
I would like to switch the kafka image we are using with this: https://hub.docker.com/r/apache/kafka-native
```

```
can you explain how adding to the dlq results in a retry?
```

```
/filament I want to work on the next shared tasks before we move on to payment and orders
```

```
/filament I know there are additional tasks but I want a cleanup sweep of the shared code that was implemented. add tests, fix bugs and do a code review
```

</details>

> **My take:** The Kafka DLQ discussion was educational — I genuinely didn't understand the retry mechanism at first and Claude walked me through it clearly. I'm glad I asked instead of just accepting the code. The gstack skills turned out to be useful for structured code reviews. I particularly like the stop and offer user options style. It forces me to sit and think instead of just waiting for output.

### The Big Koupang Day (Mar 15)

This was a marathon session. Refactored service structs from OO-style to free functions. Researched a friend's payment project and Toss (fintech company) articles. Adopted TigerBeetle's TIGER_STYLE.md, consolidated into a project STYLE.md. Created a cleanup skill. Did STYLE.md compliance refactoring across the codebase. Started order/payment service scaffolding with DOP business rules and property testing.

<details markdown="1">
<summary>Prompts</summary>

```
/filament before we continue, I want to do see if we can make the various "service" structs less OO. Right now, it's structs with DI and it's "fine" it works but if there is a way to avoid OO-ness then I would like to
```

```
/filament before we go onto payments let's do some research. this project is being done by a friend and I think it contains some useful things we may not have considered
```

```
I want to adopt this: https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/TIGER_STYLE.md where relevant and keep a version of this in our project repo. use the /research skill
```

```
let's make a stale-files-cleanup skill since I want to do this semi-frequently
```

```
let's start from the P1 tasks and work through them one by one
```

```
all the quick wins should be done, for outbox only do redis dedup for relay edge case (add test for this case) and others defer
```

</details>

> **My take:** This was one of the most productive single days. The OO → free functions refactor was something I'd been thinking about and Claude handled it cleanly. Researching my friend's payment project gave me ideas I wouldn't have had otherwise. Adopting TIGER_STYLE was a pivotal decision. It gave me a concrete reference for what "good code" means in this project rather than vague directions from me. The STYLE.md compliance refactoring that followed was extensive but worth it. I'll probably keep doing this for future projects.

### Integration Tests & Saga Wiring (Mar 16)

Implemented integration tests and Kafka consumer wiring for the order/payment saga. Did a refactor where event handlers properly use PgConnection within transactions. Code review of the full saga. Documentation and filament knowledge graph audit.

<details markdown="1">
<summary>Prompts</summary>

```
/filament start the integration test tasks as well as the kafka consumer/producer tasks for the order/payment saga
```

```
do the refactor where the event handlers properly use pgconnection and wraps things within a transaction
```

```
shouldn't we be using process with retry? not sure why there are two try process once and process with retry and the conditions for using each.
```

```
/code-eng-review of the saga that we implemented
```

```
I would like to add a task for implementing a background jobs feature from the shared crate with persisted jobs
```

</details>

> **My take:** I tried to understand what the hell was going on because this part was genuinely the most interesting part of the project so far. I really wanted to make this production quality. The engineering review skill paid off by catching issues I wouldn't have noticed in a manual read-through. However, there could still be issues that I may not know about. But this is a learning project (and I know for a fact that many other "real" codebases don't have "high" quality code due to how software projects are usually run: rushed, low resources and untested/unreviewed)

### Jujo Generators in Koupang (Mar 19)

Used the newly built jujo tool to analyze existing code patterns in Koupang and generate service scaffolding templates. Tested by generating a shipping service, verified the output, then removed it.

<details markdown="1">
<summary>Prompts</summary>

```
/filament /jujo /pattern-analyzer let's analyze the existing repeated code patterns and then add them to jujo
```

```
generate some for shipping like before and then remove them later after I finished reviewing
```

```
okay remove the generated shipping files
can you estimate token savings using this tool vs reading files + finding patterns + generating code?
```

</details>

> **My take:** This was the payoff. Jujo generated a full service scaffold that would have taken Claude many thousands of tokens to analyze patterns and produce from scratch. I'm very satisfied with this. Adding more determinism for LLMs is the way to go. I would like to add more going forward.

### Docker Sandbox & Permissions Research (Mar 21)

Explored running Claude Code in isolated Docker sandbox mode. Researched vibebox and various sandbox approaches. Also expanded the allowed bash commands to reduce permission prompts.

<details markdown="1">
<summary>Prompts</summary>

```
using this: https://docs.docker.com/ai/sandboxes/get-started/
I ran claude in docker sandbox mode but all of the local setup I have (skills, CLIs, global CLAUDE.md) don't carry over
```

```
can you also research this: https://github.com/robcholz/vibebox
```

```
I still have to frequently allow safe bash commands to run which I want to remove
```

```
oh then nevermind about this project, I will have to look for a new solution to run claude code dangerously skip permissions in an isolated way
```

</details>

> **My take:** The sandbox research was a dead end for now. None of the solutions carry over my skills, CLIs, and config cleanly. The permissions expansion was more immediately useful. I went from constantly approving safe commands to rarely seeing permission prompts. I may have to return to this at a later date and just build a solution for this on my own (isolated VM + export LLM configs). Trying to make a good harness for LLMs is proving to be an interesting challenge that keeps coming up. First Filament, then Jujo and now this.

---

## RLedger

When I heard about the ledger-cli I was quite intrigued at its ability to handle various different kinds of "assets" in a single double-entry accounting format. I shelved this fascination and just kinda forgot about it. Then in the middle of Koupang I just decided I would try porting the code over from C++ to Rust and trying to understand how the ledger-cli worked. I got through 7/8 phases planned and just had the test cases to convert then I got distracted again and did not finish lol. I also barely read the code this time since I wanted to read the codebase in full when it was all done. I can return to this any time I want to finish it off and just read and learn. This was also an experiment to see how well Claude would do in porting over a codebase that I had no idea about. Not a super high priority but I will return to it. I decided to keep this only locally because I think doing a "heist" of an open source software by rewriting it in another language for no good reason is kinda a shitty move.

No session logs for this one. I did it without the prompt logger running. It was a spontaneous detour. The port seemed to go well but I'm not sure how "good" it is. This was also done in preparation to see if I could use Claude Code to port over Vinyl Cache to Rust cleanly.

---

## Jujo

I originally had an idea of wanting to make a Ruby-on-Rails like meta framework but in Rust with Sqlite and Datastar. It was to be a simple boilerplate generating SaaS template. I got this idea a while back but real life work and just the activation energy to trial-and-error my way across multiple unknown libraries that I was unfamiliar with after a long day at work just pushed it to the side. I eventually just decided to have a go at it. While in the research and ideation phase with Claude, I realised that the "code generation" part was the more interesting problem to solve. I noticed in Koupang that the LLM was spending a lot of tokens just reading and understanding the existing codebase for patterns and re-implementing them. This felt like a waste of time and tokens so I decided to extract this part out of the project and pursue it instead.

With my learnings from Filament and Koupang (both very large and ambitious projects), I decided to keep this short and very simple. Do one thing very well. So I planned and researched with Claude and got it done in 1 long session (thanks to the new 1M token context window update) and then used a few shorter sessions afterwards to polish and release it. I am happy with my restraint and the fact I got it done quite fast without burning out again. I did also not pay a lot of attention to the code because I cared more about getting it done and because it was quite simple.

### Ideation & Naming (Mar 17 evening)

Spitballed the code generation idea. Explored Korean words for "stamp" as potential names. Tried "dojang", "gakin", and others before settling on "jujo" (Korean for casting/mold).

<details markdown="1">
<summary>Prompts</summary>

```
/filament I was thinking how to make working with LLMs more efficient
I already realized that consistent patterns across the codebase is very important
Why not make this cli codegen tool generalizable?
```

```
let's call it `dojang` which is korean for stamp but romanized
```

```
let's get synonyms for stamp first, get the romanized korean versions and check if they exist on crates
```

```
let's do jujo, gakin could be pronounced as gay-kin which is wrong
```

</details>

> **My take:** The naming discussion was fun. I wanted a Korean word that non-Korean speakers could actually pronounce. "Jujo" worked out perfectly. It's short, memorable, and the crate name was available.

### Planning → v1.0 in One Long Session (Mar 17 night)

Used /spec-driven-dev and /grill-me for the planning phase. Explored Tera templating. Implemented all 5 phases (TOML parse + Tera render → field parsing → injection/dry-run → discovery commands → AI customization markers) in a single continuous session. Code review. Added formatter hooks. Multiple live demos in /tmp directories between phases.

<details markdown="1">
<summary>Prompts</summary>

```
/spec-driven-dev let's continue with the planning phase /grill-me as well
```

```
for the templates itself, I was thinking of intellij style codegen. you can add templates which are actual code but with areas you can slot in variables
```

```
I don't quite understand what Tera is and how it fits into all of this to be honest
```

```
I would like to see phase 1 in action in a directory somewhere step by step
```

```
commit and move on, also clean up tmp directory of stuff
```

```
/code-eng-review
```

```
I would like a live demo of the formatter hook in action
```

</details>

> **My take:** This was a nice test run of the new workflow (described further down the blog). Planning → implementation → demos → review → ship, all in one evening. The 1M context window made this possible. In earlier sessions I would have run out of context mid-implementation. I did live demos between each phase which gave me confidence the code actually worked before moving on.

### Polish & v1.0 Release (Mar 18–19)

Added HTML/CSS language support. Ran QA with Claude across multiple languages. Added CI/CD, install/uninstall scripts, Makefile. Published v1.0.0 with curl install support. Registered skills in the library catalog.

<details markdown="1">
<summary>Prompts</summary>

```
/filament let's add html and css support then update the qa plan to include them
```

```
/filament let's begin the qa
```

```
let's release v1 and also give an option to install with curl
```

```
let's install jujo from source and use it in other projects
but before we do, let's do a review of the skills we will be using and add it to the local /library
```

</details>

> **My take:** Keeping scope small paid off. Filament took ~5 days of intense work. Jujo took ~3 days of relaxed work. Both shipped v1.0. The difference was scope discipline. Jujo does one thing (code generation from templates) and does it well.

---

## Meta stuff

Across the 4–5 projects I worked on, I kept noticing friction points in my workflow. I wrote them down and then actually tackled most of them in a single marathon session:

- **Done:** formalized review and planning skills (grill-me, plan-eng-review, code-eng-review, spec-driven-dev)
- **Done:** central skill library with catalog (forked disler/the-library)
- **Done:** proper coding STYLE.md (adopted from TigerBeetle's TIGER_STYLE)
- **Done:** combined all skills into a spec-driven-dev workflow
- **Done:** proper research skill using a Go CLI instead of random Python scripts
- **Done:** expanded allowed bash commands to near-zero permission prompts
- **Partial:** more rigorous testing (added property testing, mutation and fuzz testing still TODO)
- **Partial:** .md file auditing (did cleanup passes but this is ongoing)
- **Dead end:** running Claude Code in "dangerously skip permissions" mode in an isolated environment (Docker sandbox and vibebox both didn't work the way I wanted)

### Skill Ecosystem Overhaul (Mar 17)

Massive meta session. Researched mattpocock/skills (16 skills) and garrytan/gstack (15 skills), compared to my 18 existing skills. Installed 4 planning skills (grill-me, write-a-prd, prd-to-plan, prd-to-issues). Forked disler/the-library for private skill distribution. Created spec-driven-dev workflow (Research → Plan → Implement with human checkpoints). Wove filament into all skills. Installed and customized triage-issue. Removed br and bd-to-br-migration skills.

<details markdown="1">
<summary>Prompts</summary>

```
research this https://github.com/mattpocock/skills for skills we can add
```

```
research this https://github.com/disler/the-library
research and think about how to implement this
```

```
this is from a video by a netflix engineer about using AI effectively in large codebases but I think it can be generalized to a meta skill that refers to multiple skills to define a workflow
```

```
I would like to weave in filament into all this because filament has tasks, plans, lessons etc.
```

```
I think we should install the bug fix skill and weave in filament into that as well. make sure it uses lessons
```

</details>

> **My take:** This session transformed my setup. Going from 18 ad-hoc skills to 20 integrated skills with a library catalog, a proper planning pipeline (PRD → plan → issues → implement), and filament woven throughout was a step change (which I hope fixes the issue where Claude wasn't using it as much as I wanted). The spec-driven-dev workflow in particular has become my default way to start any non-trivial feature. See the [workflow snapshot](/blog/2026/03/23/current-LLM-workflow-setup.html) for the full current setup. I just came across these things from YT recommended videos. I didn't really seek out these other skills. I also thought that maybe too many skills would become cumbersome but it was surprisingly easy to manage them.

---

## LLMs for learning

In preparation for a new position (which required me to do a lot of paperwork too), I turned towards Claude Code to help me learn a new language for the position: Haskell. I decided I wanted to learn to code by hand and do it more slowly before I start to use LLMs for work (although how I plan to use it will be slightly more conservative). Having an endlessly patient tutor who will answer any dumb question I have is proving to be very useful. I strictly told it to not give me full answers and to give me only hints. It's quite good at doing that but sometimes Haskell stumps me so much I just needle it until it gives me the semblance of an answer. I also used it to generate me some practice questions I could do.

### Haskell via Exercism (Mar 21–22)

Worked through levels 5–9 of custom Haskell exercises. Topics: pattern matching on ADTs, type classes (Show, Functor, Describable), Maybe/Either error handling, State monad (stack calculator), Writer monad, IO (guessing game, address book, word counter). Also set up a "build-your-own-dkv" distributed key-value store project as another TDD learning exercise.

<details markdown="1">
<summary>Prompts</summary>

```
I am doing level 5 in exercises
I'm not sure how to get the required data from the shape for the area and perimeter functions
```

```
for prettyPrint, I want to add addParens as a helper function inside prettyPrint itself. how do I do so?
```

```
I don't quite understand the state type
```

```
for stackCalc, I don't understand where the operator is coming from
```

For the build-your-own-dkv project:

```
this is a LEARNING project I want to emphasize that
you should heavily bias towards providing hints and resources I can read
```

</details>

> **My take:** The pattern of "I try → I get stuck → I ask for a hint → I try again → I ask for more help" works really well for learning. Haskell's type system is genuinely hard and having a tutor that can look at my code and tell me exactly where my type error is without giving me the answer is invaluable. The State monad section was particularly painful but educational. I went from "what is this?" to implementing a stack calculator with it in one session. The previous sessions were not recorded but they followed the same format.

I plan to use Claude Code for more learning projects in the future. CodeCrafters is cool but I think even if I pay for it, I will just forget to do it because of my propensity of getting distracted. So I kinda plan to use Claude Code like CodeCrafters as a learning tool. This time, I will code by hand as a learning experience but with a competent(?) tutor.

---

## Observations

I think the meta-cognition/meta-observation of your own thought/work processes and vigilance of how the LLM works is the biggest takeaway. I often hear people complain about LLMs not doing things the right way and going off rails but I wonder how much they tailor their context and split up their tasks so that the LLM is most likely to output the "right" thing.

Considering that LLMs improve very rapidly, all of the stuff I am doing may be useless in a couple months time. Hell, I already found that Claude can work with worktrees already with subagents which is what Filament is supposed to help with using the file locking feature and the build coordinating I planned to implement. It also has more primitive aspects of Filament which are just various .md files but I can see Anthropic and other AI companies converging on some graph-like knowledge management structure. I also found [dgraph](https://github.com/dgraph-io/dgraph) and [GitNexus](https://github.com/abhigyanpatwari/GitNexus) which also do what Filament does but more specialized. In addition, all of the tips and tricks I learned were from when the context window was quite small. Now that the context window is 1M tokens, the precautions I have to take are lessened but not completely irrelevant. The experience of using Claude Code got easier because I don't have to constantly close and reopen sessions.

---

## Going Forward

If all goes well, I will be far too busy to continue a lot of side projects I have going on now. I will most likely continue Koupang to see how big the codebase can get and do the CodeCrafters thing on my own. I do have a fundamental unwillingness to completely let go and not understand what the code being produced is like for codebases I consider important to me and I do genuinely want to improve as a software engineer. The Filament experience was quite eye-opening in how draining it is to just full send which was a big inflection point for me. I could bounce between multiple projects (2 or 3) manually but I don't think I'm comfortable going full Gas Town industrial code factory level.

But one thing I do want to share is tackling a refactor of a legacy codebase using Claude Code. There are so many legacy codebases in real life but I don't really see actual examples of how it can be tackled with LLM tools so I think I will do that and share. So future posts are:
- Overengineering Koupang for Fun and Profit pt{n} -> Koupang obviously
- Just Refactor It Dude pt{n} -> for refactoring legacy codebases
- smaller LLM posts (maybe I go off the deep end with Gas Town or figure out how to isolate LLMs better?)
- any miscellaneous thoughts I may have
