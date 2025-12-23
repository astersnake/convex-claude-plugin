---
description: Create a new Convex action for external API calls or long-running tasks
---

# Create Convex Action

Create a new Convex action following best practices:

1. Add `"use node";` at the top of the file if using Node.js modules
2. Use `action` or `internalAction` from "./_generated/server"
3. Include argument validators using `v` from "convex/values"
4. Include return validators
5. Use `ctx.runQuery()` and `ctx.runMutation()` for database access

The action should be for: $ARGUMENTS

Follow these rules:
- Actions do NOT have direct database access - use `ctx.runQuery()` and `ctx.runMutation()`
- Add `"use node";` at the top for Node.js built-in modules
- Files with `"use node"` should ONLY contain actions
- Actions can run for up to 10 minutes (vs 1 second for queries/mutations)
- Use `internalAction` for functions that should only be called internally
- Handle errors gracefully with try/catch
