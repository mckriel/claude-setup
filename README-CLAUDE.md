# How to Use Agents and Commands

This guide explains how the Claude Code agent system works in this project. If you've never used agents before, start here.

## Quick Setup

Pull the setup into any project:

```bash
npx degit mckriel/claude-setup/.claude .claude --force
npx degit mckriel/claude-setup/docs docs --force
curl -sO https://raw.githubusercontent.com/mckriel/claude-setup/main/CLAUDE.md
```

Or add this function to your `~/.zshrc`:

```bash
claude-init() {
  local force=false
  [[ "$1" == "--force" ]] && force=true

  local existing=()
  [[ -d ".claude" ]] && existing+=(".claude/")
  [[ -d "docs" ]] && existing+=("docs/")
  [[ -f "CLAUDE.md" ]] && existing+=("CLAUDE.md")

  if [[ ${#existing[@]} -gt 0 ]] && [[ "$force" == false ]]; then
    echo "⚠️  Found existing: ${existing[*]}"
    echo "This will overwrite them. Run 'claude-init --force' to proceed."
    return 1
  fi

  npx degit mckriel/claude-setup/.claude .claude --force || { echo "❌ Failed to fetch .claude/"; return 1; }
  npx degit mckriel/claude-setup/docs docs --force || { echo "❌ Failed to fetch docs/"; return 1; }
  curl -sfO https://raw.githubusercontent.com/mckriel/claude-setup/main/CLAUDE.md || { echo "❌ Failed to fetch CLAUDE.md"; return 1; }
  echo "✅ Claude setup initialized!"
}
```

Then `source ~/.zshrc` and run `claude-init` in any project directory. Use `claude-init --force` to overwrite existing files.

## The Two Building Blocks

This project has two things that work together:

| What | Where | How you use it |
|------|-------|----------------|
| **Commands** | `.claude/commands/` | You type them as slash commands (e.g. `/implement`) |
| **Agents** | `.claude/agents/` | You don't invoke these directly. Commands and Claude invoke them behind the scenes. |

Think of it this way: **commands are the buttons you press, agents are the workers that do the job.**

## Using Commands (Slash Commands)

Commands are your primary interface. You type them directly into Claude Code's chat input prefixed with `/`.

### Available Commands

| Command | What it does | What you type |
|---------|-------------|---------------|
| `/implement` | Builds a feature end-to-end (implements, tests, reviews) | `/implement add a login page with email/password` |
| `/debug` | Finds and proves the root cause of a bug (does NOT fix it) | `/debug users get a 500 error on the profile page` |
| `/test` | Writes comprehensive tests for existing code | `/test the user authentication module` |
| `/code-review` | Reviews recent code changes for critical issues | `/code-review` or `/code-review the payments module` |
| `/observe` | Explores and explains how the codebase works | `/observe how does the authentication flow work` |
| `/hld` | Walks you through creating a High-Level Design document via Q&A | `/hld user-notifications` |

### How to Run a Command

1. Open Claude Code (CLI or IDE)
2. Type the command followed by your description
3. Press enter

Example:

```
/implement add an endpoint that returns the current user's profile
```

Claude will take it from there. It delegates work to the right agents automatically.

### What Happens Behind the Scenes

When you run `/implement add a logout button`, this is the chain of events:

1. The **implementer** agent writes the code
2. The **tester** agent writes tests for that code
3. The **code-reviewer** agent reviews everything for bugs and security issues
4. Claude reports back with a summary of what was built, tested, and flagged

You don't manage any of this. The command orchestrates the full workflow.

## Understanding Agents

Agents are specialist workers. Each one has a narrow focus and a specific set of tools it can use.

### Agent Roster

| Agent | Focus | What it can do |
|-------|-------|----------------|
| **implementer** | Writing code | Read, write, edit files. Run commands. |
| **tester** | Writing tests | Read, write, edit files. Run commands. |
| **code-reviewer** | Reviewing code quality | Read files. Run commands. Cannot edit. |
| **debugger** | Finding root causes | Read, write, edit files. Run commands. |
| **observer** | Understanding the codebase | Read files. Run commands. Cannot edit. |
| **docs-guide** | Reading project documentation | Read files only. |
| **docs-writer** | Updating project documentation | Read, write, edit files. |

### Agents You Never Need to Think About

Some agents get invoked automatically:

- **docs-guide** runs before any work starts, to load project patterns and conventions
- **code-reviewer** runs proactively after code changes
- **docs-writer** runs when the observer finds undocumented patterns

You don't trigger these. Claude handles it.

## Command vs Agent: When to Use What

**Always use commands.** That's it.

Commands exist so you don't have to think about which agent to call, in what order, or with what instructions. The command handles orchestration.

If you find yourself wanting to call an agent directly, use the corresponding command instead:

| You want to... | Use this command |
|----------------|-----------------|
| Build something | `/implement` |
| Find a bug | `/debug` |
| Write tests | `/test` |
| Review code | `/code-review` |
| Understand the codebase | `/observe` |
| Design a new feature | `/hld` |

## The `/hld` Command is Different

Unlike the other commands, `/hld` is interactive. It doesn't just run and return results. Instead:

1. It presents a 15-section checklist (problem statement, goals, architecture, etc.)
2. It asks you questions phase by phase
3. You answer, skip, or mark sections as TBD
4. When you say "done", it generates the full HLD document with ASCII diagrams

