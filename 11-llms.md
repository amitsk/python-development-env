# Chapter 11: LLMs and AI-Powered Development

[← Previous: CI/CD](./10-ci-cd.md) | [Back to README](./README.md) | [Next: Database Libraries →](./12-database-libraries.md)

## What are LLMs?

Large Language Models (LLMs) are neural networks trained on huge amounts of text. They predict the next token, which turns out to be enough to draft functions, explain tracebacks, and — on a good day — rename a module without inventing a new architecture.

In Python work they show up as:

- Inline completions in the editor
- Chat that can see the file (or the repo)
- Terminal agents that edit files, run `pytest`, and try again

They are assistants. They are not the type checker, the test suite, or the person who gets paged.

## Benefits of LLMs in Python Development

1. **Faster boilerplate.** Models are good at the tenth FastAPI router and the first draft of a `pyproject.toml`.
2. **Learning in place.** "Why is this `Depends()` here?" is a better use of chat than another tab of docs you will not finish.
3. **Review help.** "What did I miss?" is useful. "Ship it" from a model is not a review.
4. **Docs and tests.** First-draft docstrings and pytest cases save time *if* you still read them.
5. **Debugging a second opinion.** Pasting a traceback (with secrets stripped) often beats staring at it.
6. **Less context switching.** The answer can live in the editor or the terminal instead of a browser pile-up.

None of this replaces [Ruff](./05-linting.md), [pytest](./07-unit-testing.md), or [CI](./10-ci-cd.md). The model will not be embarrassed when `uv run pytest` fails. You will.

## Categories of AI Development Tools

| Category | Examples | Best for |
|----------|----------|----------|
| Editor / IDE assistants | GitHub Copilot, Cursor | Inline completion, chat in the IDE |
| Terminal coding agents | Claude Code, Codex, Grok Build | Multi-step tasks, repo-wide edits, running commands |
| Platform-integrated | GitHub Copilot (PRs, reviews) | Code review and collaboration on GitHub |

These products change names, models, and prices the way JavaScript frameworks used to. **Install steps and pricing belong on the vendor's site**, not memorized from a tutorial. What follows is a map of the landscape a Python developer is likely to meet, not a contract.

## GitHub Copilot

