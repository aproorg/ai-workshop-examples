---
name: create-endpoint
description: Scaffold a new REST API endpoint with route, handler, validation, tests, and docs
argument-hint: "[METHOD /path - description]"
---

Create a new REST API endpoint: $ARGUMENTS

Ask the user for the HTTP method, path, and description if not provided.

Follow these steps:
1. Read CLAUDE.md to understand project conventions
2. Create the route definition in the appropriate routes file
3. Create the controller/handler with proper input validation
4. Add error handling following project patterns
5. Create unit tests covering success and error cases
6. Update API documentation if it exists
7. Run tests to verify everything passes
