# Chapter 11: LLMs and AI-Powered Development

[← Previous: CI/CD](./10-ci-cd.md) | [Back to README](./README.md) | [Next: Database Libraries →](./12-database-libraries.md)

## What are LLMs?

Large Language Models (LLMs) are AI systems trained on vast amounts of text data that can understand and generate human-like text. In recent years, LLMs have become a core part of software development: code completion, generation, review, refactoring, and multi-step agent workflows.

## Benefits of LLMs in Python Development

### 1. Accelerated Development
LLMs can generate boilerplate, implement common patterns, and help you write code faster.

### 2. Learning and Discovery
Get explanations of complex code, learn new libraries, and discover best practices as you work.

### 3. Code Review and Improvement
LLMs can suggest improvements, identify potential bugs, and help refactor code.

### 4. Documentation Assistance
Generate docstrings, write tests, and draft documentation more efficiently.

### 5. Problem Solving
Get help debugging errors, understanding error messages, and finding solutions to coding challenges.

### 6. Reduced Context Switching
Get answers and generate code without leaving your editor or terminal as often.

## Categories of AI Development Tools

| Category | Examples | Best for |
|----------|----------|----------|
| Editor / IDE assistants | GitHub Copilot, Cursor | Inline completion, chat in the IDE |
| Terminal coding agents | Claude Code, Codex, Grok Build | Multi-step tasks, repo-wide edits, running commands |
| Platform-integrated | GitHub Copilot (PRs, reviews) | Code review and collaboration on GitHub |

AI tools change quickly. Prefer official docs for install steps and pricing, and treat the examples below as a starting map rather than permanent API truth.

## GitHub Copilot

