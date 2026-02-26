---
layout: post
title: "Current LLM Workflow Setup"
date: 2026-02-26
categories: blog
---

A snapshot of my current setup for LLM-assisted development, as of February 2026. This is what I use to build [Koupang](https://github.com/JYC11/koupang), the Rust e-commerce project documented in the [Getting Gud at LLMs](/blog/2026/02/23/getting-gud-at-llms-pt1.html) series.

I got Claude to look at its own configuration, write the summary of configuration and format the post nicely. Like pt3, I peppered in my commentary in block quotes. I thought this could be a separate post on its own as this is quite meta and mostly informational.

---

## Environment

|                     |                                                                    |
| ------------------- | ------------------------------------------------------------------ |
| **Terminal**        | Ghostty                                                            |
| **IDE**             | IntelliJ (Rust development), Zed (blogging)                        |
| **LLM Tool**        | Claude Code CLI (Claude Opus)                                      |
| **Task Management** | [beads_rust (br)](https://github.com/Dicklesworthstone/beads_rust) |

---

## Claude Code Configuration

### Plugins

| Plugin                | Purpose                                                                                                      |
| --------------------- | ------------------------------------------------------------------------------------------------------------ |
| **rust-skills**       | Rust-specific guidance — ownership, concurrency, error handling, domain patterns, crate research, daily news |
| **rust-analyzer-lsp** | LSP integration for go-to-definition, find references, symbol analysis                                       |

### Skills

Custom skills loaded from `~/.claude/skills/`:

| Skill                  | Trigger                                 | What it does                                                                                     |
| ---------------------- | --------------------------------------- | ------------------------------------------------------------------------------------------------ |
| **br**                 | `/br`, mentions of tasks/issues/backlog | Task lifecycle management via beads_rust CLI — create, query, update, close, dependency tracking |
| **project-context**    | `/project-context`, session start       | Reads CLAUDE.md files for onboarding; updates them after significant changes                     |
| **skill-creator**      | Creating new skills                     | Guide for writing effective Claude Code skills                                                   |
| **bd-to-br-migration** | Migrating from beads to beads_rust      | Command mapping and migration patterns from `bd` to `br`                                         |

### Hooks

One hook configured on `UserPromptSubmit`:

**Prompt logger** (`log-prompt.sh`) — automatically captures every user prompt to a daily session log file (`session-log-YYYY-MM-DD.md`). Only activates within the Koupang project directory. Filters out system/command messages and empty prompts. These logs feed directly into the blog posts — it's how I track session counts and reproduce exact prompts.

> **My take**: I am quite surprised I need so few skills, plugins and hooks to be this productive. I may try to branch out further with more stuff to see if I'm missing out. Also very satisfied with having Claude generate its own summaries bceause this saves a lot of time when writing posts where the content is mostly factual and self-documenting. Making Claude write stuff that it will later use kinda reminds me of metaprogramming. If LLMs _were_ a deterministic programming language, I think it would be a combination of Ada (the "English"-like syntax) and Lisp (language capable of a lot of metaprogramming).

### Permissions

Explicitly allowed (no confirmation needed):

- **Read-only shell**: `ls`, `cat`, `find`, `grep`, `tree`, `git status/log/diff/show/tag`, etc.
- **Git write**: `git add`, `commit`, `checkout`, `merge`, `rebase`, `push`
- **Cargo**: `check`, `build`, `test`, `clippy`, `fmt`, `run`, `add`, `doc`
- **Docker**: `docker compose up/down/ps/logs`
- **Make**: all `make` targets

Explicitly denied:

- `rm`, `sudo`, `curl`, `wget`, `chmod`, `chown`, `kill`
- `git push --force`, `git reset --hard`, `git clean -f`
- `docker rm`, `docker rmi`
- `WebSearch`, `WebFetch`

Everything else prompts for confirmation.

> **My take**: "Supposedly" allowed but I still have to manually allow permissions for the "safe" commands. Gotta figure out how to properly configure allowed stuff.

### CLAUDE.md Files

The project uses hierarchical CLAUDE.md files:

- **Root** (`koupang/CLAUDE.md`) — workspace structure, tech stack, ADR summary, key imports, scripts
- **Per-service** (`identity/CLAUDE.md`, `catalog/CLAUDE.md`, `shared/CLAUDE.md`) — detailed architecture, endpoints, domain models, test structure
- **Reference docs** (`.plan/`) — bootstrap recipe, code patterns, test standards (loaded on-demand, not auto-loaded)

These are the primary onboarding mechanism — a new session reads them first and skips redundant exploration.

> **My take**: Sometimes Claude just reads all of the CLAUDE.md files and all the plan files when it loads up. Seems inconsistent but doing my best to prevent context bloat.

---

## Development Cycle

This is the general loop I follow for each feature:

1. **Plan** — enter plan mode, let Claude explore the codebase, design the approach
2. **Iterate on the plan** — review, ask questions, refine until the plan is solid
3. **Put the plan into beads** — create `br` tasks with priorities and dependencies so Claude has a structured work queue
4. **Generate code** — Claude works through tasks, launching subagents for parallel work when possible
5. **Review code** — read through what was generated, check for scope creep and pattern violations
6. **Commit code** — stage and commit with meaningful messages
7. **Clean up** — second pass to catch anything missed: redundant code, missing tests, stale docs

### What makes this work

- **CLAUDE.md files** keep context cheap. A new session doesn't waste 10 prompts re-discovering the architecture.
- **beads_rust** gives structure to multi-step work. Instead of one giant prompt, break it into tasks with dependencies and let Claude work through them.
- **The prompt logger** means I never lose track of what I asked. Blog posts practically write themselves from the logs.
- **Strict permissions** prevent accidents. No force-pushes, no `rm`, no silent `curl` calls. Everything destructive requires confirmation.
- **Plan mode first** prevents wasted work. Getting alignment on the approach before writing code is always worth the extra 5 minutes.

> **My take**: I am currently very happy with this workflow and can see myself using this workflow in the future and in jobs.
