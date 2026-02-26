---
layout: post
title: "Getting Gud at LLMs Pt4"
date: 2026-02-26
categories: blog
---

In [Part 3](/blog/2026/02/25/getting-gud-at-llms-pt3.html), I finished the catalog service, compacted CLAUDE.md files, and kicked off planning for the order/payment phase. Today was less about writing code and more about optimizing what exists and planning what comes next. I also wrote a separate post about my [current LLM workflow setup](/blog/2026/02/26/current-LLM-workflow-setup.html) if you're curious about the tooling side. I also write about the direction of blog posts since I got to thinking about what was the end goal.

---

## Direction of blog posts

The blog posts so far contain 3 topics

- using LLMs
- the Koupang project
- my existential dread + musings about software industry

### Git Gud

My LLM usage stabilized once I figured out planning, beads and context management. According to Steve Yegge's [Gas Town](https://steve-yegge.medium.com/welcome-to-gas-town-4f25ee16dd04), I am somewhere around level 5. Personally, I haven't reached the stage in the codebase where multiple agents are justifiably needed. But when I have multiple microservices developed and have many features to develop at once (I started putting more tasks as backlogs besides this order saga to fill things up), I will start experimenting with multiple agent stuff. I am still a big proponent of human in the loop and can still see areas where the LLM messes up so I need to challenge that part as well. I want to see if I can create nearly comprehensive guidelines that many LLMs can follow so that I can grow to "trust" their output. Trust is a funny word considering LLMs are just fancy matrix multiplication calculators.

When I reach maybe level 7/8 (or whatever the highest possible level I can reach) from that Gas Town post, I think that will be a nice place to end this series.

### Koupang

On the topic of the project itself, I decided that the MVP will be when the full order cycle is completed and is in a deployable state. Once that is done, the functional and non-functional challenges will become more difficult.

- On the functional side, I will try to leverage multi-agents to develop many features at once.
- On the non-functional side, I will also try to leverage multi-agents to make this service ready for "production" grade traffic and up-time.

These updates will be a bit less frequent and will mostly focus on non-functional stuff. The "feature" work can be outsourced to LLMs since a lot of it is mostly CRUD, but the "tech" side I want to take a bit more time to make it right. I think the posts will be titled "Overengineering Koupang for Fun and Profit pt{n}".

### Trauma dumping on main

Honestly have no clue about this. May suddenly decide to randomly drop an 10k word essay or whatever.

---

## AI summary of work

Same deal as pt3 — I got Claude to summarize the work from its own session logs and git history. My commentary is in block quotes.

### By the numbers

5 sessions, ~34 prompts across Feb 25 evening and Feb 26:

- 4 git commits since `v0.3-catalog-complete`
- 19 files changed, +879 lines added, -2,523 lines removed (net -1,644 lines)
- Tests: 207 → 235 (deduped many redundant tests, added shared module tests)
- 4 plan files revised with inline comments
- 35 beads tasks created with full dependency DAG
- 1 new doc: `.plan/test-standards.md`

Git tag: `v0.4-test-optimization-and-planning`

Most of the work fell into two categories: test optimization and order/payment planning.

---

### Test optimization (2 sessions, ~13 prompts)

Analyzed the test suites across identity and catalog and found significant redundancy. The same CRUD operations were being tested at every layer (router, service, repository) with full end-to-end Postgres containers each time.

Key changes:

- **Shared container infrastructure** — instead of spinning up a separate Postgres/Redis container per test, containers are now shared per test binary. This cut test setup overhead significantly.
- **Test deduplication** — removed 107 redundant tests across identity and catalog (identity 115 → 82, catalog 209 → 135). Coverage maintained — the removed tests were asserting the same behavior at multiple layers.
- **Extracted test helpers to shared** — auth fixtures (`seller_user()`, `admin_user()`), HTTP request builders (`authed_json_request`, `authed_get`), and pagination unit tests all moved to the shared module so every future service gets them for free.
- **Test standards doc** — created `.plan/test-standards.md` defining what each test layer should cover, preventing redundancy from creeping back in.

Net result: -2,523 lines of test code removed, +879 added (mostly shared infrastructure), and the test suite runs faster.

> **My take:** This was necessary to do and quite quickly achieved. This would have taken me ages to type otherwise so I'm glad I got the LLM to do it. Emphasizes the importance of actually reading what the LLM generates. I made the AI create a `test-standards.md` file so that it can refer back to these standards repeatedly to prevent further code like this.

---

### Order/payment mega-planning (3 sessions, ~21 prompts)

This was the big one. The order/payment phase touches multiple services and needs careful coordination. The planning was split across multiple sessions:

**Session 1 — Initial 4-plan structure (from pt3)**
Claude explored the entire codebase and created 4 detailed plan files:

1. **Shared infrastructure** — Kafka KRaft, transactional outbox (`outbox-core`), event system with `rdkafka`, distributed tracing with Jaeger
2. **Cart service** — Redis-only, 6 endpoints, 30-day TTL
3. **Order + Payment** — choreography saga, state machines, mock payment gateway, inventory reservation, compensation flows
4. **Workflow docs** — ADRs 010-013, CLAUDE.md files, saga flow documentation

**Session 2 — Plan review with my comments**
I left inline comments on plans 1-3 (titled "Comment on [relevant part]"), then walked through each one with Claude to revise:

- Plan 1: Added `ServiceBuilder` pattern, typed event enums, DLQ topics, programmatic Kafka topic creation
- Plan 2: Changed cart to display-only totals, added `/validate` endpoint, seller order endpoint
- Plan 3: Added `PaymentTimedOut` handling, `sku_availability` view