[GitHub Copilot](https://github.com/features/copilot) is GitHub’s AI pair programmer. It works inside popular editors and across the GitHub product surface (chat, PR assistance, code review workflows).

### Key Features

- **Inline completions**: Suggests lines and whole functions as you type (accept with Tab)
- **Copilot Chat**: Ask questions, explain code, refactor, and generate tests in the editor
- **Agent mode**: Multi-step tasks across files with more autonomy than single completions
- **Editor support**: VS Code, JetBrains IDEs, Neovim, Visual Studio, and others
- **GitHub integration**: Assistance in pull requests, reviews, and related GitHub workflows
- **Multi-model options**: Plan tiers may offer choice among models (check current product docs)

### Getting Started

1. Get a Copilot-enabled plan via [GitHub Copilot](https://github.com/features/copilot)
2. Install the Copilot extension for your editor (for VS Code: Extensions → “GitHub Copilot”)
3. Sign in with your GitHub account
4. Start coding — suggestions appear as gray ghost text; use Chat for larger tasks

### Using Copilot for Python

**Typical workflows:**

1. **Complete a function**  
   Write a clear function signature and docstring; accept or edit the suggested body.

2. **Chat for a feature**  
   Open Copilot Chat and ask: “Add pytest tests for `calculate_total` with edge cases for empty lists.”

3. **Explain code**  
   Select a block → ask Copilot Chat to explain it step by step.

4. **Agent-style tasks**  
   Use agent mode for multi-file changes (for example: “Add type hints to all modules under `src/`”).

### Tips

- Prefer specific prompts (“validate email with regex, return bool, type hints, Google-style docstring”)
- Use project/custom instructions where your plan supports them so Copilot follows team conventions
- Always run your usual quality gate: Ruff, type checker, and pytest on AI-written code

### Links

- [GitHub Copilot](https://github.com/features/copilot)
- [Copilot documentation](https://docs.github.com/en/copilot)

## Cursor

[Cursor](https://cursor.com) is an AI-first code editor built as a fork of VS Code. It embeds chat, inline edit, and multi-file agent workflows deeply into the editing experience.

### Key Features

- **AI chat** with codebase context
- **Inline editing** (for example Cmd+K / Ctrl+K style flows)
- **Multi-file edits** across a project
- **VS Code familiarity**: extensions and keybindings feel familiar if you already use VS Code

### Getting Started

1. Download from [cursor.com](https://cursor.com)
2. Import VS Code settings/extensions if you want
3. Sign in / choose a plan per Cursor’s current product options
4. Use inline edit and chat while working in the project root so the AI sees your codebase

### Using Cursor for Python

1. **Generate a function** — position the cursor, invoke inline AI, describe the function  
2. **Refactor** — select code, ask to use dataclasses / type hints / async  
3. **Tests** — select a module, ask for pytest coverage including edge cases  
4. **Explain** — select complex code and ask for a step-by-step explanation  

### Tips

- Be specific in prompts
- Review every apply/accept before committing
- Keep Ruff + pytest as the source of truth for quality

### Links

- [Cursor website](https://cursor.com)
- [Cursor documentation](https://cursor.com/docs)

## Claude Code

[Claude Code](https://github.com/anthropics/claude-code) is Anthropic’s agentic coding tool for the terminal (and related workflows). It can read and edit files, run commands, and work through multi-step engineering tasks.

### Key Features

- Terminal-first agent workflow
- Codebase-aware analysis and edits
- Multi-step tasks (implement, test, fix, commit help)
- Strong fit for refactoring, debugging, and “do this across the repo” work

### Installing Claude Code

Prefer the **native installer** (recommended by Anthropic; no Node.js required):

```bash
# macOS / Linux / WSL
curl -fsSL https://claude.ai/install.sh | bash
```

Alternative (npm delivery of the CLI):

```bash
npm install -g @anthropic-ai/claude-code
```

Verify:

```bash
claude --version
```

Follow current auth instructions from the [Claude Code docs](https://docs.anthropic.com/en/docs/claude-code) (account / plan requirements change over time).

### Using Claude Code

**Interactive session:**

```bash
cd myproject
claude
```

Then describe tasks in natural language, for example:

```text
Add a FastAPI health endpoint and pytest coverage for it
```

**One-shot style prompts** (patterns vary by version; check `claude --help`):

```bash
claude "Review src/parser.py for edge cases and suggest tests"
```

### Example Python workflows

```bash
# New module
claude "Create a small auth module with password hashing and JWT helpers"

# Tests
claude "Write pytest tests for src/auth.py including failure cases"

# Quality pass
claude "Run ruff and pytest, then fix any issues you introduced"
```

### Links

- [Claude Code on GitHub](https://github.com/anthropics/claude-code)
- [Claude Code documentation](https://docs.anthropic.com/en/docs/claude-code)

## OpenAI Codex

[OpenAI Codex](https://openai.com/codex/) (including the open-source [Codex CLI](https://github.com/openai/codex)) is OpenAI’s coding agent family. The CLI runs in your terminal, can inspect a repository, edit files, and run commands. Codex also appears in ChatGPT / cloud workflows depending on product availability.

### Key Features

- Terminal coding agent with repo context
- Multi-step implementation and debugging loops
- Local-first CLI workflows (edit + run on your machine)
- Ecosystem ties to OpenAI accounts / ChatGPT plans where applicable
- Editor integrations may be available for hybrid IDE + agent use

### Installing Codex CLI

Check the official install path for your OS. Common patterns:

```bash
# Example: install via npm (verify current package name in docs)
npm install -g @openai/codex

# Or follow the GitHub README for the recommended installer
# https://github.com/openai/codex
```

Authenticate per current OpenAI / Codex documentation, then run from a project directory:

```bash
cd myproject
codex
```

### Using Codex for Python

Typical tasks:

```bash
# Interactive agent
codex

# Task-oriented prompts (exact flags depend on CLI version)
codex "Add a CLI entry point with Typer and a --verbose flag"
codex "Fix the failing pytest in tests/test_main.py"
```

**Good fits:**

- Implementing features with tests in a tight loop  
- Repo exploration and refactoring  
- Teams already standardized on OpenAI models  

### Tips

- Keep an `AGENTS.md` or project instructions file if the agent supports it, so conventions (Ruff, pytest, uv) are explicit  
- Prefer small, reviewable diffs over huge unreviewed rewrites  
- Always re-run `uv run pytest` and `uv run ruff check .` after agent edits  

### Links

- [OpenAI Codex](https://openai.com/codex/)
- [Codex CLI on GitHub](https://github.com/openai/codex)

## Grok Build

**Grok Build** is xAI’s AI-assisted software engineering environment (terminal UI / agent workflows powered by Grok models). It is designed for multi-step coding work: exploring a codebase, editing files, running commands, and iterating until tests and checks pass.

### Key Features

- Agent-style development in a terminal UI
- Project-aware edits (read, search, patch files)
- Command execution for builds, tests, and linters
- Strong fit for “implement this feature end-to-end” tasks
- Skills / structured workflows for common engineering jobs (review, plan, execute)

### Getting Started

Follow current xAI / Grok product documentation for access and install. Once available in your environment:

1. Open your project directory  
2. Start Grok Build  
3. Describe the outcome you want (not only a single file change)  
4. Review diffs, then run your quality tools  

### Using Grok Build for Python

Example prompts that work well with this tutorial’s stack:

```text
Set up this repo with uv, Ruff, pytest, and a GitHub Actions CI workflow
matching our existing pyproject.toml conventions.

Add SQLModel models and a FastAPI CRUD API for a Hero resource,
with pytest + httpx TestClient coverage.

Review uncommitted changes for security issues and missing tests.
```

### Tips

- Point the agent at your golden `pyproject.toml` and CI config so it follows project standards  
- Ask it to **run** `uv run ruff check .`, `uv run pytest`, and type checks rather than only generate code  
- Treat Grok Build like a junior teammate: clear requirements, incremental tasks, human review  

### Links

- [xAI](https://x.ai/)
- Product and install details: see current Grok / xAI developer documentation for Grok Build

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

Plans, models, and pricing change often — confirm on each product’s site before adopting for a team.

## Best Practices for AI-Assisted Development

### 1. Review All Generated Code

Never blindly accept AI-generated code. Always review for:

- Correctness  
- Security issues  
- Performance concerns  
- Consistency with project style  

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

1. Generate initial code  
2. Review and identify issues  
3. Ask the tool to fix specific problems  
4. Re-run tests and linters  

### 4. Use AI for Learning

```text
Explain how this function works step by step
Why this pattern instead of X?
What are the trade-offs?
```

### 5. Combine with Traditional Tools

AI complements — it does not replace:

- Linters (Ruff)  
- Type checkers (ty / mypy)  
- Tests (pytest)  
- Human code review  

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
# After AI edits
uv run ruff check .
uv run ruff format --check .
uv run pytest
```

### 8. Keep Secrets Out of Prompts

Never paste:

- API keys or passwords  
- Private customer data  
- Production credentials  

Use environment variables and `.env` files that stay out of git (see [Chapter 4](./04-version-control.md)).

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

## Security Considerations

1. **Do not share secrets** with any AI tool  
2. **Review for injection and auth bugs** in generated web/DB code  
3. **Prefer env vars** for configuration:

```python
import os

API_KEY = os.environ["API_KEY"]  # fail fast if missing
```

## Quick Reference

```text
# GitHub Copilot (VS Code)
# - Install GitHub Copilot + Copilot Chat extensions
# - Tab to accept completions; open Chat for larger tasks

# Cursor
# Cmd/Ctrl+K  inline edit
# Cmd/Ctrl+L  chat (typical defaults; confirm in your build)

# Claude Code
claude
claude "Implement X and add tests"

# Codex CLI
codex
codex "Implement X and add tests"

# Grok Build
# Start from your project root; describe the full outcome
# Ask the agent to run: uv run ruff check . && uv run pytest

# Always
uv run ruff check .
uv run pytest
```

## What's Next?

AI tools speed up implementation, but data and APIs still need solid libraries. Next: database access with DBAPI, SQLAlchemy, and SQLModel.

[← Previous: CI/CD](./10-ci-cd.md) | [Back to README](./README.md) | [Next: Database Libraries →](./12-database-libraries.md)
