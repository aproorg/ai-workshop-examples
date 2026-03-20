# MCP (Model Context Protocol) -- Reference Guide

## What Is MCP?

The Model Context Protocol (MCP) is an open standard that defines how AI coding assistants like Claude Code communicate with external tools and data sources. MCP servers are standalone processes that run alongside Claude Code and expose capabilities -- tools, resources, and prompt templates -- through a structured JSON-RPC interface over stdio (standard input/output).

In practical terms, an MCP server is a small program that Claude Code launches in the background. Claude Code sends it requests ("list open pull requests", "run this SQL query", "read that file"), and the server responds with results. This lets Claude Code interact with systems it has no built-in support for: GitHub, databases, file directories outside the working tree, internal APIs, and more.

MCP is not specific to Claude Code. It is a general protocol that any AI tool can implement, which means servers you configure today can work with other MCP-compatible tools in the future.

---

## How MCP Works with Claude Code

When Claude Code starts, it reads your configuration file and launches each configured MCP server as a child process. Communication follows this pattern:

```
Claude Code                        MCP Server (child process)
    |                                  |
    |-- spawn process ---------------->|
    |                                  |
    |-- JSON-RPC request (stdin) ----->|
    |                                  |  (process request)
    |<-- JSON-RPC response (stdout) ---|
    |                                  |
    |-- JSON-RPC request (stdin) ----->|
    |                                  |  (process request)
    |<-- JSON-RPC response (stdout) ---|
    |                                  |
    |-- terminate process ------------>|
```

Key points:

- MCP servers are **external processes**, not internal plugins. They run in their own Node.js (or other runtime) process.
- Communication uses **stdio** -- JSON-RPC messages over stdin/stdout. Servers log diagnostic output to stderr.
- Claude Code manages the **lifecycle** -- starting servers when it launches and stopping them when it exits.
- Each server exposes **tools** (functions Claude can call), **resources** (data for context), and **prompts** (reusable templates).
- Claude Code decides **when to call** an MCP tool based on your natural language requests.

---

## Configuring MCP Servers

MCP servers are configured in `.claude/settings.json`. You can place this file at two levels:

| Location | Scope |
|----------|-------|
| `<project-root>/.claude/settings.json` | Project-level -- applies only when Claude Code runs in that project directory |
| `~/.claude/settings.json` | Global -- applies to all Claude Code sessions for your user account |

Each MCP server entry lives inside the `mcpServers` object and requires at minimum a `command` and `args` array:

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "@scope/package-name"],
      "env": {
        "OPTIONAL_VAR": "value"
      }
    }
  }
}
```

| Field | Description |
|-------|-------------|
| `command` | The executable to run. Typically `npx` (to fetch and run an npm package) or `node` (for local builds). |
| `args` | Arguments passed to the command. For `npx`, include `-y` to auto-confirm the install, followed by the package name and any server-specific arguments. |
| `env` | Optional environment variables passed to the server process. Use this for API tokens and configuration values. |

---

## Common MCP Servers

### GitHub MCP Server

**Package:** `@modelcontextprotocol/server-github`

Connects Claude Code to the GitHub API for repository, issue, and pull request management. With this server active, you can ask Claude Code to list PRs, create issues, review diffs, search repositories, and manage branches -- all without leaving the terminal.

**Configuration:**

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

**Requirements:**
- Node.js 18+
- A GitHub personal access token with appropriate scopes (`repo`, `read:org`, etc.)

**Example prompts once configured:**
- "List my open PRs in this repository."
- "Create an issue titled 'Fix login validation' with the label 'bug'."
- "Review PR #42 and summarize the changes."

---

### Filesystem MCP Server

**Package:** `@modelcontextprotocol/server-filesystem`

Gives Claude Code structured read/write access to specific directories on your machine. While Claude Code already has file tools for the current working directory, this server lets you reach directories outside the project -- useful for cross-project analysis, shared configuration, and data directories.

**Configuration (single directory):**

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/home/user/projects"
      ]
    }
  }
}
```

**Configuration (multiple directories):**

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/home/user/projects",
        "/home/user/documents",
        "/home/user/data"
      ]
    }
  }
}
```

**Configuration (Windows paths):**

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "C:\\Users\\username\\Documents\\work",
        "C:\\Users\\username\\Documents\\data"
      ]
    }
  }
}
```

**Requirements:**
- Node.js 18+

**Security note:** Only directories you explicitly list are accessible. The server denies requests targeting paths outside the allowed set, including via symlinks.

---

### Database MCP Servers

**Packages:**
- `@modelcontextprotocol/server-postgres` -- for PostgreSQL databases
- `@modelcontextprotocol/server-sqlite` -- for SQLite files

Connect Claude Code to relational databases for schema exploration, query execution, data analysis, and migration generation.

**PostgreSQL configuration:**

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

**SQLite configuration:**

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