[GitHub Copilot](https://github.com/features/copilot) is GitHub's AI pair programmer. It lives in editors and across GitHub (chat, pull requests, review).

### Key Features

- **Inline completions** — ghost text as you type; Tab to accept
- **Copilot Chat** — explain, refactor, generate tests without leaving the editor
- **Agent mode** — multi-step, multi-file tasks with more autonomy than a single completion
- **Editors** — VS Code, JetBrains, Neovim, Visual Studio, and others
- **GitHub integration** — help on PRs and reviews
- **Model choice** — some plans let you pick a model; check current docs

### Getting Started

1. Get a Copilot-enabled plan at [GitHub Copilot](https://github.com/features/copilot)
2. Install the Copilot extension (VS Code: Extensions → "GitHub Copilot")
3. Sign in with GitHub
4. Code as usual — completions appear as gray text; use Chat for larger tasks

Confirm editor support and plan details in the [Copilot documentation](https://docs.github.com/en/copilot). GitHub rearranges the product surface more often than this chapter will be edited.

### Using Copilot for Python

1. **Complete a function** — write a precise signature and docstring; accept or edit the body.
2. **Chat for a feature** — "Add pytest tests for `calculate_total` with edge cases for empty lists."
3. **Explain code** — select a block, ask Chat to walk through it.
4. **Agent-style tasks** — "Add type hints to every module under `src/`." Then run `ty` yourself.

### Tips

- Specific prompts beat vibes: "validate email with a regex, return `bool`, type hints, Google-style docstring."
- Project / custom instructions exist on many plans — put Ruff, pytest, and uv in writing.
- Copilot's output is a patch, not a merge. `uv run ruff check .` and `uv run pytest` still get the last word.

### Links

- [GitHub Copilot](https://github.com/features/copilot)
- [Copilot documentation](https://docs.github.com/en/copilot)

## Cursor

[Cursor](https://cursor.com) is an AI-first editor forked from VS Code. Chat, inline edit, and multi-file agents are the product, not a plugin you remember to install.

### Key Features

- **AI chat** with codebase context
- **Inline editing** (Cmd+K / Ctrl+K style flows)
- **Multi-file edits**
- **VS Code familiarity** — extensions and keybindings if you already live there

### Getting Started

1. Download from [cursor.com](https://cursor.com)
2. Import VS Code settings if you want the same muscle memory
3. Sign in / pick a plan per Cursor's current options
4. Work from the project root so the model can see `pyproject.toml` and the tests

### Using Cursor for Python

1. **Generate a function** — cursor in the right file, inline AI, describe inputs and return type
2. **Refactor** — select code, ask for dataclasses, type hints, or async
3. **Tests** — point at a module, ask for pytest including the failure cases
4. **Explain** — select the clever bit you will not remember in a month

### Tips

- Be specific. "Make this better" is how you get a rewrite you cannot review.
- Read every apply/accept. Cursor is fast; undoing a bad apply is slower than reading.
- Keep Ruff and pytest as the quality bar.

### Links

- [Cursor](https://cursor.com)
- [Cursor documentation](https://cursor.com/docs)

## Claude Code

[Claude Code](https://github.com/anthropics/claude-code) is Anthropic's agentic coding tool for the terminal. It can read and edit files, run commands, and grind through a multi-step task until the tests pass — or until it has made a convincing mess. You decide which.

### Key Features

- Terminal-first agent
- Codebase-aware analysis and edits
- Multi-step loops (implement, test, fix)
- A natural fit for refactors and "do this across `src/`"

### Installing Claude Code

Anthropic recommends the **native installer** (no Node.js required):

```bash
# macOS / Linux / WSL
curl -fsSL https://claude.ai/install.sh | bash
```

Alternative, if you already live in npm:

```bash
npm install -g @anthropic-ai/claude-code
```

Verify:

```bash
claude --version
```

Auth, plans, and which models you get change. Follow the [Claude Code docs](https://docs.anthropic.com/en/docs/claude-code) rather than a screenshot from last quarter.

### Using Claude Code

Interactive session:

```bash
cd myproject
claude
```

Then describe the outcome:

```text
Add a FastAPI health endpoint and pytest coverage for it
```

One-shot style prompts (flags vary; check `claude --help`):

```bash
claude "Review src/parser.py for edge cases and suggest tests"
```

### Example Python workflows

```bash
# New module
claude "Create a small auth module with password hashing and JWT helpers"

# Tests
claude "Write pytest tests for src/auth.py including failure cases"

# Quality pass — make the agent eat its vegetables
claude "Run ruff and pytest, then fix any issues you introduced"
```

### Links

- [Claude Code on GitHub](https://github.com/anthropics/claude-code)
- [Claude Code documentation](https://docs.anthropic.com/en/docs/claude-code)

## OpenAI Codex

[OpenAI Codex](https://openai.com/codex/) is OpenAI's coding-agent family. The open-source [Codex CLI](https://github.com/openai/codex) runs locally: it inspects the repo, edits files, and runs commands. Codex also appears in ChatGPT / cloud workflows depending on what OpenAI is shipping this month.

### Key Features

- Terminal agent with repo context
- Multi-step implement-and-debug loops
- Local CLI (edit + run on your machine)
- Ties to OpenAI / ChatGPT plans where applicable
- Editor integrations for a hybrid IDE + agent setup

### Installing Codex CLI

Prefer the install path on [openai.com/codex](https://openai.com/codex/) and the [GitHub README](https://github.com/openai/codex). Common options:

```bash
# Standalone installer (macOS / Linux) — confirm on official docs
curl -fsSL https://chatgpt.com/codex/install.sh | sh

# npm
npm install -g @openai/codex

# Homebrew
brew install --cask codex
```

Then, from a project directory:

```bash
cd myproject
codex
```

Sign in with a ChatGPT account or an API key, per current docs.

### Using Codex for Python

```bash
# Interactive agent
codex

# Task-oriented prompts (exact flags depend on CLI version)
codex "Add a CLI entry point with Typer and a --verbose flag"
codex "Fix the failing pytest in tests/test_main.py"
```

**Good fits:**

- Features with tests in a tight loop
- Repo exploration and refactors
- Teams already standardized on OpenAI models

### Tips

- Keep an `AGENTS.md` (or whatever instruction file the agent reads) so Ruff, pytest, and uv are not a surprise.
- Small diffs you can review beat a 40-file rewrite you will rubber-stamp at 6 p.m.
- After the agent stops talking, run `uv run pytest` and `uv run ruff check .` yourself.

### Links

- [OpenAI Codex](https://openai.com/codex/)
- [Codex CLI on GitHub](https://github.com/openai/codex)

## Grok Build

**Grok Build** is xAI's terminal coding agent. It explores a codebase, edits files, runs commands, and iterates until checks pass — the same job as the other terminal agents, with Grok models underneath.

### Key Features

- Agent-style development in a terminal UI
- Project-aware edits (read, search, patch)
- Command execution for builds, tests, and linters
- End-to-end "implement this feature" tasks
- Skills / structured workflows for review, plan, and execute

Do not invent extra APIs for it. If a flag is not in `grok --help` or the official docs, it does not exist in this tutorial.

### Getting Started

```bash
# macOS / Linux / WSL (common install; confirm on xAI docs)
curl -fsSL https://x.ai/cli/install.sh | bash

grok --version
```

Then, in a project:

```bash
cd myproject
grok
```

Access, auth, and pricing live on xAI's site. Start at [x.ai](https://x.ai/) and the current Grok Build / CLI docs rather than assuming a plan from a blog post.

### Using Grok Build for Python

Prompts that match this tutorial's stack:

```text
Set up this repo with uv, Ruff, pytest, and a GitHub Actions CI workflow
matching our existing pyproject.toml conventions.

Add SQLModel models and a FastAPI CRUD API for a Hero resource,
with pytest + FastAPI TestClient coverage.

Review uncommitted changes for security issues and missing tests.
```

Ask it to **run** the tools, not just generate code that would theoretically pass them.

### Tips

- Point the agent at `pyproject.toml` and `.github/workflows/ci.yml` so it copies *your* standards.
- Require `uv run ruff check .`, `uv run pytest`, and the type checker as part of the task.
- Treat it like a fast junior teammate: clear requirements, incremental work, human review.

### Links

- [xAI](https://x.ai/)
- CLI install and product docs: start from [x.ai/cli](https://x.ai/cli) and current Grok Build documentation

## Comparing AI Development Tools

| Feature | GitHub Copilot | Cursor | Claude Code | Codex | Grok Build |
|---------|----------------|--------|-------------|-------|------------|
| Primary interface | Editor + GitHub | AI IDE (VS Code fork) | Terminal agent | Terminal agent (+ ecosystem) | Terminal / agent UI |
| Inline completion | Excellent | Excellent | Limited (agent-first) | Varies | Agent-first |
| Multi-file agents | Yes (agent mode) | Yes | Yes | Yes | Yes |
| Runs shell commands | Via agent / CLI | Via agent features | Yes | Yes | Yes |
| GitHub-native workflows | Strong | Basic | Strong via CLI | Varies | Varies |
| Learning curve | Low | Low | Medium | Medium | Medium |
| Typical access model | Subscription (GitHub) | Subscription | Plan / API (Anthropic) | OpenAI / ChatGPT ecosystem | xAI / Grok product |

Plans, models, and prices move. Confirm on each product's site before you standardize a team on one of them.

## Best Practices for AI-Assisted Development

### 1. Review All Generated Code

Never merge what you have not read. Check:

- Correctness (does it do what the ticket asked?)
- Security (injection, authz, path traversal, SSRF — the greatest hits)
- Performance (N+1 queries, loading a 200 MB JSON into memory "for simplicity")
- Style (does it match the rest of `src/`?)

A model that sounds sure of itself is not a source.

### 2. Provide Context

**Weak:**

```text
Write a function
```

**Strong:**

```text
Write a Python function that validates email addresses with a regex,
returns bool, includes type hints and a short docstring with examples.
Use the project's Ruff-formatted style.
```

### 3. Iterate and Refine

1. Generate a first draft
2. Point at the actual problems
3. Ask for a targeted fix, not a rewrite of the universe
4. Re-run tests and linters

### 4. Use AI for Learning

```text
Explain how this function works step by step
Why this pattern instead of X?
What are the trade-offs?
```

If you cannot explain the code after the chat, you cannot maintain it.

### 5. Combine with Traditional Tools

AI complements — it does not replace:

- Linters (Ruff)
- Type checkers (ty / mypy)
- Tests (pytest)
- Human review
- CI on GitHub Actions

### 6. Be Specific About Requirements

```text
Create a function that:
- Accepts a list of integers
- Returns the median, or None for empty input
- Includes type hints
- Has a docstring with examples
- Raises TypeError for non-integer values
```

### 7. Test AI-Generated Code

```bash
# After AI edits — no exceptions for "the agent said it passed"
uv run ruff check .
uv run ruff format --check .
uv run pytest
```

If the agent ran these, run them again. The thirty seconds are cheaper than the revert.

### 8. Keep Secrets Out of Prompts

Never paste:

- API keys, tokens, passwords
- Private customer data
- Production credentials, connection strings, or `.env` files
- Internal URLs that are the whole security model

Use environment variables and gitignored `.env` files ([Chapter 4](./04-version-control.md)). If a tool needs cloud auth, use its documented login flow — do not paste a key "just this once."

## Practical Example: Building a Feature with an Agent

Using a terminal agent (Claude Code, Codex, or Grok Build):

**1. Planning**

```text
Help me plan a small user authentication module for a FastAPI app.
List components and implementation order. Prefer uv + pytest + Ruff.
```

**2. Implementation**

```text
Create a user model, password hashing helpers, and JWT issue/verify functions.
```

**3. Testing**

```text
Generate pytest tests with fixtures, happy paths, and failure cases.
```

**4. Quality**

```text
Run ruff and pytest; fix anything that fails.
```

**5. Review**

```text
Review the auth module for security issues and missing edge cases.
```

Then *you* read the diff. Hashing, JWT, and "looks like auth" are where confident-wrong code is expensive.

## Security Considerations

1. **Do not share secrets** with any AI tool
2. **Review for injection and auth bugs** in generated web and database code
3. **Prefer env vars** for configuration:

```python
import os

API_KEY = os.environ["API_KEY"]  # fail fast if missing
```

Generated code loves `API_KEY = "sk-..."` in a constant. Delete that on sight.

## Quick Reference

```text
# GitHub Copilot (VS Code)
# - Install GitHub Copilot + Copilot Chat extensions
# - Tab to accept completions; open Chat for larger tasks

# Cursor
# Cmd/Ctrl+K  inline edit
# Cmd/Ctrl+L  chat (typical defaults; confirm in your build)

# Claude Code
curl -fsSL https://claude.ai/install.sh | bash
claude
claude "Implement X and add tests"

# Codex CLI
# see https://github.com/openai/codex and https://openai.com/codex/
codex
codex "Implement X and add tests"

# Grok Build
curl -fsSL https://x.ai/cli/install.sh | bash
grok
# Ask the agent to run: uv run ruff check . && uv run pytest

# Always
uv run ruff check .
uv run pytest
```

Install commands, flags, and pricing: official docs. This page will be wrong about a flag before it is wrong about "run the tests."

## What's Next?

AI tools speed up the typing. They do not pick your database layer. Next: DBAPI, SQLAlchemy, and SQLModel.

[← Previous: CI/CD](./10-ci-cd.md) | [Back to README](./README.md) | [Next: Database Libraries →](./12-database-libraries.md)
