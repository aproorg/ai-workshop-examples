# Agents in Claude Code

## Overview

Agents are reusable personas that shape how Claude Code approaches a task. Each agent lives in its own directory under `.claude/agents/` and is defined in a Markdown file. When you invoke an agent, Claude Code adopts the role and workflow defined inside it.

Think of agents as specialists you can call on: one might be an expert at implementing features, another at reviewing code.

## Directory Structure

```
your-project/
  .claude/
    agents/
      feature-request/
        feature-request.md
      code-reviewer/
        code-reviewer.md
  CLAUDE.md
  src/
  ...
```

Each agent gets its own directory. The file inside contains YAML frontmatter and a Markdown body that becomes the agent's system prompt.

## Agent File Format

Each agent file has two parts:

1. **YAML frontmatter** (between `---` markers) with metadata
2. **Markdown body** with the agent's system prompt and instructions

### Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Unique identifier (lowercase, hyphens) |
| `description` | Yes | When Claude should delegate to this agent |
| `tools` | No | Available tools (inherits all if omitted) |
| `model` | No | `sonnet`, `opus`, `haiku`, or `inherit` (default) |
| `maxTurns` | No | Maximum agentic turns before stopping |
| `permissionMode` | No | `default`, `acceptEdits`, `plan`, etc. |

## Included Examples

### feature-request

A feature implementation specialist. Give it a feature request and it will analyze the codebase, plan, implement, test, and summarize.

### code-reviewer

A senior code reviewer. Point it at a diff or branch and it will produce a structured review organized into Critical, Warnings, Suggestions, and Positive Observations.

## Installation

Copy the agent directories into your project:

```bash
# From the root of your project
mkdir -p .claude/agents

# Copy an agent (the whole directory)
cp -r feature-request .claude/agents/
cp -r code-reviewer   .claude/agents/
```

The agents become available immediately.

To try them out, clone the [demo project](https://github.com/aproorg/CopilotDemo) and copy the agents into it.

## How Agents are Invoked

Agents can be triggered in several ways:

1. **Automatic**: Claude delegates to the agent when its `description` matches the current task
2. **Explicit**: Ask Claude to "use the code-reviewer agent" in conversation
3. **CLI**: Run `claude --agent code-reviewer` from the terminal

## Scope Levels

| Level | Location | Applies to |
|-------|----------|------------|
| Project | `.claude/agents/` | This project only |
| Personal | `~/.claude/agents/` | All your projects |

## Customization Tips

- **Add domain knowledge**: Include your stack details in the system prompt
- **Combine with CLAUDE.md**: Put universal conventions in CLAUDE.md, role-specific instructions in the agent file
- **Keep it focused**: Each agent should have a clear specialty
