---
layout: post
title: "Getting Gud at LLMs Pt5"
date: 2026-03-04
categories: blog
---

In [Part 4](/blog/2026/02/25/getting-gud-at-llms-pt4.html), I finished planning for the big order orchestration saga and then started implementing. I am in the middle of implementing it but I decided to go on a little side quest. Things were getting complacent and I wanted to shake things up.

---

## Problem and Solution

I was already falling into established patterns because I was working like how I would work at a real serious full time job. Using LLMs to enhance my speed by outsourcing typing but not going all in. I found a pace I was comfortable in. But, I was definitely not satisfied. The fact that human intervention is needed or LLMs speed you up is not revolutionary. Plenty of other people are saying it too. Also, I wanted to challenge myself more with the single agent setup. So I changed 2 variables:

1. I sped up the pace of work. I was going reasonably slow-ish pace compared to other devs who embraced this fully so I have room to improve there.
2. The amount of human review and intervention needed to be reduced. I still meticulously read the code generated in Koupang and directed refactoring efforts.

I didn't want to apply this to Koupang, so I decided on a new project. Tangent: [Jevons Paradox](https://en.wikipedia.org/wiki/Jevons_paradox) "is said to occur when technological improvements that increase the efficiency of a resource's use lead to a rise, rather than a fall, in total consumption of that resource". This applies because now if I have an idea, I can just execute. I started the second project because now, the cost of starting is so low that I can afford to take a quick jaunt at something completely unrelated and come back to the original project without losing so much time.