Run it like this:

```
/hld my-feature-name
```

It writes the final document to `docs/hlds/<feature-name>.md`.

## Project Structure Reference

```
.claude/
  commands/          <-- Slash commands (user-facing)
    implement.md
    debug.md
    test.md
    code-review.md
    observe.md
    hld.md
  agents/            <-- Agent definitions (system-facing)
    implementer.md
    debugger.md
    tester.md
    code-reviewer.md
    observer.md
    docs-guide.md
    docs-writer.md
docs/                <-- Project documentation (agents read/write this)
  system_patterns.md
  tech_context.md
  testing.md
  formatting.md
  conventions.md
```

## Documentation Files Explained

The `docs/` folder contains five files that agents read from and write to. These files are the project's knowledge base -- they tell agents how your codebase works so they don't have to guess.

**All five files start empty.** You populate them by running `/observe` against your codebase (see "How to Populate These Files" below).

### `system_patterns.md`

**Purpose:** Describes how the system is architected and why.

**What goes here:**
- High-level architecture (monolith, microservices, modular monolith, etc.)
- Design patterns in use (repository pattern, CQRS, event sourcing, etc.)
- Component relationships and boundaries (what talks to what)
- Key technical decisions and the reasoning behind them
- Critical implementation paths (e.g. "all API requests go through middleware X before reaching handlers")

**Example content:** "The app uses a layered architecture: routes → controllers → services → repositories. Services never access the database directly -- they always go through a repository."

### `tech_context.md`

**Purpose:** Lists the technologies, dependencies, and constraints of the project.

**What goes here:**
- Language and runtime versions (e.g. Python 3.12, Node 20)
- Frameworks and libraries in use (e.g. FastAPI, React, SQLAlchemy)
- Development setup instructions (how to install, run, build)
- External services and integrations (databases, message queues, third-party APIs)
- Technical constraints (e.g. "must run on Lambda with 512MB memory limit")
- Environment variables and configuration approach

**Example content:** "Backend: Python 3.12, FastAPI, SQLAlchemy 2.0, PostgreSQL 15. Frontend: TypeScript, Next.js 14, Tailwind CSS. Deployed on AWS ECS with RDS."

### `testing.md`

**Purpose:** Documents how tests are structured, written, and run in this project.

**What goes here:**
- Test framework and runner (pytest, jest, vitest, etc.)
- How to run tests (the actual commands)
- Test file location conventions (e.g. `tests/` folder vs co-located `__tests__/`)
- Fixture and factory patterns (what's available, how to use them)
- Mocking approach (what gets mocked, what doesn't)
- Database setup/teardown for tests
- Coverage requirements if any

**Example content:** "Run tests with `pytest -x`. Fixtures are in `conftest.py`. All external API calls must be mocked using `responses` library. Database tests use a transaction rollback strategy."

### `formatting.md`

**Purpose:** Defines code style and formatting rules so agents write code that matches the existing codebase.

**What goes here:**
- Formatter and linter tools (black, ruff, prettier, eslint, etc.)
- Configuration file locations (e.g. `pyproject.toml`, `.eslintrc`)
- Indentation style (tabs vs spaces, width)
- Line length limits
- Import organization rules (grouping, sorting)
- Language-specific style rules (trailing commas, quote style, etc.)
- How to run formatters (the actual commands)

**Example content:** "Format with `ruff format .` and lint with `ruff check --fix .`. Config in `pyproject.toml`. Line length: 120. Imports sorted by ruff with isort-compatible grouping."

### `conventions.md`

**Purpose:** Captures the naming conventions, file structure patterns, and standard practices specific to this project.

**What goes here:**
- File and folder naming conventions (e.g. kebab-case for files, PascalCase for components)
- Standard app/module structure (what files go where)
- Naming conventions for functions, classes, variables, database tables
- API endpoint naming patterns (e.g. RESTful conventions, URL structure)
- Pagination approach
- Error handling patterns
- How new modules/features should be structured

**Example content:** "All API endpoints follow REST conventions: `GET /api/v1/users`, `POST /api/v1/users`. Serializers live in `serializers.py` within each app. All model fields use snake_case. All endpoints return paginated responses using cursor-based pagination."

### How to Populate These Files

1. Drop this template into your actual project (copy `.claude/` and `docs/` folders)
2. Run a broad observe pass:
   ```
   /observe the full codebase - architecture, tech stack, patterns, conventions, and testing setup
   ```
3. The observer agent analyzes your code, then the docs-writer agent fills in the doc files
4. Review what was written -- agents can miss things or get details wrong
5. Run targeted follow-up passes on thin areas:
   ```
   /observe the testing patterns and test infrastructure
   /observe the file naming conventions and module structure
   ```
6. Manually edit anything the agents got wrong or missed

Once populated, every agent reads these files before doing work. The better your docs, the better your agents perform.

## Common Mistakes

**Giving vague instructions.** `/implement make it better` gives the agent nothing to work with. Be specific: `/implement add input validation to the registration form that rejects emails without an @ symbol`.

**Running `/debug` and expecting a fix.** The debug command only proves the root cause. It writes a failing test that demonstrates the bug. To fix it, run `/implement fix the null check in user.profile identified by the debug test`.

**Skipping `/observe` on an unfamiliar codebase.** Before building anything in a codebase you don't know, run `/observe` first. It maps out architecture, patterns, and conventions so your implementation aligns with existing code.
