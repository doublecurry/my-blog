---
title: "How I Use Every Claude Code Feature: A Comprehensive Guide"
description: "A reference manual for daily-driving Claude Code — CLAUDE.md discipline, context management, hooks, skills, MCP, the SDK, and GitHub Actions."
created: 2026-03-17
updated: 2026-05-11
tags:
  - Claude Code
  - Developer Workflow
  - Agents
---

# How I Use Every Claude Code Feature

As a heavy user of [Claude Code](https://code.claude.com/docs/en/overview), I run it in a VM a few times a week for personal projects, often using `--dangerously-skip-permissions` to let it move freely. At work, my team builds AI IDE rules where code generation alone consumes billions of tokens monthly.

The CLI agent space is crowded right now — Gemini CLI, Codex CLI, Cursor CLI, Copilot CLI — but the real battle is between Anthropic and OpenAI. Developers often get caught up in output styles or system-prompt "vibes," yet these tools are already incredibly solid. The goal is simply to hand off a task, set the context, and evaluate the final Pull Request — not to micromanage the process.

After months of daily-driving Claude Code, here is a deep dive into its ecosystem. Treat this as a reference manual rather than a one-shot read.

---

## `CLAUDE.md`: Setting the Rules of Engagement

The `CLAUDE.md` file in your root directory is the agent's behavioral guide. How you treat it depends on the context:

* **Personal Projects:** Let Claude do whatever it wants.
* **Work / Monorepos:** Keep it strictly curated. Our monorepo `CLAUDE.md` is currently around 13KB. It only documents tools used by a large portion of engineers. If a tool can't be explained concisely, it isn't ready for `CLAUDE.md`.

**Tips and anti-patterns:**

1. **Restrict first, don't just guide.** Start small and add rules based on mistakes Claude actually makes.
2. **Avoid @-mentioning docs everywhere.** Referencing huge files bloats the context window. Instead, explain *why* and *when* to check a specific path (e.g. *"For complex usage, see path/to/docs.md"*).
3. **Don't just say "No".** Avoid pure negative constraints. Give the agent a viable alternative so it doesn't get stuck looping on a forbidden flag.
4. **Use it as a forcing function.** If your CLI commands are too complex to explain, write a simple bash wrapper and document *that* instead.

## Context Management: Compression and Cleanup

Running `/context` reveals how your 200k-token window is utilized. A baseline session may start at 20k tokens, leaving 180k for actual work — which fills up fast.

Three main workflows for managing this:

* `/compact` **(avoid):** the automatic compression is opaque and poorly optimized.
* `/clear` + `/catchup` **(simple restart):** clear the state, then use a custom `/catchup` command to make Claude read all changed files on the current git branch. This is the best default.
* **"Record and clear" (complex restart):** for large tasks, have Claude write its plan to a `.md` file, clear the context, then start a new session that reads that file.

## Custom Slash Commands

Treat these as simple shortcuts for common prompts. A massive list of complex custom commands is an anti-pattern. The beauty of an agent is typing what you want and getting a result — not memorizing magic spells.

My setup is highly lean:

* `/catchup` — prompts Claude to read changed files.
* `/pr` — cleans code, stages changes, and preps a Pull Request.

## Custom Subagents

In theory, subagents save context by outsourcing tasks and returning only the final answer. In practice, they create two problems:

1. **Gatekeeping context.** If you create a `PythonTests` subagent, the main agent loses context on how to test its own code. It becomes entirely reliant on the subagent.
2. **Rigid workflows.** It forces the agent into human-defined delegation.

**The alternative:** put key context in `CLAUDE.md` and let the main agent use the built-in `Task(...)` feature to clone itself dynamically. This "main-clone" pattern lets the agent manage its own delegation while keeping the overarching context intact.

## Resume, Continue, and History

Commands like `claude --resume` and `claude --continue` are excellent for restarting a broken terminal or recovering an old session to see exactly how the agent solved a past error. Claude stores session history in `~/.claude/projects/`, which is incredibly useful for meta-analysis. You can script log reviews to find common permission requests and errors, then fold the lessons back into your contextual setup.

## Hooks

Hooks provide deterministic "must-do" rules, complementing the "should-do" advice in `CLAUDE.md`.

* **Block-at-submit hooks.** We use a `PreToolUse` hook wrapping `git commit`. It checks for a success file generated only when tests pass; if it's missing, the hook blocks the commit and forces the agent into a test-and-fix loop.
* **Hint hooks.** Non-blocking, fire-and-forget feedback if the agent makes a suboptimal choice.

Avoid block-at-write hooks. Stopping an agent mid-plan causes confusion. Let it finish its work, then check the final result at the submit stage.

## Plan Mode

Plan mode is essential for large feature changes. The built-in Plan mode aligns you and the agent on *how* to build something and sets checkpoints before it starts writing code.

At the enterprise level, you can build custom planning tools on top of the Agent SDK to enforce internal best practices and architectural designs natively.

## Skills

Skills formalize the "scripting" model of agent autonomy. Instead of manually creating rigid tools for every action, you give the agent access to raw environments (scripts, binaries) so it can write code to interact with them dynamically. A `SKILL.md` file is a structured, shareable way to expose these CLIs and scripts to the agent.

## Model Context Protocol (MCP)

MCP isn't dead, but its role has shifted. An MCP shouldn't be a bloated REST API clone mirroring dozens of basic functions — it should act as a simple, secure **gateway**.

Give the agent one or two high-level tools (like raw data access or sandboxed code execution) and let it use its scripting skills to do the rest. Most stateless tools (Jira, AWS, GitHub) are better handled via simple CLIs, leaving MCP for complex, stateful environments like Playwright.

## Claude Code (Agent) SDK

The SDK is a powerful framework for building entirely new agents.

1. **Mass parallel scripting.** For massive refactors, writing simple bash scripts that run parallel `claude -p` commands scales infinitely better than managing dozens of subagent tasks.
2. **Internal chat tools.** Wrapping complex processes into simple UIs for non-technical users.
3. **Rapid prototyping.** Quickly testing new agent ideas before committing to a heavier framework.

## Claude Code in GitHub Actions

Running Claude Code inside GitHub Actions is one of its most underrated capabilities. It provides a sandboxed, highly customizable environment with full access to Hooks and MCP.

You can use it to build "PR from anywhere" workflows — trigger a fix from Slack or a CloudWatch alert, and GHA returns a fully tested PR. Reviewing GHA logs regularly creates a data-driven flywheel: analyze bugs, improve your `CLAUDE.md`, deploy a better agent.

## `settings.json`

A few configurations worth keeping in mind:

* `HTTPS_PROXY` / `HTTP_PROXY` — great for debugging raw traffic and sandboxing network access.
* `MCP_TOOL_TIMEOUT` / `BASH_MAX_TIMEOUT_MS` — increase these. The defaults are often too conservative for complex, long-running commands.
* `ANTHROPIC_API_KEY` — swap to an enterprise, usage-based key if you prefer that over a per-seat license.
* `permissions` — regularly audit the exact list of commands you allow Claude to run automatically.

## Wrap Up

If you haven't started using command-line coding agents yet, dive in. There are very few exhaustive guides on these advanced features, so the best way to learn is to put them to the test in your own terminal.
