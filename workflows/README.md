# Workflows in Claude Code

## Overview

Workflows are not a separate built-in feature in Claude Code. They are well-structured multi-step prompts that you store as a skill. While agents define a persona and skills define a single task, a workflow chains multiple phases together into a sequence -- often pausing between steps for your approval.

The power comes from the structure, not from any special runtime support. A workflow is just a skill that happens to describe a multi-phase process.

## How Workflows Work

A workflow prompt typically:

1. Breaks a large objective into numbered phases
2. Describes what Claude Code should do in each phase
3. Includes explicit pause points ("wait for my approval before continuing")
4. References project conventions from `CLAUDE.md`

To use a workflow, store it as a skill:

```
your-project/
  .claude/
    skills/
      feature-development/
        SKILL.md
```

Then invoke it like any other skill: `/feature-development`

## Included Example

### feature-development

A full feature development lifecycle stored as a skill. Five phases:

| Phase | What Happens |
|---|---|
| **Requirements** | Claude asks clarifying questions about scope, edge cases, and acceptance criteria |
| **Implementation** | Claude analyzes the codebase, plans, and implements following CLAUDE.md |
| **Code Review** | Claude reviews its own code for security, naming, and error handling |
| **Testing** | Claude writes unit tests and runs the full suite |
| **Ship** | Claude stages changes, writes a commit message, and helps create a PR |

The workflow pauses after each phase for your approval.

## Installation

Copy it as a skill into your project (e.g. the [demo project](https://github.com/aproorg/CopilotDemo)):

```bash
git clone https://github.com/aproorg/CopilotDemo.git
cd CopilotDemo
mkdir -p .claude/skills/feature-development
cp feature-development.md .claude/skills/feature-development/SKILL.md
```

## Tips for Custom Workflows

- **Add pause points**: Always include "wait for my approval before continuing" between phases
- **Keep it under 40 lines**: Long prompts lose focus; move details into separate skills
- **Reference other skills**: "For the code review phase, follow `/review-pr`"
- **Version control**: Store workflows in your repo so the team shares the same processes
