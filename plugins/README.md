# Plugins in Claude Code -- Reference Guide

## What Are Plugins in Claude Code?

Plugins extend Claude Code beyond its built-in capabilities. They integrate external tools, services, and workflows directly into your Claude Code session, allowing Claude to perform actions -- managing GitHub issues, querying databases, reading files across directories -- that it cannot do on its own.

Plugins are not monolithic add-ons. They are lightweight integrations that Claude Code launches and manages automatically based on your configuration.

---

## Two Extension Mechanisms

Claude Code supports two distinct approaches to plugins:

### 1. MCP Servers

MCP (Model Context Protocol) servers are the primary plugin system. Each MCP server is a standalone process that communicates with Claude Code over stdio using the JSON-RPC protocol. Servers expose **tools** (functions Claude can call), **resources** (contextual data), and **prompts** (reusable templates).

MCP servers can connect Claude Code to virtually any external system: APIs, databases, file systems, CI/CD pipelines, internal services, and more.

**Characteristics:**
- Run as separate processes alongside Claude Code
- Communicate over stdin/stdout using JSON-RPC
- Claude Code manages their lifecycle (start, stop, restart)
- Configured in `.claude/settings.json`
- Can be official packages, community packages, or custom-built

### 2. Skills (Slash Commands)

Skills are reusable prompt templates stored as `SKILL.md` files. When you type `/` followed by the skill name in Claude Code, it loads the template and executes it. This is the simplest form of plugin -- no external process, no API, just a standardized prompt.

**Characteristics:**
- Stored as `SKILL.md` files in `.claude/skills/<name>/` at the project root
- Support YAML frontmatter for `name`, `description`, `argument-hint`, etc.
- Invoked with the `/` prefix (e.g., `/review`, `/commit`)
- Can be version-controlled and shared across a team
- No installation or external dependencies required
- Ideal for codifying team standards and repeatable workflows

---

## How Plugins Are Configured

### MCP Servers

MCP servers are configured in `.claude/settings.json` inside the `mcpServers` object:

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "@scope/package-name"],
      "env": {
        "API_TOKEN": "your_token_here"
      }
    }
  }
}
```

| Scope | File Location |
|-------|---------------|
| Project-level | `<project-root>/.claude/settings.json` |
| Global (all projects) | `~/.claude/settings.json` |

### Skills

Skills are configured by placing `SKILL.md` files in named directories under `.claude/skills/`:

```
<project-root>/
  .claude/
    skills/
      review/
        SKILL.md
      commit/
        SKILL.md
      test-plan/
        SKILL.md
```

Each directory becomes a command: `/review`, `/commit`, `/test-plan`.

---

## The /mcp Command

Claude Code provides a built-in `/mcp` command for managing MCP server connections:

```
/mcp
```

Use this command to:

- View the status of all configured MCP servers (connected, disconnected, error)
- See which tools each server exposes
- Diagnose startup failures and connection issues
- Verify configuration changes without restarting Claude Code

This is the first thing to check when an MCP server is not behaving as expected.

---

## Recommended Plugins to Explore

The following plugins cover the most common needs in day-to-day development.

---

### GitHub MCP Server

**Package:** `@modelcontextprotocol/server-github`

Connects Claude Code to the GitHub API. Manage issues, pull requests, code reviews, branches, and repository search without leaving the terminal.

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_your_token_here"
      }
    }
  }
}
```

**Requirements:** Node.js 18+, GitHub personal access token.

**Example use:** "Review PR #42 and summarize the changes."

---

### Filesystem MCP Server

**Package:** `@modelcontextprotocol/server-filesystem`

Provides Claude Code with structured read/write access to directories outside the current project. Useful for cross-project analysis, shared configuration, and data management.

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/home/user/projects",
        "/home/user/data"
      ]
    }
  }
}
```

**Requirements:** Node.js 18+.

**Example use:** "Read the Dockerfile from /home/user/projects/other-app and adapt it for our project."

---

### Database MCP Servers

**Packages:**
- `@modelcontextprotocol/server-postgres` -- PostgreSQL
- `@modelcontextprotocol/server-sqlite` -- SQLite

Connect Claude Code to relational databases for schema exploration, query execution, data analysis, and migration generation.

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://user:pass@localhost:5432/dbname"
      ]
    }
  }
}
```

```json
{
  "mcpServers": {
    "sqlite": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-sqlite",
        "/path/to/database.db"
      ]
    }
  }
}
```

**Requirements:** Node.js 18+, a running database instance or SQLite file.

**Example use:** "Show me all tables in the database and their column definitions."

---

### Custom Skills

Skills codify team workflows into reusable prompt templates. Create them by adding `SKILL.md` files in `.claude/skills/<name>/`.

**Example: `/review` skill**