**Requirements:**
- Node.js 18+
- A running PostgreSQL instance or an existing SQLite file

**Security note:** Database MCP servers provide read-write access by default. For safety, create a dedicated read-only database user for exploratory and analytical tasks.

---

### Running Multiple Servers Together

You can configure all servers simultaneously in a single settings file:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "ghp_your_token" }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/home/user/projects"]
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://user:pass@localhost/dev"]
    }
  }
}
```

Claude Code launches each server as a separate process and selects the appropriate one based on your prompt context.

---

## Building Your Own Custom MCP Server

When the available servers do not cover your use case -- proprietary APIs, internal services, domain-specific logic -- you can build a custom MCP server.

### Overview

A custom MCP server is a program (typically TypeScript/Node.js) that:

1. Reads JSON-RPC messages from stdin
2. Processes requests (calling your APIs, querying your systems)
3. Writes JSON-RPC responses to stdout
4. Logs diagnostic output to stderr

### The TypeScript SDK

The official `@modelcontextprotocol/sdk` package handles protocol negotiation, message parsing, and response formatting:

```bash
npm install @modelcontextprotocol/sdk zod
```

### Minimal Example

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({
  name: "my-custom-server",
  version: "1.0.0",
});

server.tool(
  "lookup_user",
  "Look up a user by email address.",
  {
    email: z.string().email().describe("The user's email address"),
  },
  async ({ email }) => {
    // Replace with your actual data source
    const user = { email, name: "Jane Smith", role: "Engineer" };
    return {
      content: [{ type: "text" as const, text: JSON.stringify(user, null, 2) }],
    };
  }
);

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("Custom MCP server running on stdio");
}

main().catch((error) => {
  console.error("Server failed to start:", error);
  process.exit(1);
});
```

### Registering Your Server

Add it to `.claude/settings.json` pointing to the compiled JavaScript:

```json
{
  "mcpServers": {
    "my-custom-server": {
      "command": "node",
      "args": ["/absolute/path/to/dist/index.js"],
      "env": {
        "API_KEY": "your_api_key_if_needed"
      }
    }
  }
}
```

Or, if published to npm:

```json
{
  "mcpServers": {
    "my-custom-server": {
      "command": "npx",
      "args": ["-y", "@yourorg/mcp-server-custom"]
    }
  }
}
```

---

## The /mcp Command

Claude Code provides a built-in `/mcp` command for managing MCP server connections during a session:

```
/mcp
```

This command displays the status of all configured MCP servers -- which are connected, which failed to start, and what tools each server exposes. Use it to:

- Verify that your servers started correctly after editing `settings.json`
- Check which tools are available in the current session
- Diagnose connection issues without restarting Claude Code

---

## Links and Resources

| Resource | URL |
|----------|-----|
| MCP Specification | [https://modelcontextprotocol.io](https://modelcontextprotocol.io) |
| Official MCP Servers Repository | [https://github.com/modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers) |
| TypeScript SDK | [https://github.com/modelcontextprotocol/typescript-sdk](https://github.com/modelcontextprotocol/typescript-sdk) |
| Claude Code Documentation | [https://docs.anthropic.com/en/docs/claude-code](https://docs.anthropic.com/en/docs/claude-code) |
| Community MCP Servers | [https://github.com/modelcontextprotocol/servers#community-servers](https://github.com/modelcontextprotocol/servers#community-servers) |

---

## Security Considerations

MCP servers can access external systems on your behalf. Follow these guidelines to stay safe:

### Token and Credential Management

- Never commit `.claude/settings.json` to version control if it contains API tokens or passwords. Add it to `.gitignore`.
- Use environment variables in your shell profile and reference them in the configuration rather than hardcoding secrets.
- Rotate tokens periodically, especially shared seminar tokens.

### Principle of Least Privilege

- Grant the minimum permissions each server needs. For GitHub, scope your token to only the repositories and actions required. For databases, use a read-only user when you only need to query data.
- For the filesystem server, allow access only to specific directories -- never your entire home directory or system root.

### Sensitive Directories to Avoid

Never grant MCP servers access to directories containing:

- SSH keys (`~/.ssh`)
- Cloud credentials (`~/.aws`, `~/.gcloud`, `~/.azure`)
- Password managers or keychains
- System configuration directories (`/etc`, `C:\Windows\System32`)
- Private keys or certificates

### Review Before Execution

Claude Code may present SQL queries, file write operations, or API calls before executing them through MCP servers. Review any destructive operations (DELETE, DROP, file overwrites) carefully before approving.

### Network Security

For database connections, use SSL (`?sslmode=require`) and restrict access by IP address. Avoid exposing database ports to the public internet.

### First-Run Package Downloads

When using `npx`, the first invocation downloads the MCP server package from npm. Subsequent runs use the cached version. If you are in a restricted network environment, pre-cache packages or use local builds instead.
