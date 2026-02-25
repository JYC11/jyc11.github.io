---
layout: post
title: "Getting Gud at LLMs Pt3"
date: 2026-02-25
categories: blog
---

In [Part 2](/blog/2026/02/24/getting-gud-at-llms-pt2.html), I finished the catalog service CRUD, reflected on backend development with LLMs, and predicted that complex cross-service features would be where things get hard. Since then, I've been building out the catalog further and planning the next major phase.

---

I realized that the summary of what I did can be automated with Claude Code, so that's what I did. I've read through the summary and can verify it's accurate. I'll pepper in my own commentary in between (those sections are in blockquotes). When the AI summary says "I," it's Claude writing as me.

## By the numbers

About 12 sessions and ~50 user prompts across Feb 24 afternoon and Feb 25:

- 11 git commits since pt2
- Tests: 209 → 207 (net -2 from deduplicating redundant tests, coverage maintained)
- 2 new features: brands + categories with ltree hierarchy, keyset pagination with filters
- 4 major plans created for the next phase
- CLAUDE.md files compacted significantly (catalog 250→88, identity 125→68, shared 187→82 lines)
- ADR count: 8 → 9 (added ADR-009 for ltree categories)

The work fell into a few categories: building new catalog features, managing LLM context, testing and code quality, and planning for the order/payment phase.

---

## Building out the catalog

### Category & brand planning (3 prompts)

Added two new br tasks for categories and brands. Entered plan mode specifically for categories because:

