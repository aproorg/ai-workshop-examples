# Skills (Slash Commands) in Claude Code

## Overview

Skills are reusable task templates invoked as slash commands inside Claude Code. Each skill lives in its own directory under `.claude/skills/` and is defined in a file called `SKILL.md`. When you type `/skill-name`, Claude Code reads the file and executes the task it describes.

While agents define a persona that persists for a session, skills are action-oriented: they describe a specific task to accomplish, then they are done.

## Directory Structure

```
your-project/
  .claude/
    skills/
      create-endpoint/
        SKILL.md
      review-pr/
        SKILL.md
  CLAUDE.md
  src/
  ...
```

The directory name becomes the command name. `.claude/skills/create-endpoint/SKILL.md` is invoked with `/create-endpoint`.

## SKILL.md Format

Each `SKILL.md` file has two parts:

1. **YAML frontmatter** (between `---` markers) with metadata
2. **Markdown body** with the instructions Claude follows

### Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | No | Slash command name (defaults to directory name) |
| `description` | Recommended | When Claude should use this skill |
| `argument-hint` | No | Hint for expected arguments (e.g., `[issue-number]`) |
| `allowed-tools` | No | Tools Claude can use without asking (e.g., `Read, Grep, Glob`) |
| `model` | No | Model to use when skill is active |
| `context` | No | Set to `fork` to run in isolated subagent context |
| `disable-model-invocation` | No | Set to `true` to prevent Claude from auto-invoking |

### Argument Placeholders

| Placeholder | Description |
|-------------|-------------|
| `$ARGUMENTS` | All arguments passed when invoking the skill |
| `$0`, `$1`, ... | Individual arguments by index |

If `$ARGUMENTS` is not present in the body, Claude Code automatically appends the arguments at the end.

## Included Examples

### create-endpoint

A REST API endpoint scaffolding skill. Copy `.claude/skills/create-endpoint/SKILL.md` into your project to use it.

Usage:

```
> /create-endpoint POST /api/users/invite - accepts email and role, sends invitation
```

### review-pr

A pull request review skill that produces a structured review with quality rating.

Usage:

```
> /review-pr
Review PR #42 against main.
```

## Installation

Copy the skill directories into your project:

```bash
# From the root of your project
mkdir -p .claude/skills

# Copy a skill (the whole directory)
cp -r create-endpoint .claude/skills/
cp -r review-pr       .claude/skills/
```

No additional setup is needed. The commands become available immediately.

To try them out, clone the [demo project](https://github.com/aproorg/CopilotDemo) and copy the skills into it.

## Scope Levels

Skills can be defined at multiple levels:

| Level | Location | Applies to |
|-------|----------|------------|
| Project | `.claude/skills/` | This project only |
| Personal | `~/.claude/skills/` | All your projects |

## Customization Tips

- **Tailor to your stack**: Edit the SKILL.md to reference your frameworks and patterns
- **Keep skills focused**: A good skill does one thing well
- **Version control**: Skill files live in your repo, so the team shares the same commands
