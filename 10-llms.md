# Chapter 10: LLMs and AI-Powered Development

[← Previous: CI/CD](./09-ci-cd.md) | [Back to README](./README.md) | [Next: Putting It Together →](./11-putting-it-together.md)

## What are LLMs?

Large Language Models (LLMs) are AI systems trained on vast amounts of text data that can understand and generate human-like text. In recent years, LLMs have revolutionized software development by providing intelligent code completion, generation, and assistance.

## Benefits of LLMs in Python Development

### 1. Accelerated Development
LLMs can generate boilerplate code, implement common patterns, and help you write code faster.

### 2. Learning and Discovery
Get explanations of complex code, learn new libraries, and discover best practices as you work.

### 3. Code Review and Improvement
LLMs can suggest improvements, identify potential bugs, and help refactor code.

### 4. Documentation Assistance
Generate docstrings, write tests, and create documentation more efficiently.

### 5. Problem Solving
Get help debugging errors, understanding error messages, and finding solutions to coding challenges.

### 6. Reduced Context Switching
Get answers and generate code without leaving your editor, reducing the need to search documentation or Stack Overflow.

## AI-Powered Development Tools

Several tools integrate LLMs into the development workflow. Let's explore the major options.

## Cursor

[Cursor](https://cursor.sh) is an AI-powered code editor built on VSCode. It integrates LLMs directly into the editing experience.

### Key Features

- **AI Chat**: Ask questions about your codebase in natural language
- **Inline Editing**: AI can edit code directly in your files
- **Codebase Understanding**: AI has context about your entire project
- **Multi-file Editing**: Make changes across multiple files
- **Terminal Integration**: AI assistance in the terminal

### Getting Started with Cursor

1. **Download**: Visit [cursor.sh](https://cursor.sh) and download for your OS
2. **Import Settings**: Import your VSCode settings and extensions
3. **Add API Key**: Configure your OpenAI or Anthropic API key
4. **Start Coding**: Use Cmd+K (Mac) or Ctrl+K (Windows) for inline AI editing

### Using Cursor for Python Development

**Example workflow:**

1. **Generate a function:**
   - Select code or position cursor
   - Press Cmd+K / Ctrl+K
   - Type: "Create a function to validate email addresses"
   - AI generates the code

2. **Refactor code:**
   - Select existing code
   - Press Cmd+K / Ctrl+K
   - Type: "Refactor this to use dataclasses"
   - AI refactors your code

3. **Add tests:**
   - Select a function
   - Press Cmd+K / Ctrl+K
   - Type: "Generate pytest tests for this function"
   - AI creates comprehensive tests

4. **Explain code:**
   - Select complex code
   - Press Cmd+L / Ctrl+L for chat
   - Ask: "What does this code do?"
   - Get detailed explanations

### Cursor Tips

- Use specific prompts for better results
- Leverage the "Apply to All" feature for consistent changes
- Use the codebase indexing feature for better context
- Review AI-generated code before accepting

### Links

- [Cursor Website](https://cursor.sh)
- [Cursor on GitHub](https://github.com/getcursor) (organization)
- [Cursor Documentation](https://cursor.sh/docs)

## Antigravity

[Antigravity](https://antigravity.com) is an AI coding assistant focused on natural language to code generation.

### Key Features

- **Natural Language Interface**: Describe what you want in plain English
- **Context-Aware**: Understands your codebase and coding style
- **Multi-Language Support**: Works with Python and many other languages
- **Integration Options**: Works with popular editors

### Using Antigravity

Antigravity focuses on translating high-level descriptions into working code:

**Example:**
```
You: "Create a Python function that fetches user data from an API,
     validates it, and saves it to a database with error handling"

Antigravity: [Generates complete function with API calls, validation,
              database operations, and error handling]
```

### Links

- [Antigravity Website](https://antigravity.com)
- [Antigravity on GitHub](https://github.com/antigravity-ai/antigravity)

**Note**: Check the official website for the latest installation and usage instructions.

## Claude Code

[Claude Code](https://github.com/anthropics/claude-code) is Anthropic's official CLI tool that brings Claude AI to your terminal and development workflow.

### Key Features

- **Terminal Integration**: AI assistance directly in your terminal
- **Codebase Analysis**: Understands your project structure and code
- **File Operations**: Can read, write, and edit files
- **Multi-Step Tasks**: Handles complex workflows autonomously
- **Tool Use**: Can run commands, tests, and other tools
- **Git Integration**: Helps with commits, PRs, and git workflows

### Installing Claude Code

```bash
# Using curl (macOS/Linux)
curl -fsSL https://claude.ai/install.sh | sh

# Or using npm
npm install -g @anthropic/claude-code

# Verify installation
claude --version
```

### Setting Up Claude Code

```bash
# Authenticate
claude auth login

# Check setup
claude doctor
```

### Using Claude Code

**1. Interactive Mode:**
```bash
# Start interactive session
claude

# Then chat with Claude
> Help me create a Python function to parse CSV files

> Review this file and suggest improvements: src/parser.py

> Write tests for my calculator module
```

**2. Command Mode:**
```bash
# Generate code
claude "Create a FastAPI application with user authentication"

# Review code
claude "Review the code in src/api.py and suggest improvements"

# Generate tests
claude "Write pytest tests for all functions in src/utils.py"
```

**3. File Operations:**
```bash
# Claude can read and edit files
claude "Refactor src/database.py to use async/await"

# Multi-file changes
claude "Add type hints to all Python files in src/"
```

**4. Git Integration:**
```bash
# Help with commit messages
claude "Review my changes and create a good commit message"

# Create pull requests
claude "Create a PR for the current branch"
```

### Claude Code for Python Development

**Example workflows:**

**1. Creating a new module:**
```bash
claude "Create a Python module for handling user authentication
        with password hashing, JWT tokens, and session management"
```

**2. Adding tests:**
```bash
claude "Generate comprehensive pytest tests for src/auth.py
        including fixtures and parametrized tests"
```

**3. Code review:**
```bash
claude "Review src/api.py for potential bugs, security issues,
        and style improvements"
```

**4. Refactoring:**
```bash
claude "Refactor src/legacy.py to use modern Python 3.11 features
        and improve performance"
```

**5. Documentation:**
```bash
claude "Add comprehensive docstrings to all functions in src/
        following Google style"
```

### Claude Code Configuration

Create a `.claude-code` configuration file:

```json
{
  "model": "claude-sonnet-4.5",
  "temperature": 0.7,
  "max_tokens": 4096,
  "auto_lint": true,
  "auto_format": true,
  "test_framework": "pytest"
}
```

### Integration with Development Workflow

**1. Pre-commit hooks with Claude:**
```bash
# .git/hooks/pre-commit
#!/bin/bash
claude "Review staged changes for issues" --exit-on-error
```

**2. CI/CD integration:**
```yaml
# .github/workflows/claude-review.yml
name: Claude Code Review

on: [pull_request]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install Claude Code
        run: npm install -g @anthropic/claude-code
      - name: Review PR
        run: claude "Review this PR for potential issues"
```

### Links

- [Claude Code on GitHub](https://github.com/anthropics/claude-code)
- [Claude Code Documentation](https://docs.anthropic.com/claude-code)
- [Claude AI Platform](https://claude.ai)

## Comparing AI Development Tools

| Feature | Cursor | Antigravity | Claude Code |
|---------|--------|-------------|-------------|
| Interface | GUI (Editor) | GUI/CLI | CLI |
| Base | VSCode fork | Standalone | Terminal |
| Codebase Context | Excellent | Good | Excellent |
| Multi-file Editing | Yes | Yes | Yes |
| Terminal Integration | Built-in | Limited | Native |
| Git Integration | Basic | Basic | Advanced |
| Learning Curve | Low | Low | Medium |
| Cost | Subscription | Subscription | Pay-per-use |
| Offline Mode | No | No | No |

## Best Practices for AI-Assisted Development

### 1. Review All Generated Code

Never blindly accept AI-generated code. Always review for:
- Correctness
- Security issues
- Performance concerns
- Code style consistency

### 2. Provide Context

The more context you provide, the better the results:

**Bad:**
```
"Write a function"
```

**Good:**
```
"Write a Python function that validates email addresses using regex,
returns a boolean, and includes type hints and docstrings"
```

### 3. Iterate and Refine

Don't expect perfect results on the first try:
```
1. Generate initial code
2. Review and identify issues
3. Ask AI to fix specific problems
4. Repeat until satisfied
```

### 4. Use AI for Learning

Ask questions to understand the code:
```
"Explain how this function works step by step"
"Why did you use this pattern instead of that one?"
"What are the trade-offs of this approach?"
```

### 5. Combine with Traditional Tools

AI tools complement but don't replace:
- Linters (catch additional issues)
- Type checkers (verify type safety)
- Tests (ensure correctness)
- Code review (human judgment)

### 6. Be Specific About Requirements

```
"Create a function that:
- Accepts a list of integers
- Returns the median value
- Handles empty lists by returning None
- Includes type hints
- Has docstring with examples
- Raises TypeError for non-integer values"
```

### 7. Test AI-Generated Code

Always write or generate tests for AI code:
```bash
claude "Generate the function"
claude "Now write comprehensive tests for this function"
pytest
```

### 8. Maintain Code Quality Standards

Run your quality tools on AI-generated code:
```bash
# After AI generates code
ruff check .
ruff format .
mypy src/
pytest
```

## Practical Example: Building a Feature with AI

Let's use Claude Code to build a complete feature:

**1. Planning:**
```bash
claude "Help me plan a user authentication system for a FastAPI app.
        List the components I'll need and the implementation order."
```

**2. Implementation:**
```bash
claude "Create a user model with email, password hash, and timestamps"

claude "Create password hashing utilities using bcrypt"

claude "Create JWT token generation and validation functions"

claude "Create FastAPI endpoints for register, login, and logout"
```

**3. Testing:**
```bash
claude "Generate pytest tests for the authentication system including
        fixtures, happy paths, and error cases"
```

**4. Documentation:**
```bash
claude "Add docstrings to all functions in src/auth.py"

claude "Create a README.md section explaining how to use the auth system"
```

**5. Code Review:**
```bash
claude "Review the authentication system for security issues
        and best practices"
```

## Security Considerations

### 1. Don't Share Sensitive Data

Never paste:
- API keys or passwords
- Personal information
- Proprietary business logic
- Private data

### 2. Review for Security Issues

Check AI-generated code for:
- SQL injection vulnerabilities
- XSS vulnerabilities
- Insecure authentication
- Hardcoded secrets
- Unsafe deserialization

### 3. Use Environment Variables

Always use environment variables for secrets:
```python
# Good - AI should generate this
import os
API_KEY = os.getenv("API_KEY")

# Bad - never do this
API_KEY = "sk-1234567890abcdef"
```

## Future of AI in Development

AI tools are rapidly evolving:

- **Better Context**: Larger context windows for entire codebases
- **Improved Accuracy**: More reliable code generation
- **Specialized Models**: Models trained specifically for coding
- **Better Integration**: Deeper integration with development tools
- **Autonomous Agents**: AI agents that can complete entire features

Stay updated with the latest developments to leverage these tools effectively.

## Quick Reference

```bash
# Cursor
Cmd+K / Ctrl+K    # Inline AI editing
Cmd+L / Ctrl+L    # Open AI chat
Cmd+Shift+L       # Quick actions

# Claude Code
claude            # Interactive mode
claude "prompt"   # Single command
claude --help     # Show help

# Best practices
1. Review all generated code
2. Provide detailed context
3. Iterate and refine
4. Test thoroughly
5. Run quality checks
```

## What's Next?

Now that you've learned about all the essential tools and practices, let's put everything together in a complete project workflow.

[← Previous: CI/CD](./09-ci-cd.md) | [Back to README](./README.md) | [Next: Putting It Together →](./11-putting-it-together.md)