**Session 3 — Double-entry accounting discussion**
I pushed for a double-entry accounting ledger in the payment service, inspired by [Alvaro Duran's article](https://news.alvaroduran.com/p/engineers-do-not-get-to-make-startup) about big tech companies begrudgingly building their own double-entry payment ledgers. Claude revised plan 3 to adopt this approach and added a note about platform commission (out of scope for now but will affect the ledger design).

After the plans were finalized, Claude created 35 beads tasks with a full dependency DAG across all 4 plans, plus 3 MVP milestone tasks (docker-compose deployment, seed script, API walkthrough) and a standalone non-functional requirements task for high traffic/uptime planning.

> **My take:** Still very planning heavy and I could have decided to research way more before going through amendments but I think I need to execute and learn through pain and suffering. The accounting double entry stuff was my call because I know from experience that money records needs a special kind of care/domain knowledge. I predict the Kafka stuff will bite me in the ass but oh well.

## What's next

The 4 plans are reviewed and waiting for implementation. Next time, I'll try to push my current setup to its limits by handling this huge set of requirements with me managing 1 agent.

Here's the full beads dependency tree — this is what Claude will work through:
(I got Claude to use the beads_rust skill to read from beads_rust and format this. This is so very convenient lmao)

```
PLAN 1: SHARED INFRASTRUCTURE (foundation for everything)
──────────────────────────────────────────────────────────
bd-na8  Event types + typed enums (EventType, AggregateType, EventEnvelope)
├── bd-1j2  KafkaEventPublisher (rdkafka) ← also needs bd-3ga
│   ├── bd-3sv  KafkaEventConsumer with DLQ support
│   │   ├──→ [Plan 3: Order schema, Payment schema, Catalog inventory]
│   │   └── bd-2v4  ADR-010 Event-driven architecture ← also needs bd-1ee, bd-5y4
│   └── bd-27z  Kafka health check
├── bd-5y4  ServiceBuilder composable bootstrap
│   ├──→ [Plan 2: Cart value objects]
│   └──→ bd-2v4 (see above)
└── bd-1fo  MockEventPublisher (test infra)

bd-3ga  Docker compose additions (Kafka KRaft, Kafka UI, Jaeger)
├──→ bd-1j2 (see above)
└── bd-2g4  Programmatic topic creation (AdminClient)

bd-3ej  Research outbox-core crate API compatibility
└── bd-1ee  Outbox integration via outbox-core + migration templates
    ├──→ [Plan 3: Order schema, Payment schema]
    └──→ bd-2v4 (see above)

bd-8fc  Distributed tracing OTLP exporter (independent)


PLAN 2: CART SERVICE (Redis-only)
─────────────────────────────────
bd-337  Cart value objects (Quantity, PriceSnapshot, Currency) ← blocked by bd-5y4
└── bd-2aq  Cart domain model (CartItem, Cart) + Redis data model
    ├── bd-1tw  Cart DTOs (request, validated, response) ──┐
    └── bd-3sf  Cart repository (Redis ops) + tests ───────┤
                                                           ▼
                                              bd-1ra  Cart service + tests
                                              └── bd-aqs  Cart routes + router tests
                                                  └── bd-1p7  Cart bootstrap (lib.rs, main.rs)
                                                      └── bd-305  cart/CLAUDE.md


PLAN 3: ORDER + PAYMENT + INVENTORY
────────────────────────────────────
Order chain:                              Payment chain:
bd-32f  Order schema ← bd-1ee, bd-3sv    bd-1tp  Payment double-entry schema ← bd-1ee, bd-3sv
└── bd-lwv  Order value objects           ├── bd-2cj  Payment gateway trait + mock
    └── bd-186  Order repository          └── bd-1te  Payment repository
        └── bd-2ac  Order service             └── bd-1rk  Payment service ← needs both
            └── bd-3pu  Order routes              └── bd-3of  Payment routes
                └── bd-a1p  Order outbox
                    └── bd-1a3  Order Kafka consumers

Inventory chain:
bd-1yx  Catalog inventory migration ← bd-3sv
└── bd-xsn  Inventory service + repository
    └── bd-2h3  Inventory Kafka consumer

              ┌─── bd-1a3 (order) ──────┐
All three ──► │    bd-3of (payment) ────►├──► bd-b02  Wire Kafka consumers in all main.rs
              └─── bd-2h3 (inventory) ──┘
                                         └── bd-1zd  Saga integration tests
                                             ├── bd-jp3  order/payment CLAUDE.md
                                             └──► MVP track (below)


PLAN 4: DOCS
─────────────
bd-jp3  order/CLAUDE.md + payment/CLAUDE.md ──┐
bd-305  cart/CLAUDE.md ────────────────────────┤
                                              ▼
                              bd-1jp  ADRs 010-014
                              bd-m7m  Saga flow docs ← bd-jp3
                              └── bd-2ne  Progress summary pt3


MVP MILESTONES
──────────────
bd-1zd  Saga integration tests
└── bd-32d  MVP: Docker Compose (all services + infra)
    └── bd-o7r  MVP: Seed data script
        └── bd-9mm  MVP: API walkthrough / Postman collection


BACKLOG (independent, no blockers)
──────────────────────────────────
P2: bd-2yx  Redis caching for product reads → bd-2sh Search engine planning
P3: bd-1yk  Bulk product/SKU CSV processing
P3: bd-3jn  Brands list keyset pagination
P3: bd-1dh  Image upload for products/SKUs
P3: bd-v0a  Plan for high traffic and uptime (NFRs)
P3: bd-dsh  Resilient auth (gRPC + Redis cache + circuit breaker)
P3: bd-2kq  Evaluate repository trait pattern for mockable tests
P4: bd-x38  Advertisements service planning
P4: bd-7m8  Discounts/coupons planning
P4: bd-dj9  Evolve domain FK refs to embedded domain objects
```