As I developed Koupang, I felt that task management was solved with [beads_rust](https://github.com/Dicklesworthstone/beads_rust) but other stuff like knowledge management was not so convenient or agent first. I looked into [Flywheel](https://github.com/Dicklesworthstone#the-agentic-coding-flywheel) but I didn't really want to install 10+ tools and learn all of them. So instead, I decided to build my own: [Filament](https://github.com/JYC11/filament). I wanted a single rust binary with all the tools I needed. I directly got Claude to research the tools and codebases of my inspiration and with some of my input, we began building.

## Process and Outcomes

I took an evening or so to plan and research. Then the next day, I started executing. I decided to give myself a day to get as much done as possible since I wanted to go back to Koupang ASAP. Thus, I went fast and reduced human intervention (the 2 variables I talked about above). Cracks immediately started to show. The cracks were:

- programming shortcuts taken (eg: N+1 queries, LLM didn't just write a new query to get things in batch)
- code quality issues (eg: god functions, circular dependencies)
- bugs (too numerous to count)

There is quite a lot to cover so I got Claude to summarize the interactions I had below. But in the end, I mostly managed to finish all the features I wanted. I will need to do some extensive QA but after, I plan to implement this into my workflow afterwards. I am currently planning and executing an aggressive QA part of the development where I will try to break what I built using Claude as I write this blog post.

## The Numbers

- **40 sessions**, **227 prompts** over ~1.5 days (Mar 2 evening → Mar 4 morning)
- **42 commits**, **~15,000 lines of Rust** across 4 crates
- **235 tests** (120 core + 58 CLI + 39 daemon + 10 MCP + 8 TUI)
- **20 ADRs** documenting architecture decisions
- **5 phases completed** (Core, CLI, Daemon+MCP, Agent Dispatching, TUI)
- **~6 code review sessions**, **2+ manual QA rounds**
- Multiple context window exhaustions (sessions continued from summaries)

## Prompt & Progress summaries by AI with my takes in between as usual

### Sessions 1–2: Planning & Architecture Decisions (Mar 2 evening)

Copied the Makefile and util-scripts from Koupang as a starting point. Wrote 6+ Architecture Decision Records. Key decisions made: messages are NOT graph nodes (separate inbox/outbox pattern), file reservations and agent runs also not graph nodes, single-binary architecture (installable via `curl` not just `cargo`), and per-project `.filament/` directory with local SQLite + Unix socket.

<details markdown="1">
<summary>Prompts</summary>

```
/project-context some chore work, let's copy over the makefile and util-scripts from koupang
```

```
let's record the current architecture decision records
```

```
for inter-agent messaging are the messages stored as graphs as well? that doesn't seem to make sense to me
```

```
should we adopt an inbox/outbox type structure for messages?
```

```
let's do single-binary because I want it to be installable via curl as well, not just cargo
```

</details>

> **My take:** I decided to do a significant amount of planning. What isn't captured here is in other sessions with the LLM where I explicitly planned knowledge graph tools, multi agent managing with bash scripts, tool research/brainstorming and naming the project. They have been left out but yeah I did them.

### Sessions 3–4: Phase 1 — Core Library (Mar 3 morning)

Researched beads_rust's JSONL/flush design for comparison. Implemented the core library: models, errors, schema, store, graph, connection, protocol. Value objects (Priority, Weight, NonEmptyString) to make invalid states unrepresentable. Code review + test review against the test guide, marked Phase 1 complete.

<details markdown="1">
<summary>Prompts</summary>

```
could you assume any reasons why beads_rust uses jsonl and a flush feature?
```

```
let's start implementing
```

```
can you review the tests based on the test guide? also mark phase 1 as complete in the plans
```

```
look through the codebase and see if there are more opportunities to use value objects
```

</details>

> **My take:** I mostly did directing as I developed this part. I wanted this part to be airtight because it is literally the core. I implemented the stuff I talked about in previous posts (ADTs, value objects, etc) for correctness and hoped that maybe Claude will follow the pattern as it continued the project (it didn't lol). I still read the code Claude generated.

### Sessions 5–7: Phase 1 Polish & Code Reviews (Mar 3 ~12:00–13:30)

Two rounds of code review — found 3 bugs, made 4 improvements. Added ADR-018 for value types/newtypes. Created a gotchas document since the project was already accumulating pitfalls (sqlx custom newtypes, thiserror v2 `source` field behavior, petgraph 0.7 API changes).

<details markdown="1">
<summary>Prompts</summary>

```
I don't think priority should be i32 because I don't want negative priority. Also, I want invalid states to be unrepresentable
```

```
let's do a code review session for phase 1
```

```
what about the macros for the repetitive code?
```

```
let's do another code review over phase 1 just in case
```

</details>

> **My take:** I was still quite slow during these sections. I wanted things to be airtight.

### Koupang Interlude 1: Outbox & Shared Module Code Review (Mar 3 ~13:30)

Switched back to Koupang briefly. Did a code review of the outbox implementation, fixed 8 issues. Then reviewed the shared module, found and fixed 9 more issues. Context window ran out mid-session.

<details markdown="1">
<summary>Prompts</summary>

```
let's do a code review of the current outbox implementation
```

```
let's fix all of the issues from 1 to 8
```

```
let's do some more code review on the shared module, we can skip outbox since that was done.
```

```
let's fix them all 1 to 9
```

</details>

> **My take:** After realizing I could even offload the review work to Claude, I tried it with Koupang as a trial run to see how it was. I think I'm somewhat satisfied with the performance.

### Sessions 8–9: Phase 2 — CLI (Mar 3 ~13:40–14:00)

Implemented all CLI commands: entity, task, relation, query, message, reserve. 27 integration tests. This was one of the fastest phases — two sessions, plan then execute.

<details markdown="1">
<summary>Prompts</summary>

```
let's do phase 2
```

```
Implement the following plan: Phase 2: CLI Implementation Plan...
```

</details>

> **My take:** Still reading the code and I still have a good idea of the codebase.

### Sessions 10–11: Phase 2 Code Reviews + Manual QA (Mar 3 ~14:00–15:00)

Two code review rounds — fixed 5 bugs, 3 architecture improvements, 18 new tests. Created a manual QA skill in `.claude/skills/` for structured end-to-end testing. Ran 50 manual test cases. Decided on dual-track project management: keep `.md` files as committed source of truth AND use filament's own knowledge graph for live tracking.

<details markdown="1">
<summary>Prompts</summary>

```
let's do a codereview. bugs, test coverage, test cases, architecture improvements should be the focus
```

```
before actually using filament for self-tracking, I want you to do some manual QA with some dummy information
```

```
the manual qa should be a proper SKILL.md in .claude/skills
```

```
the test results should have date-time in the file name as well in case of multiple tests within the same day
```

</details>

> **My take:** I started off by being quite aggressive with reviews, fixing and refactoring. This changes as time passes.

### Sessions 12–14: Phase 3 — Daemon (Mar 3 ~15:00–16:00)

Implemented Unix socket daemon with NDJSON protocol. 9 daemon integration tests. CLI now routes through daemon when running, falls back to direct DB access otherwise.

<details markdown="1">
<summary>Prompts</summary>

```
final round of manual qa and code review before importing project docs, tasks, context into filament
```

```
let's do phase 3
```

```
Implement the following plan: Phase 3: Daemon Implementation Plan...
```

</details>

> **My take:** This is where I started going much faster I think. I started reading the code less.

### Sessions 15–18: Phase 3 Bug Fixes & Dogfooding (Mar 3 ~16:30–17:30)

This is where things got interesting. Manual QA of the daemon revealed that neither `create_entity` nor `update_entity_status` were creating events — the store layer just didn't do it. I accidentally deleted the tmp directory at one point. Started using filament to track its own tasks (dogfooding). Refactored the daemon handler "god function" into 7 domain sub-modules. Added multi-agent concurrency tests.

<details markdown="1">
<summary>Prompts</summary>

```
Neither create_entity nor update_entity_status creates events. Events must be created separately.
```

```
add that as a task on filament
```

```
I have accidentally deleted the tmp directory
```

```
I want you to add test cases of cli connecting to the daemon for multiple agents using multithreading
```

</details>

> **My take:** I got an idea that Claude can probably manual QA this and it should because this is an agent tool first so I did it. I'm glad I did. It also exposed bugs and shortcuts Claude was taking (not fully implementing the event store feature) so it was a good call. Another effect of going fast.

### Sessions 19–20: Phase 3 Refactoring (Mar 3 ~17:50–18:30)

Handler refactoring and code review of Phase 3. Deduplicated gotchas from MEMORY.md into a proper gotchas document + filament knowledge graph. Context window ran out during this — had to continue from summary.

<details markdown="1">
<summary>Prompts</summary>

```
I want to do some refactoring of the big handler function
```

```
I want you to deduplicate some content in .md files. In memory.md there are a bunch of gotchas, those can be in a proper gotchas markdown file and in filament as well
```

</details>

> **My take:** A consequence of going too fast. I noticed a god function and cruft being built up in .md files. I decided to slow down and fix things.

### Sessions 21–24: MCP Server (Mar 3 ~18:25–20:00)

Planned and implemented MCP server using the `rmcp` crate — 12 tools via stdio transport. Code audit: removed dead code, fixed clippy warnings, added 4 new MCP tools. Manual QA of MCP implementation.

<details markdown="1">
<summary>Prompts</summary>

```
let's do the MCP server, start planning
```

```
put the plan into filament as tasks
```

```
let's do a code review and try to also fix warnings given by clippy that got bypassed with allows
```

```
let's do some manual testing of the mcp implementation
```

</details>

> **My take:** Making fast progress but doing more reviews and manual QA as needed.

### Sessions 25–28: Major Refactoring — Slugs + Entity ADT (Mar 3 ~20:00–21:30)

This was the biggest cross-cutting change. Identified the name collision problem — entities were looked up by name which could overlap. Switched to 8-char base36 slug identity (ADR-019). Then refactored `Entity` from a flat struct to a tagged enum (`Task | Module | Service | Agent | Plan | Doc`) with typed variants and `TypeMismatch` errors for compile-time safety (ADR-020). Then further type-safety improvements, replacing runtime `is_task()` checks with pattern matching.

<details markdown="1">
<summary>Prompts</summary>

```
the entities are related/found using names which I think could overlap. beads_rust uses a randomly generated slug to identify and match
```

```
Implement the following plan: Refactor: Slug-Based Identity + Entity ADT...
```

```
we did a big ADT refactoring all over, let's do an analysis of the codebase on similar issues
```

```
the features require task and knowledge graph management both but it seems that it only resolves to task and agent, is this correct?
```

</details>

> **My take:** I knew this part would be massively breaking and set back progress but I didn't want problems in the future so I did it. This meant that I had to stop using filament to self check filament (easier and faster to just trash everything). This is I think where the bigger cracks started to show in my workflow. I was going fast and not fully reviewing the code well because the amount of code was overwhelming. Maybe I could have prompted better or provided better context.

### Sessions 29–31: Phase 4 — Agent Dispatching (Mar 3 ~21:30–22:30)

Implemented the dispatch engine: spawn subprocess via `std::process`, monitor via `tokio::spawn`, parse `AgentResult` JSON, route messages, death cleanup (revert task, release reservations, refresh graph). Agent roles: Coder, Reviewer, Planner, Dockeeper with compiled-in prompts and tool whitelists. 23 new tests.

<details markdown="1">
<summary>Prompts</summary>

```
let's do phase 4 and before we start, let's make sure the docs are up to date
```

```
Implement the following plan: Phase 4: Agent Dispatching...
```

```
let's do the next priorities that are in MEMORY.md
```

</details>

> **My take:** I was getting tired but pushing through. I let Claude plan and just do it and decided to catch bugs later. At this point, I was barely reading the code. A mistake but I guess deliberate. I wanted to go fast. That was the purpose of this experiment/project.

### Session 32: P1 Bug Deep Dive (Mar 3 ~23:00)

The most complex debugging session. The dispatch engine had a child reaping race condition — when the server batched multiple agent dispatches, child processes could be reaped by the wrong handler. Fix: `std::process` + remove server-side batch dispatch, CLI `dispatch-all` now loops individual `dispatch_agent` RPCs instead.

<details markdown="1">
<summary>Prompts</summary>

```
let's fix the P1 bug and before you do, explain to me the dispatch code logic/overall structure and how the P1 bug occurs in detail
```

</details>

> **My take:** This part was genuinely technically challenging because multithreading/multiprocessing is not an area I am well versed in. I made an executive decision to just do things sequentially instead of chasing this. I will have to sit down and study this part in detail in the future (I hope).

### Sessions 33–34: Phase 5 — TUI (Mar 3 ~23:10–23:30)

Implemented ratatui-based TUI with task, agent, and reservation views. 7 TUI tests. Fastest phase to implement.

<details markdown="1">
<summary>Prompts</summary>

```
let's do phase 5
```

```
Implement the following plan: Phase 5: TUI Implementation Plan...
```

</details>

> **My take:** Final push to get shit done.

### Sessions 35–38: Code Reviews & Bug Fixes (Mar 3 23:40 – Mar 4 02:00)

Phase 5 code review fixed N+1 query patterns in both CLI and TUI. Created `batch_get_entities` API in core to eliminate them. Then a comprehensive code smell analysis: where to use ADTs/value objects to make illegal states unrepresentable, where to simplify code. Created a 15-task code review plan. Completed all items: type-strengthened DTOs, CLI args, dedup utils, broke a bidirectional dependency.

<details markdown="1">
<summary>Prompts</summary>

```
I want to see if there are any more inefficient SQL usage patterns in the UI (cli and tui) and I want them fixed
```

```
where can we use ADTs/value objects to make illegal states unrepresentable?
where can we make the code simpler?
```

```
fix the bugs and code smells from session34 in .plan
```

</details>

> **My take:** I decided to do a review of the code with Claude. I was quite tired and honestly could not read everything myself. Instead, I directed Claude to look for things that would be code smells and bugs and got Claude to fix them. This made me think deeply about "what is good code?" and "how do I properly balance speed and quality in software development?"

### Koupang Interlude 2: Dev Rules & Autonomy Discussion (Mar 4 ~01:10)

Switched back to Koupang again late at night. Added development rules to Koupang's CLAUDE.md and had a discussion about code style preferences — god functions vs overly fragmented code. Then explored what additional subagents/skills/MCP tools would help Claude be more autonomous. Did some housekeeping on context-affecting `.md` files.

<details markdown="1">
<summary>Prompts</summary>

```
I have added some development rules into CLAUDE.md. Let's have a discussion about them
```

```
for a god function/class I think of a function/class where I have to scroll thousands of lines of code. for overly fragmented functions/classes I think of a function/class I have to jump around between dozens of files...
```

```
let's do some more guides to help you be more autonomous if needed, what other subagents/skills/mcp whatever would be useful. 1. in general? 2. for this project?
```

```
should we do some housekeeping on the context affecting .md files?
```

</details>

> **My take:** After thinking deeply about software and guidelines, I decided to go back to Koupang and put them in the CLAUDE.md. It forced me to introspect and clearly communicate what I want out of software which I think was good. This also made it clear to me why I had resistance to simply adopting other people's LLM workflows. I didn't fully know what the other people valued and what I would be feeding the LLM. I wanted full control and customization.

### Sessions 39–40: README, License & Aggressive QA (Mar 4 ~10:00–11:00)

Wrote a comprehensive README with installation and usage guide. Added MIT license and an inspiration section crediting beads_rust and Flywheel. Planned aggressive QA rounds targeting concurrency, state corruption, and edge cases. Pushed to GitHub.

<details markdown="1">
<summary>Prompts</summary>

```
I need you to write a comprehensive readme on installation instructions and how to use all of the features
```

```
added MIT license, also put in section about inspirations. I was directly inspired by beads_rust and flywheel but wanted one tool in rust that did everything for me
```

```
I want to do some aggressive QA where the goal is to not conservatively test but break stuff
```

</details>

> **My take:** I still think I didn't manage to catch as many bugs as I hoped. I kinda expected this but the goal was to go fast and finding problems within my workflow. I think I achieved the goals. As I am writing this blog post, I am conducting aggressive QA with Claude on the filament tool. I think this was overall a productive experiment. I discovered where the limitations in my LLM assisted workflows are and made me think about what I consider good software (a deeply contentious topic in the industry). I also reminded myself that you should challenge yourself more often because it's easy to fall into patterns.

## Observations

1. Reducing reviews led to more bugs and worse code. No surprise there. I also lost some understanding of the later parts of the code. Those can be rectified.
2. The speed felt sustainable at first but as things progressed, it did not. I was in a "vibe coding", ugh I hate that phrase, stupor and was just focused on shipping like I was in a super early stage startup.
3. I don't think I found the right balance in speed and review with this single agent setup. Maybe I will discover it later.
4. "Good code" is whatever you (or a team if you are working in a team) find comfortable to work in I think. I think my thoughts on good code will change as I progress in my career and meet more people and work on different projects. Right now, I am doing things alone and am sticking to what I am comfortable with and following principles I agree with ("A Philosophy of Software Design" is my preferred overall software guide).
5. Claude definitely is better at catching small subtle bugs than me while I am better at bigger code smells and code direction. I should try to get better at what Claude is doing (catching small bugs) but that could be difficult considering the amount of code produced. I noticed that big PRs get quick LGTM! from reviewers and yeah that ain't gonna change soon.
6. Going fast on Filament made me appreciate the slower more deliberate pace of Koupang. Working on Koupang felt like more proper engineering while Filament felt like a startup. I can do both but I learned that LLMs just enhance existing aspects. If you go fast and break things, LLMs will let you go even faster and break even more things. If you go slow and deliberate, LLMs will speed you up but it will help you be more deliberate. It really is all up to the user.


## What's next

I will finish Filament and maybe add a couple more features on the TUI side then I will return to Koupang (I have been implementing the outbox which is quite crucial before I got distracted). Stay tuned.
