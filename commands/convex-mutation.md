---
description: Create a new Convex mutation function with proper validators and auth
---

# Create Convex Mutation

Create a new Convex mutation function following best practices:

1. Use the new function syntax with `mutation` from "./_generated/server"
2. Include argument validators using `v` from "convex/values"
3. Include return validators (use `v.null()` if no return)
4. Add authentication check if needed using `getAuthUserId`
5. Use proper TypeScript types with `Id<>` and `Doc<>`

The mutation should be for: $ARGUMENTS

Follow these rules:
- ALWAYS include `args` and `returns` validators
- Check authentication with `getAuthUserId` for protected mutations
- Use `ctx.db.insert()`, `ctx.db.patch()`, `ctx.db.replace()`, or `ctx.db.delete()`
- Return `null` explicitly if the function doesn't return a value
- Validate that referenced documents exist before operations
