---
title: "Building a Lean AI Coding Assistant: My Pi Package Setup"
description: "A hands-on walkthrough of configuring pi's package ecosystem — context compression, code search, safety gates, and custom tooling."
created: 2026-06-05
updated: 2026-06-05
tags:
  - pi
  - Developer Workflow
  - AI Tools
---

# Building a Lean AI Coding Assistant: My Pi(π) Package Setup

I've been running **pi** as my terminal AI coding tool for a few months now. After working through its package ecosystem, I have some practical thoughts on what works, what doesn't, and how I ended up with a setup that feels genuinely useful.

---

## Why Pi(π) in the First Place

Before pi, I used Claude Code and a few other tools. They worked fine for small projects, but once the codebase grew, context handling became messy. The tools would pull in too much irrelevant information, and the model's understanding would drift.

My takeaway: **controlling exactly what the model sees matters more than adding features.**

That's what drew me to **pi**. It's deliberately minimal:

* The core system prompt is under 1000 tokens.
* No built-in MCP, background bash runners, todo lists, or plan modes.
* Just four base tools: `read`, `write`, `edit`, and `bash`.

It's bare-bones by design. Everything else comes from packages:

* **Extensions:** register new tools, commands, or UI components.
* **Skills:** instruction sets that guide the model through specific workflows.
* **Prompt Templates**
* **Themes**

---

## The Packages I Actually Use

Here's what I've installed, what I kept, and what I ended up removing.

### 1. Core Functionality Gaps

* **[pi-subagents](https://www.npmjs.com/package/pi-subagents)** and **[pi-mcp-adapter](https://www.npmjs.com/package/pi-mcp-adapter)**: these fill in features other tools have by default — subagents and MCP interfaces. I kept subagents but skipped the MCP adapter. Most of what I need from MCP can just be built as a normal package.

### 2. Context Compression and Management

This took the longest to figure out. I tested three different approaches:

* **[context-mode](https://www.npmjs.com/package/context-mode)**: it's the most-installed package, but I removed it after a week. It intercepts tool output and only shows the model a summary. The problem: the model constantly made wrong calls because key details were stripped out. Saving a few tokens isn't worth the accuracy hit.
* **[pi-dynamic-context-pruning (DCP)](https://github.com/complexthings/pi-dynamic-context-pruning)**: summarization-based pruning. This one actually works as expected. I'm using it now.
* **[pi-observational-memory](https://www.npmjs.com/package/pi-observational-memory)**: runs a separate model to build a memory layer, which helps prevent drift in long sessions. It's more complex than DCP, so I'm still testing it.

### 3. Continuous Goal Execution

* **[pi-until-done](https://www.npmjs.com/package/pi-until-done)**: when I give it a goal, I want the agent to keep running until it's finished, not stop after every step. This does exactly that. If you prefer something closer to Codex's task-oriented style, try **[pi-codex-goal](https://www.npmjs.com/package/pi-codex-goal)**.

### 4. Code Search and Retrieval

* **[pi-ace-tool](https://github.com/justhil/pi-ace-tool)**: a code retrieval plugin I wrote. If you've used MCP-based retrieval before, this should feel familiar.
* **[@ff-labs/pi-fff](https://www.npmjs.com/package/@ff-labs/pi-fff)**: a Rust-based fuzzy finder with SIMD acceleration. Replaces native `find` and `grep` with something noticeably faster.
* **[pi-fast-context](https://github.com/justhil/pi-fast-context)**: I threw this together for quick context loading. Works well enough for daily use.

### 5. Web Search and Scraping

* **[pi-search](https://github.com/justhil/pi-search)**: I rewrote a Grok search MCP tool and added context7 integration plus anti-detection extractors. This covers web search and scraping in one package. If you want something lighter, check out **[pi-web-access](https://www.npmjs.com/package/pi-web-access)** or **[pi-smart-fetch](https://www.npmjs.com/package/pi-smart-fetch)**.

### 6. Safety, Review, and Extended Thinking

* **[@narumitw/pi-plan-mode](https://www.npmjs.com/package/@narumitw/pi-plan-mode)**: entering `/plan` locks the agent into read-only mode. It can't modify files or run dangerous commands until I confirm the plan. Simple and effective.
* **[pi-btw](https://www.npmjs.com/package/pi-btw)**: same as Claude Code's `/btw` — lets you spin off a side discussion without polluting the main conversation's code context.
* **[@feniix/pi-sequential-thinking](https://www.npmjs.com/package/@feniix/pi-sequential-thinking)**: adds sequential thinking capability.
* **[@juicesharp/rpiv-advisor](https://www.npmjs.com/package/@juicesharp/rpiv-advisor)** and **[pi-simplify](https://www.npmjs.com/package/pi-simplify)**: the first gives a second opinion before critical decisions; the second reviews recent code changes for clarity and consistency.

### 7. UI and Interaction Improvements

* **[pi-nano-context](https://www.npmjs.com/package/pi-nano-context)**: a lightweight context usage bar. Shows how much space system, user, assistant, and tool messages are taking up. Clean and unobtrusive.
* **[pi-tool-display](https://www.npmjs.com/package/pi-tool-display)**: folds tool output and renders diffs. Prevents the terminal from getting flooded with raw logs.
* **[pi-markdown-preview](https://www.npmjs.com/package/pi-markdown-preview)**: previews Markdown and LaTeX.
* **[@juicesharp/rpiv-ask-user-question](https://www.npmjs.com/package/@juicesharp/rpiv-ask-user-question)**: structured question UI for when the agent needs clarification.

### 8. Image Generation and Operation Rollback

* **[pi-image-gen](https://github.com/justhil/pi-image-gen)**: Image2-based generation and editing. I use this before building frontend components — quick way to generate design references or placeholder assets.
* **[pi-rewind](https://www.npmjs.com/package/pi-rewind)**: Git-based rollback tool. Native-level undo for agent operations.

---

## What I Removed

I tried **[pi-powerline-footer](https://www.npmjs.com/package/pi-powerline-footer)**, but it was way too heavy. It takes over the editor layout and hijacks scroll behavior, which ruins pi's clean TUI. I switched back to **[pi-nano-context](https://www.npmjs.com/package/pi-nano-context)** — just shows context usage without touching anything else.

---

## Some Observations

Pi's extension interface is solid, but a lot of package authors seem to default to "throw everything in." That misses the point. The tool is minimal for a reason.

Finding a package that does one thing well, with minimal dependencies, is harder than it should be. My suggestion: **use AI to write custom packages for your own workflow.** If you already have a solid workflow in another tool, just port it to pi with an AI-generated package. It's faster than hunting through half-finished ecosystem experiments.
