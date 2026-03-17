---
name: feature-development
description: Full feature development lifecycle from requirements through shipping
argument-hint: "[feature description]"
---

Walk me through the full feature development workflow for: $ARGUMENTS

1. **Requirements**: Ask me clarifying questions about what I want to build. Cover scope, edge cases, authentication, data model, and error handling. Confirm acceptance criteria before proceeding.

2. **Implementation**: Analyze the codebase, create an implementation plan, and implement the feature following project conventions from CLAUDE.md. Write clean, minimal code.

3. **Code Review**: Review the code you just wrote as if you were a separate reviewer. Check for security vulnerabilities, breaking changes, naming conventions, and error handling. Fix any critical issues.

4. **Testing**: Write unit tests for all new functionality. Run the full test suite. Fix any failures.

5. **Ship**: Stage the changes, write a conventional commit message, and help me create a pull request with a clear description.

Pause after each step for my approval before continuing.