- Tree data structures in a relational DB are tricky (chose Postgres `ltree`)
- Brand-category validation was needed (e.g. a car brand can't appear on food products)

### Brand & category implementation (7 prompts across 2 sessions)

Executed the plan. Implemented brands CRUD, categories with ltree hierarchy, and a brand-category association table. Asked Claude to work on tasks in parallel. Reminded it to reuse value objects (e.g. `HttpUrl` for brand logo URL). Ended the session when context started filling up.

> **My take:** I decided to experiment with two coupled features that have interdependencies with each other and are dependent on the existing product code, to see how Claude handles additions to existing code and manages a set of parallel intertwined tasks. I'm quite happy with how it handled itself once the plan was in place and shoved into beads. I got it to launch subagents to do the work as well when it could, to speed up task completion.

### Domain model clarification + scope control (5 prompts)

Clarified to Claude what I meant by "domain objects" — rich domain models where business logic lives, not just FK validation helpers. Claude suggested building FK traversal (following foreign key references to load related domain objects automatically), which I correctly flagged as ORM scope creep. Pushed it to a P4 backlog "nice to have."

```
to clarify, this is going into the realm of reimplementing orm features which
dramatically increases the scope of the task/project to unmanageable levels
```

> **My take:** The concept of "correct" code appeals to me greatly due to my work experience. When I worked in not-ideal legacy code environments or with fast-paced schedules, I would often skip writing tests and code as fast as possible. That obviously leads to poor and buggy code. Besides writing tests — which is an entire ordeal itself if the code is sloppy or very legacy — I wanted techniques to write code that _can't_ be wrong (assuming I have the correct understanding of the requirements).
>
> This led me to shop around for techniques, which landed me on several topics:
>
> 1. Functional Core, Imperative Shell
> 2. Making illegal states unrepresentable.
>
> The first concept taught me that code can be split into Logic (CPU-bound tasks or Calculation) and Side Effects (I/O or Data production/consumption). So whenever I look at or think about code, I decompose it into these two broad categories, which lets me reason about how to organize/debug code and choose technology.
>
> The second concept helped me understand that compilers can encode business logic. I learned that certain language features can be used as guards and as a representation for business logic — that the compiler can be made to encode and "understand" business logic. This was the specific "technique" I was looking for.
>
> Since LLMs can always make mistakes, I wanted another pre-emptive "layer" of validation besides what the compiler provides. If the LLM accidentally hallucinates something that isn't just technical and wrong in a business-logic sense, the compiler will be able to catch that as well. This specific case hasn't happened yet but I'm putting it in there just in case.

### Keyset pagination (3 prompts)

Planned and implemented keyset cursor pagination using UUID v7 ordering for product listing endpoints. Added a thumbnail image (sort_order=1) to the paginated product response. Skipped pagination for images and SKUs since they're low cardinality and only appear inside product detail views.

### Product filters (4 prompts)

Implemented `ProductFilterQuery`/`ProductFilter` for `GET /api/v1/products`. Filters by category, brand, price range, search (ILIKE), and status. Fixed 4 SQL linter bugs that Claude introduced (double WHERE / AND WHERE issues). 5 new router filter tests. All 207 tests passing.

---

## Testing and code quality

### Integration tests (5 prompts)

Implemented 33 brand & category repository tests (17 brand, 16 category). All 144 tests passing at this point.

### Test refactoring (6 prompts)

Extracted test helpers to the shared module: auth fixtures (`test_auth_config`, `test_token`, `seller_user`/`buyer_user`/`admin_user`), HTTP request builders (`json_request`, `authed_json_request`, `authed_get`, `authed_delete`). Removed redundant pagination integration tests and duplicated constructor functions from catalog router tests. Net result: -520 lines removed, +337 added. Compacted shared/CLAUDE.md from 187 to 82 lines.

> **My take:** LLMs seem to always default to not preemptively making abstractions and are prone to repeating code. So between the Product Filters and Test Refactoring sections, I spent some time looking over the code Claude generated and identified points for improvement. I'm sure I could spend more time on this, but I'd like to get to the complicated parts and see how my architecture decisions and LLM management skills hold up.

---

## Managing LLM context

### CLAUDE.md optimization & skill creation (7 prompts)

Asked Claude to suggest ways to improve the CLAUDE.md files as "onboarding docs" for new LLM sessions. Created a `project-context` skill that handles session onboarding (gathering context efficiently) and documentation maintenance (updating CLAUDE.md files after significant work). Pruned redundant info from root CLAUDE.md. I think this is the most useful skill I've created so far.

### Context optimization (5 prompts)

Explored how to minimize the context that gets loaded at the start of every session. Moved code patterns and bootstrap recipe to on-demand `.plan/` files instead of always-loaded CLAUDE.md. Cut always-loaded context by roughly 50%. Discussed knowledge graph tools vs markdown for managing project knowledge — decided to stay with markdown. Added deduplication rules so MEMORY.md and CLAUDE.md don't drift apart with redundant info.

### CLAUDE.md compaction + ADR-009 (5 prompts)

Compacted catalog CLAUDE.md from 250 to 88 lines, identity CLAUDE.md from 125 to 68 lines. Created ADR-009 for the ltree categories decision.

### Housekeeping (8 prompts)

Created `v0.2-catalog-crud` git tag and pushed to remote. Wrote a `make run SERVICE=<name>` command with a `run.sh` script, debugged it for both services. Small things, but the kind that save time every day.

> **My take:** I frequently checked the root `.claude` folder and kept seeing things pile up, which concerned me. Context management has become a first-class part of the workflow — every session now follows: `/project-context` → `/br` → work → flush to memory → end session.

---

## Planning the next phase

### Order/payment mega-planning session (4 prompts)

The big one. Created 4 detailed implementation plans:

1. **Shared infrastructure** — Kafka (KRaft mode, no Zookeeper), transactional outbox pattern, event system with rdkafka, Jaeger for distributed tracing, Redis-only service bootstrap function
2. **Cart service** — Redis-only microservice, 6 endpoints, 30-day TTL, max 50 SKUs per cart
3. **Order + Payment services** — choreography saga with state machines, mock payment gateway (Stripe/PayPal), inventory reservation, compensation flows, ~190 tests planned
4. **Workflow documentation** — ADRs 010-013, CLAUDE.md updates, saga flow documentation

> **My take:** I'm going to spend some time reviewing, researching independently, and iterating on the plans with Claude. Infrastructure (specifically Kafka) and saga orchestration are areas where I'm weakest in terms of experience and knowledge, so I'll have to tread carefully. I personally hate microservices, but this is what I specifically planned for — to push both myself and the LLM.

---

## Observations

> **My take:** These are actually Claude's observations. They're pretty accurate.

### Scope creep needs a human check

Claude suggested building automatic FK traversal for domain objects. Sounds nice in theory but it's basically reimplementing an ORM — massive scope increase for questionable benefit. Pushing back on LLM suggestions is a skill that matters.

### LLMs don't refactor proactively

Test infrastructure and shared helpers only got cleaned up because I reviewed the code and pointed out duplication. As the project grows, I don't want test times ballooning from duplicated setup code and tests that verify the same generic pagination behavior repeatedly.

### The mega-planning session validates my pt2 predictions

I predicted that orders, payments, and cross-service coordination would be where complexity explodes. The fact that I needed 4 separate plans just to approach this phase — before writing a single line of code — confirms that. The planning covered saga patterns, state machines, transactional outboxes, compensation flows, and distributed tracing. This is a fundamentally different challenge from "implement CRUD endpoints."

---

## Why I started programming

I started programming because at my first job, I spent hours on Google Sheets copy-pasting meaningless bullshit. After an agonizing amount of time doing that, developing unwanted muscle memory and getting tired of the farce\*, I decided to look for solutions to this Sisyphean task. The answer was programming with — ugh — JavaScript in an online scripting environment integrated into Google Workspace. It was a terrible dev experience. It had none of the convenience features that modern IDEs provide. All I had were my unending fury, hatred of manual work, googling skills, and bottomless willpower. This was around 2020-2021, for context.

When I got my custom macros to work and automated the entire boring task, it was an amazing feeling. The act of programming itself was addicting, and I continued to do it until it became my full-time job.

That's why when I got my jobs at startups, I enthusiastically threw myself into coding-heavy repetitive tasks (mostly refactoring and test writing without much of the fun "intellectual" domain object stuff). This helped me develop taste and opinions, but it also tired me out physically and mentally. I realized that programming, like any job, requires manual, boring, repetitive work.

Now that LLMs are here to automate the act of writing code itself, I don't know if I'll enjoy programming like I used to. Right now, it's so comfortable to get the LLM to do things for me, and to be honest, I'm not sure sometimes what the point of writing code faster even is. Right now, the shiny new toy is very interesting and I want this blog to be a showcase of my skills so that I don't need to do silly leetcode-style tests. That also makes me think: is being a good employee my true end goal? My thoughts are complicated and I need to think about it more. Nevertheless, I am still excited about the project.

\* Looking back now I understand why work was done that way(making interns do spreadsheet manual labour) but I still hate it with a great passion

## What's next

The 4 plans are created and waiting for review. Next is implementing them in order — shared infrastructure first (Kafka, outbox, tracing), then cart, then order+payment. This is where the real test begins.
