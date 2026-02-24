---
layout: post
title: "Getting Gud at LLMs Pt2"
date: 2026-02-24
categories: blog
---

In [Part 1](/blog/2026/02/23/getting-gud-at-llms-pt1.html), I built the identity service for [Koupang](https://github.com/JYC11/koupang/tree/main) and was surprised at how well Claude handled Rust and niche crates. Now I'm tackling the catalog service and experimenting with a task management workflow.

---

## Using a task manager

I figured out how to get Claude to use beads_rust. The key insight was splitting planning and execution into separate sessions so that the execution session starts with a clean context and only has the task list to work from.

The workflow:

1. Load in the beads_rust skill at the start of the session
2. Start plan mode
3. Give requirements
4. Iterate plan
5. When Claude asks to go ahead with the plan, I "reject" the execution and get it to put the tasks with dependencies into beads_rust
6. Close the planning Claude session
7. Start a new Claude session and load in beads_rust skill
8. Tell Claude to use beads_rust to look at what it needs to do and to execute it

The reason I close the planning session and start fresh is to prevent context bloat. The planning conversation can get long, and I don't want all of that history polluting the execution phase. By writing the plan into beads_rust, the new session can pick up exactly what it needs to do without carrying the baggage of the planning discussion.

---

## Results and Thoughts

### First impressions

- It followed the beads it created well
  - To be fair, it also kept notes in its internal MEMORY.md file about the next task (catalog service) and which bead to use
  - I should consider clearing the memory and then trying a new task that it put in beads to see how well it performs without that crutch
- It does simple CRUD very well so I was not too surprised it did the simple CRUD stuff perfectly
- It unfortunately just queries the entire db for list endpoints instead of using pagination despite there being common pagination support modules in the context (maybe it got lost?)
- It properly used value objects (I added stuff about value objects to the identity service) without me having to mention it
- It appropriately suggested claims based authentication. When I pushed back on using the gRPC server and a circuit breaker pattern, it suggested that those can be enhancements for later
- It didn't do router tests and only implemented them when I told it to
- It just implemented dynamic updating based on optional parameters in the product update request dto without me telling it to
- Catalog service is currently very CRUD-y which makes sense considering there isn't much "business logic" yet so that's fine for now
- This saved me a TON of time typing
  - I would say 2-3 days of repetitive typing and debugging got reduced to 2-ish hours of planning and waiting for the LLM to generate code

### Deeper reflections (after stepping away for lunch)

- It properly planned out + implemented slugs for products, sku code, non-zero skus and other such domain specific requirements correctly without me explicitly saying so
- Something I forgot was dealing with locks in databases. Something I used to do a lot was doing very conservative pessimistic locking for records before updating for strong consistency requirements. It didn't really suggest doing something like that but I should have at least considered it somehow.
- The get product detail was done with 3 separate queries (1 for product, 1 for skus, 1 for images). I personally would have used a join and some json aggregation to do one query but this is acceptable. I found that many people have many different opinions about the best way to interface with a database from the application code side.
- Not using an ORM (although I think they are a fantastic tool **if** used well with intention) I think was a good choice. I shied away from raw SQL before LLMs due to the finicky nature of raw string manipulation, having to map between code and sql result sets by casting `Any` types to concrete types, and syncing between code and sql being annoying. But LLMs make these kinds of things very trivial to do and they are quite decent at SQL and they do not have to learn some potentially niche ORM library without many examples.
- The LLM still handles the current size of the codebase well.

---

## Predictions

- I am building simple foundations for more complex features so things are going well so far
- I expect dealing with orders, shipping, payment, refunds (things surrounding products) to massively increase complexity. These features require cross-service coordination, state machines (order lifecycle, payment states), and careful handling of failure scenarios (partial refunds, failed shipments). I would like to see how I can use LLMs to handle them well but I expect the LLM to struggle here.
- Certain features more directly related to the catalogs like dynamic pricing based on algorithms and discount features, searching for products, handling high traffic while keeping track of stock could also challenge me and LLMs. These involve algorithmic thinking and concurrency concerns that go beyond CRUD patterns, so I expect the LLM to struggle in this regard as well.
- My Claude Code install is fresh so there isn't much cruft in the various config and memory files on my computer (I checked). As this grows, I expect Claude to be a bit more confused even if I keep the context fresh each session. Stale or accumulated memories from past sessions could mislead future ones, so I think I would have to clear those regularly.

---

## Thoughts about backend development

A lot of backend development is just plumbing data in my humble opinion. You receive some kinda data through the network, you shove it into some kinda persistence layer, you retrieve something from the persistence layer, you throw it out into the network, repeat ad infinitum. There are lots of established patterns that I just need to re-implement again and again.

"Difficulty" in backend development came from:

1. Not knowing programming well
2. Not understanding how to use the libraries/packages
3. Writing bad code and suffering from it
4. Working with code that others have wrote
5. Crazy deadlines and fluctuating requirements

But as time passed:

1. Programming by hand solved the not knowing programming well as I developed intuition and muscle memory about the language
2. Reading documentation and looking up guides solved the library/package issue
3. Writing bad code got solved when I was forced to refactor my own code to make it testable and I developed intuition on how to write simpler more testable code
4. Working with code written by other people is still difficult
5. I can't do anything about deadlines and fluctuating requirements — but faster coding helps with both

### How LLMs change this

- LLMs solve issue 1 and 2 (syntax and library/package issues). They have tons of training data and can look things up online.
- LLMs don't really solve issue 3 (bad code) definitively. It really depends on what you feed it.
- LLMs _could_ solve issue 4 (working with code written by other people) but I haven't used LLMs in a context where I'm completely new to the codebase and there is no one to onboard me.
- LLMs don't solve issues about deadlines and requirements directly, but they make working with fluctuating requirements easier because writing and rewriting code is much faster.

### Skill atrophy and the next generation

I have experience writing bad code, improving and unraveling my cocoon of ignorance on various programming topics which arguably lets me be effective in structuring code, providing samples, fixing code produced by AI. But I wonder if these skills would atrophy as I use LLMs more. I also wonder if the newer generation of software engineers will develop different kinds of intuition.

### Problems I haven't faced yet

There are also problems I haven't faced a lot yet such as dealing with extremely high traffic, working with low hardware resource constraints, maintaining very high uptime, dealing with distributed systems and such. I never got to develop experience and intuition doing this "the old way" from working at smaller companies so I wonder how I will develop as an engineer when I will hopefully get to tackle these kinds of issues with an LLM in the future.

---

## Prompt Log: Planning the Catalog Service

One thing I wanted to do with these posts was show the actual interaction flow, not just the results. Here's the exact sequence of prompts I used to plan the catalog microservice with Claude Code.
This section was done with help from Claude parsing the jsonl files and I have to say it did it VERY well. I expected mild prompt injection as it was reading about previous prompts but it didn't?? I also automated this step with Claude suggesting I put reminders in the Claude.md file and added a hook with a script Claude created. I am quite impressed.

### 1. Check task board state

```
/br
```

Checks current beads_rust task board. Confirms starting from a clean slate.

### 2. Clean up stale memory

```
remove plan 5 from your memory wherever that is because it has been partially implemented
```

Housekeeping — removes outdated progress entries from auto-memory before starting new work.

### 3. Enter plan mode with detailed requirements

```
/plan
I want to start working on the catalog microservice now
The catalog microservice will allow buyers and admins to upload and manage products
The 2 main important tables will be Product and Sku. Sku is a child table of Product.
1 Product can have many Skus. A Sku is a variation of a Product
Eg: A product can be a shoe and a sku can be the shoe sizes
The granularity of the sku can be something we discuss
The catalog service will also need to keep track of the inventory levels as well
For the initial iteration of the catalog service, we will assume that image files are
handled somehow and we receive links to images. Detailed image/media handling can be
implemented later
We also need to keep track of prices here. When dealing with money, we need to make sure
to use the correct types because floating point numbers do not reflect money behaviour
accurately.
Do the plan first and then I will take a look
```

Enters plan mode. Provides high-level requirements with open questions (SKU granularity). Explicitly says "do the plan first" to let Claude explore and design before execution.

### 4. Answer design questions

Claude asked 3 targeted questions:

- **SKU variant attributes model?** → Selected: "Flexible JSON attributes (Recommended)"
- **Price and inventory on SKU level?** → Selected: "Yes, both on SKU (Recommended)"
- **Who can create/manage products?** → Selected: "Sellers and Admins (Recommended)"

### 5. Challenge a design decision

```
can you explain the claims based auth decision?
```

Rejected the initial plan approval to ask about a specific architectural choice. Claude explains the claims-based vs gRPC auth trade-off. This is a key technique — rejecting plan approval doesn't lose work, it just lets you dig deeper.

### 6. Propose alternative with nuance

```
I prefer the gRPC to identity but the point about coupling is correct. Instead, I think
adding a periodic health check to the identity grpc to determine whether to call the gRPC
service would be better and then gracefully fail to claims based. Also, caching on the
catalog service side for the gRPC identity service could work. What do you think about
these 2 suggestions?
```

Pushes back with a hybrid approach. Claude analyzes both suggestions and recommends phasing the work.

### 7. Agree on phasing

Claude asked whether to phase the work or include everything now. I selected "Phase it (Recommended)".

### 8. Log tasks in beads_rust

```
I want you to use beads_rust to log the plan for future use
```

Before approving execution, asks Claude to create br tasks with dependencies so the plan is tracked in the task management system. This is the step that enables the "close session and start fresh" workflow described above.

### 9. Document the workflow

```
do not execute yet, I want to restart the session. I also want a workflow of user input
prompts into claude cli for blogging purposes to demonstrate claude cli tool usage. Can you
log all of my user prompts and then create a repeatable workflow for future use in all
other sessions?
```

This is where I stopped the planning session and asked Claude to document everything before restarting for execution.

### Key techniques demonstrated

- `/plan` mode separates research from execution
- Rejecting plan approval to ask questions (doesn't lose work)
- Using `br` (beads_rust) for persistent task tracking across sessions
- Phased delivery: start simple, enhance later
- Pushing back on design decisions with your own suggestions

---

## Prompt Log: All Sessions So Far

I extracted the user prompts from all 26 Claude Code sessions across both the identity and catalog services. Below is the condensed version — system noise stripped out, plans summarized instead of pasted in full. This covers about 2 days of work.

### Identity Service: Integration Tests (Sessions 1-5)

```
/plan
your task now is to plan implementing integration tests for the identity microservice.
The integration tests should use the #[sqlx::test] macro for actual database usage for
all levels of tests which are to be implemented (repository_test, service_test,
router_test). Plan test cases for me to review as well
```

```
Implement the following plan: [full plan with 39 test cases across 3 layers,
plus 2 bug fixes found during planning]
```

```
I want you to run integration tests for the identity service using the Makefile command.
There will be a test failure for get_current_user_returns_correct_user this test. I want
you to identify the cause, explain it then fix it
```

```
can you fix the other issues found by the other test failures?
```

```
your task is to refactor the GetCurrentUser trait to make it async and fix the
implementation on the identity service side. Use the identity service integration test
to verify the fix works
```

```
to make myself clear, I meant make it async fn get_by_id, the current implementation
"technically" is but I want to use the async syntax for the trait
```

### Claude Code Configuration (Session 6)

```
I want help configuring claude code cli. I want to allow safe commands like ls, grep,
cat, etc bash commands for reading to be allowed while I want commands like rm, curl
(potentially accessing malicious links), etc to require permission from me
```

### Shared Module Extraction (Sessions 7-8)

```
/plan
based on the high level description and current implementation of the identity service,
I want to identify some common code/utility things that can be put in the shared module.
Do some planning for me to review and then after planning, use beads (br skill) to put
those as tasks with proper dependencies
```

```
Implement the following plan: [extract 6 modules to shared: observability, server
bootstrap, API responses, auth guards, health check, DTO helpers]
```

```
cool, now update CLAUDE.md file in the root folder to record for future use what can
be reused from the shared module in a compact manner
```

```
there are other pieces of code in the shared module that aren't mentioned in the
CLAUDE.md file, put those in as well
```

### Auth Flows (Sessions 9-14)

```
/plan
read the .plans/critical-user-flows.md 8 Auth Flows and do some planning 1 at a time.
I will review each plan 1 by 1 and then you will add them to beads
```

```
Implement the following plan: [Plan #1: Email Interface — trait + mock]
```

```
let's move to phase 2 and remember when running tests, refer to the makefile for the
test running commands
```

```
Implement the following plan: [Plan #2: Email Verification on Registration]
```

```
when making migrations, use the make migration command (refer to the makefile for
details) to create the migration file, continue with phase 2
```

```
let's do phase 3 now
```

```
Implement the following plan: [Plan #3: Password Reset Flow]
```

```
go ahead with the plan for phase 3
```

```
now let's do phase 4
```

```
/br
create br issues for remaining plan#5 AFTER you make the plan and let me approve
```

```
Implement the following plan: [Plan #5: gRPC + Redis Caching]
```

```
the grpc_service currently has a build error, the generated type and the implementation
seems to be not matching despite it using the same type
```

```
now I need you to abstract away the redis connecting and grpc server bootstrapping to
the shared module as I can see it being used commonly in many services
```

```
Implement the following plan: [Abstract Redis + gRPC bootstrapping into shared]
```

### Testing Infrastructure (Sessions 15-18)

```
I need you to implement integration tests for the grpc service in the identity service
user package
```

```
is there a way to actually run the grpc server and then call from a grpc client for
the test?
```

```
/plan
the current GetCurrentUser implementation and test gracefully handles the non-presence
of an actual redis client, however I want to use an actual redis client for the test
for more comprehensive integration testing. Explore options on how to do this so that
this setup can be abstracted away and reused in many cases
```

```
Implement the following plan: [Real Redis Integration Tests via Testcontainers]
```

```
there are several test utils like starting a grpc server and starting a redis
testcontainers instance that can be put in shared for common use across all
microservices, do so
```

```
Implement the following plan: [Extract Reusable Test Utilities to Shared]
```

```
now, while the sqlx-test macro is convenient to use it requires a running db instance
to run tests. I don't want a dependency on a running db managed outside of the testcode
in case these tests runs in CI. Now that we established that redis testcontainers work,
refactor the tests to use postgres testcontainers and remove the use of sqlx-test macro
and put the postgres testcontainers setup in the test utils and make it be used in the
integration tests for identity
```

```
Implement the following plan: [Replace #[sqlx::test] with Postgres Testcontainers — 82 tests]
```

### Refinements (Sessions 19-22)

```
while we are in the early stages of the project, I want to set stricter roles using
enums. The roles should be Seller, Buyer and Admin. Make changes to the identity and
shared modules referring to role which is a String right now
```

```
Implement the following plan: [Refactor Role from String to Enum]
```

```
the claude.md file is getting quite large, I can see that the information about the
shared module can be split up and put into a new claude.md file. Put the compact overview
of the shared module in the shared crate and update the shared module Claude.md so that
it reflects the most current code in the shared module. Point towards the shared module
Claude.md in the Claude.md in the root folder.
```

```
create a CLAUDE.md for the identity service module and make the root folder CLAUDE.md
reference it. Make sure the identity service module CLAUDE.md is compact
```

```
I want you to look at the code so far (mainly identity, shared modules) and the various
scripts and other stuff created to create a summary. I wrote a blog post about using AI
and I want to add a summary about what was built so far.
```

```
let's put the summary of what is there so far in a .md file, I will copy paste to the
blog at a later point. Now let's do the ADR and git tags as you have suggested.
```

### Value Objects & Validation (Sessions 23-24)

```
/plan
in the identity microservice, there needs to be validation for username, password,
email and phone strings
email -> research well known email regexes
password -> research "strong" password regexes
phone -> assume that we store country code with phone numbers, allow - characters
username -> no empty strings and minimum 3 characters, no profanities (nice to have)
make value objects, parse the strings into valid value objects where the new function
in the struct impl validates with regex
write unit tests for these value objects
make sure when writing into the db that the strings are valid
create a new validated struct ValidUserReq that uses the value objects and replace the
use of UserCreateReq, UserUpdateReq
also update the password flows to use password value objects
```

```
Implement the following plan: [Value Objects & Input Validation — 4 value objects, ~35 unit tests]
```

### Catalog Service Execution (Sessions 25-26)

```
/br
start work on bd-jx9 and close issues as you go
```

```
make a task for router tests for products in br, implement the product router tests,
close the br issue
```

```
add following tasks to br
implementing pagination for listing product endpoints (there are several which just
return the entire list)
implementing caching for read endpoints
planning search engine implementation
```

```
to specify, I meant keyset pagination. update the bead about pagination
```

### Claude's Observations from the logs

- **76 total user prompts across 26 sessions** over roughly 2 days
- Most prompts are short (1-3 sentences). The longest ones are the plan pastes and initial requirements.
- The pattern is very consistent: `/plan` → review → paste plan into new session → implement → follow-up corrections
- Corrections tend to be brief and specific ("I meant keyset pagination", "make it async fn")
- I spent more time on testing infrastructure than I expected — sessions 15-18 are entirely about making tests self-contained with testcontainers

---

## Git Tag

- `v0.2-catalog-crud`
- This is the new tag that has been create for the latest amount of code that got generated.

---

## What's next

In Part 3, I want to push into the more complex services — starting with orders — and see if my predictions about LLM struggles hold up.