Create `.claude/skills/review/SKILL.md`:

```markdown
---
name: review
description: Review current git diff for quality and security
---

Review the current git diff for the following:

1. Code correctness -- identify bugs, logic errors, or edge cases
2. Performance -- flag operations that could be slow at scale
3. Security -- check for injection vulnerabilities and exposed secrets
4. Style -- verify consistency with project coding standards
5. Tests -- assess whether changes have adequate test coverage

Provide findings organized by category with specific file and line references.
```

**Example: `/commit` skill**

Create `.claude/skills/commit/SKILL.md`:

```markdown
---
name: commit
description: Generate a conventional commit message from staged changes
---

Look at the current staged changes (git diff --cached) and write a conventional commit message:

type(scope): subject

body (optional, wrap at 72 characters)

Use one of: feat, fix, docs, style, refactor, perf, test, chore.
Infer the scope from the files changed.
Keep the subject under 50 characters.
```

**Example: `/test-plan` skill**

Create `.claude/skills/test-plan/SKILL.md`:

```markdown
---
name: test-plan
description: Generate a test plan for the current project
---

Analyze the current project and generate a test plan covering:

1. Unit tests for core business logic
2. Integration tests for API endpoints
3. Edge cases and error handling scenarios
4. Performance-sensitive paths

Output the plan as a Markdown checklist grouped by test category.
```

Once created, type `/review`, `/commit`, or `/test-plan` in Claude Code to execute them.

---

## Discovering More Plugins

The MCP ecosystem is actively growing. To find additional servers and integrations:

| Source | URL |
|--------|-----|
| Official MCP Servers | [https://github.com/modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers) |
| Community Servers | [https://github.com/modelcontextprotocol/servers#community-servers](https://github.com/modelcontextprotocol/servers#community-servers) |
| MCP Specification | [https://modelcontextprotocol.io](https://modelcontextprotocol.io) |
| TypeScript SDK (build your own) | [https://github.com/modelcontextprotocol/typescript-sdk](https://github.com/modelcontextprotocol/typescript-sdk) |
| Claude Code Documentation | [https://docs.anthropic.com/en/docs/claude-code](https://docs.anthropic.com/en/docs/claude-code) |

The community servers section is particularly valuable -- contributors have built MCP servers for Slack, Jira, Notion, Docker, Kubernetes, and many other tools.

---

## Security Checklist for Plugins

Before enabling plugins, especially in a team or production-adjacent environment, review the following:

- [ ] **API tokens are stored securely.** Do not commit tokens to version control. Use environment variables or secret managers.
- [ ] **Database connections use read-only credentials** where the workflow only requires querying data.
- [ ] **Filesystem access is scoped tightly.** Only allow directories that Claude Code actually needs. Never expose `~/.ssh`, `~/.aws`, credential stores, or system directories.
- [ ] **`.claude/settings.json` is in `.gitignore`** if it contains any secrets, tokens, or passwords.
- [ ] **Team members understand what each plugin can access.** Document which servers are configured and what permissions they hold.
- [ ] **Tokens are rotated periodically.** Do not use long-lived tokens with broad scopes when short-lived or narrowly-scoped alternatives exist.
- [ ] **Destructive operations are reviewed.** SQL mutations (INSERT, UPDATE, DELETE), file writes, and GitHub actions (merge, close) should be reviewed before Claude Code executes them.
- [ ] **Third-party MCP servers are audited.** Before using a community MCP server, review its source code to understand what it does and what data it accesses.

---

## Performance Considerations

Each MCP server runs as a separate process. Keep the following in mind:

### Process Overhead

Running three to five MCP servers simultaneously has negligible impact on most development machines. Each server typically consumes minimal CPU when idle and a small amount of memory (roughly 30-80 MB per Node.js process).

### First-Run Latency

When using `npx`, the first invocation downloads the MCP server package from npm. This can take several seconds depending on your network speed. Subsequent runs use the cached package and start much faster. To eliminate first-run latency, pre-cache packages:

```bash
npx -y @modelcontextprotocol/server-github --help
npx -y @modelcontextprotocol/server-filesystem --help
```

### Database Query Performance

For database MCP servers, query execution time depends on the database itself. Slow queries are not a plugin issue -- they are a database performance issue. Consider setting a statement timeout on your database user:

```sql
-- PostgreSQL example
ALTER ROLE claude_readonly SET statement_timeout = '30s';
```

### When to Disable Servers

If you are not using a particular MCP server for your current task, you can remove or comment out its entry in `settings.json` to reduce resource usage. This is rarely necessary but can help on resource-constrained machines.

### Monitoring

Use the `/mcp` command to check if servers are running and responsive. If a server shows as disconnected, it may have crashed -- check stderr output or restart Claude Code.
