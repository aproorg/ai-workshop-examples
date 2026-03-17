---
name: review-pr
description: Review a pull request or branch for code quality, security, and test coverage
argument-hint: "[branch or PR number]"
---

Review the pull request or branch: $ARGUMENTS

Steps:
1. Fetch the diff (git diff main...branch or use GitHub MCP if available)
2. Analyze all changed files
3. Check for:
   - Security vulnerabilities
   - Breaking changes
   - Test coverage
   - Documentation updates needed
   - Code style consistency
4. Provide a structured review summary
5. Rate overall quality: Approve / Request Changes / Needs Discussion
