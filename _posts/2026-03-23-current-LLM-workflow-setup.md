---
layout: post
title: "Current LLM Workflow Setup"
date: 2026-03-23
categories: blog
---

An updated snapshot of my LLM-assisted development setup, as of March 2026. The [previous snapshot](/blog/2026/02/26/current-LLM-workflow-setup.html) was from late February. A lot has changed — new tools, more skills, a proper planning pipeline, and significantly expanded permissions. Same deal as before: Claude examined its own configuration and I peppered in my commentary.

---

## Environment

|                     |                                                                    |
| ------------------- | ------------------------------------------------------------------ |
| **Terminal**        | Ghostty                                                            |
| **IDE**             | IntelliJ (Rust development), Zed (blogging)                        |
| **LLM Tool**        | Claude Code CLI (Claude Opus, 1M context)                          |
| **Task Management** | [Filament](https://github.com/JYC11/filament) (replaced beads_rust) |
| **Code Generation** | [Jujo](https://github.com/JYC11/jujo)                             |

> **What changed:** Upgraded from beads_rust to filament for task tracking. Filament adds a knowledge graph, inter-agent messaging, and a TUI — all in one Rust binary. Added jujo for deterministic code generation from templates. The 1M context window is a game changer — no more context exhaustion mid-session.

---

## Claude Code Configuration

### Plugins

| Plugin                | Purpose                                                                                                      |
| --------------------- | ------------------------------------------------------------------------------------------------------------ |
| **rust-skills**       | Rust-specific guidance — ownership, concurrency, error handling, domain patterns, crate research, daily news |
| **rust-analyzer-lsp** | LSP integration for go-to-definition, find references, symbol analysis                                       |
| **code-review**       | Structured PR code review                                                                                    |

> **What changed:** Added the code-review plugin since February.

### Skills (20 installed)

Custom skills loaded from `~/.claude/skills/`:

| Skill                  | Source | What it does                                                                              |
| ---------------------- | ------ | ----------------------------------------------------------------------------------------- |
| **project-context**    | local  | Reads CLAUDE.md files for onboarding; updates them after changes                          |
| **skill-creator**      | local  | Guide for writing effective Claude Code skills                                             |
| **filament**           | local  | Task lifecycle, knowledge graph, lesson capture via `fl` CLI                              |
| **jujo**               | local  | Code generation from Tera templates via `jujo` CLI                                        |
| **pattern-analyzer**   | local  | Analyze codebase patterns → generate jujo templates                                       |
| **research**           | local  | GitHub repo exploration and web article fetching via Go CLI                                |
| **handoff**            | local  | Structured session handoff summaries                                                      |
| **cleanup**            | local  | Scan and remove stale files across /tmp, ~/.claude, project dirs                          |
| **datastar**           | local  | Datastar hypermedia framework patterns                                                    |
| **spec-driven-dev**    | local  | Three-phase workflow: Research → Plan → Implement with human checkpoints                  |
| **grill-me**           | matt   | Interview user relentlessly about a plan until shared understanding                       |
| **write-a-prd**        | matt   | Create PRD through user interview, submit as GitHub issue                                 |
| **prd-to-plan**        | matt   | Turn PRD into multi-phase implementation plan with tracer bullets                         |
| **prd-to-issues**      | matt   | Break PRD into independently-grabbable GitHub issues                                      |
| **triage-issue**       | matt   | Triage bugs: search filament lessons → investigate → fix plan with TDD                   |
| **review**             | gstack | Pre-landing PR review against project checklist                                           |
| **code-eng-review**    | gstack | Eng manager-mode review of implemented code                                               |
| **plan-eng-review**    | gstack | Eng manager-mode plan review with architecture focus                                      |
| **retro**              | gstack | Engineering retrospective with trend tracking                                             |
| **library**            | fork   | Private skill distribution via YAML catalog + git sync                                    |

Sources: local = original, matt = [mattpocock/skills](https://github.com/mattpocock/skills), gstack = [garrytan/gstack](https://github.com/garrytan/gstack), fork = [disler/the-library](https://github.com/disler/the-library)

> **What changed:** Went from 4 skills to 20. The biggest additions are the planning pipeline (write-a-prd → prd-to-plan → prd-to-issues), the spec-driven-dev meta-workflow, and the library catalog for managing skills across devices. All skills now have filament integration for task tracking and lesson capture. Removed br and bd-to-br-migration skills.

### Hooks

| Event              | Hook                    | What it does                                                                              |
| ------------------ | ----------------------- | ----------------------------------------------------------------------------------------- |
| UserPromptSubmit   | `log-prompt.sh`         | Captures every user prompt to a daily session log file. Now works across multiple projects |
| PostToolUse (Bash) | cargo check after build | Runs `cargo check` after any cargo/make command to catch compile errors immediately       |

> **What changed:** The prompt logger now works across multiple projects (previously Koupang-only). Added the PostToolUse hook for Koupang that runs cargo check after build commands — catches compilation errors before I even look at the output.

### Permissions

The permissions list has grown significantly. The philosophy is: anything that only reads or only modifies local project files should be auto-allowed.

**Explicitly allowed (no confirmation needed):**

- **Read-only shell**: `ls`, `cat`, `find`, `grep`, `rg`, `tree`, `stat`, `which`, `file`, `wc`, `sort`, `uniq`, `diff`, `basename`, `dirname`, `realpath`, `jq`, `cut`, `tr`, `awk`, `sed`, `xargs`, `tee`
- **Git read**: `status`, `log`, `diff`, `branch`, `show`, `tag`, `remote`, `stash`, `blame`, `rev-parse`
- **Git write**: `add`, `commit`, `checkout`, `switch`, `merge`, `rebase`, `fetch`, `pull`, `push`, `cherry-pick`
- **Cargo**: `check`, `build`, `test`, `clippy`, `fmt`, `run`, `add`, `tree`, `doc`, `metadata`, `install`, `update`, `clean`, `bench`, `fix`
- **Docker**: `compose up/down/ps/logs/build/exec`, `images`, `logs`, `build`, `ps`
- **Make**: all `make` targets
- **Custom CLIs**: `fl` (filament), `jujo`, `sqlx`, `rustup`, `research` (Go CLI)
- **File ops**: `mkdir`, `touch`, `cp`, `mv`
- **Shell**: `printf`, `read`, `echo`, `whoami`, `id`, `env`, `date`, `uname`

**Explicitly denied:**

- `rm`, `sudo`, `curl`, `wget`, `chmod`, `chown`, `kill`, `killall`, `pkill`, `dd`, `mkfs`
- `git push --force`, `git reset --hard`, `git clean -f`
- `docker rm`, `docker rmi`
- `WebSearch`, `WebFetch`
- Output redirection (`> *`)

> **What changed:** Massively expanded from ~25 allowed commands to ~80+. Added all the text processing tools (jq, awk, sed, etc.), file operations (mkdir, cp, mv), more git commands (cherry-pick, blame), the full cargo suite, and custom CLIs (fl, jujo, research). The goal was to reduce the number of "approve this?" prompts to near-zero for normal development work. It mostly worked — I rarely see permission prompts now unless Claude is doing something genuinely unusual.

### CLAUDE.md Files

Same hierarchical structure as before, but now with more content:

- **Root** (`koupang/CLAUDE.md`) — workspace structure, tech stack, ADR summary, key imports, scripts
- **STYLE.md** — coding style guide adopted from TigerBeetle's TIGER_STYLE, customized for this project. Covers: data-oriented programming, value objects, assertions, error handling, naming, function size
- **Per-service** (`identity/CLAUDE.md`, `catalog/CLAUDE.md`, `shared/CLAUDE.md`, `order/CLAUDE.md`, `payment/CLAUDE.md`, `cart/CLAUDE.md`) — detailed architecture, endpoints, domain models, test structure
- **Reference docs** (`.plan/`) — detailed implementation plans, test standards

> **What changed:** Added STYLE.md which is now the single source of truth for coding conventions. Added CLAUDE.md files for order, payment, and cart services. The STYLE.md adoption was a turning point — it gives Claude a concrete reference for what "good code" looks like rather than relying on vibes.

---

## Development Cycle

### For new features (spec-driven-dev workflow)

1. **Research** — `/spec-driven-dev` triggers filament lesson search for prior knowledge, then codebase exploration
2. **Plan** — `/grill-me` to stress-test the design, then `/plan-eng-review` for architecture review
3. **Create tasks** — break plan into filament tasks with dependency chains
4. **Implement** — work through tasks with `fl task ready` to find unblocked work
5. **Review** — `/code-eng-review` for structured code review against STYLE.md
6. **Capture lessons** — `fl lesson add` for gotchas and patterns discovered

### For boilerplate/scaffolding

1. **Analyze patterns** — `/pattern-analyzer` to find repeated structures in the codebase (done once or when patterns need updates)
2. **Generate templates** — export to jujo generator with `jujo init` + template files
3. **Stamp out code** — `jujo generate` for deterministic scaffolding
4. **Customize** — Claude fills in AI customization markers for business logic

### For bug fixes

1. **Triage** — `/triage-issue` searches filament lessons first, then investigates
2. **Fix** — TDD approach with the fix
3. **Capture** — lesson recorded in filament for future reference

### What makes this work

- **STYLE.md** gives Claude a concrete reference for code quality, not vibes
- **Filament** provides persistent context across sessions — lessons, tasks, and knowledge graph survive session boundaries
- **Jujo** eliminates token waste on boilerplate — deterministic code gen for repetitive patterns, Claude only handles the unique parts
- **The planning pipeline** (PRD → plan → grill → review → implement) prevents wasted work on under-specified features
- **Expanded permissions** make the flow nearly frictionless — I rarely see "approve?" prompts
- **1M context window** means I can do planning + implementation + review in a single session without context exhaustion
- **The prompt logger** captures everything for blog posts and retrospectives

> **My take:** The February setup was functional but ad-hoc. The March setup feels like a proper workflow. The biggest wins were: (1) filament replacing beads with a knowledge graph that accumulates project wisdom across sessions, (2) STYLE.md giving Claude a codified standard to follow, and (3) the planning pipeline preventing the "just start coding" impulse that led to problems in the Filament sprint. I'm still not at the "fully autonomous multi-agent" level but I'm getting more comfortable delegating larger chunks of work to single sessions with good context. I also found that STYLE.md helped a lot in getting the agent to produce better code.

## Closing Thoughts
Since this is all setup locally and I will have to use work computers, I will need a handy way to package all this and export it to other computers. I'll get around to doing that.