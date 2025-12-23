---
description: Create a new Convex query function with proper validators and index usage
---

# Create Convex Query

Create a new Convex query function following best practices:

1. Use the new function syntax with `query` from "./_generated/server"
2. Include argument validators using `v` from "convex/values"
3. Include return validators
4. Use `withIndex()` instead of `filter()` for queries
5. Use proper TypeScript types with `Id<>` and `Doc<>`

The query should be for: $ARGUMENTS

Follow these rules:
- ALWAYS include `args` and `returns` validators
- NEVER use `.filter()` - use `.withIndex()` with a defined index
- Use `v.null()` if returning null is possible
- Add appropriate index to schema if needed
