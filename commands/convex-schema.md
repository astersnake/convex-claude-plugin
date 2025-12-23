---
description: Create or update Convex schema with proper indexes and validators
---

# Create/Update Convex Schema

Update the Convex schema in `convex/schema.ts` following best practices:

1. Use `defineSchema` and `defineTable` from "convex/server"
2. Use proper validators from `v` in "convex/values"
3. Define indexes for all query patterns
4. Include `authTables` if using Convex Auth

Schema changes for: $ARGUMENTS

Follow these rules:
- System fields `_id` and `_creationTime` are automatic - don't define them
- NEVER create `.index("by_creation_time", ["_creationTime"])` - it's built-in
- NEVER include `_creationTime` as the last field in an index - it's automatic
- Index names should describe all fields: `by_field1_and_field2`
- Create separate indexes for different query patterns
- Use `v.optional()` for optional fields
- Use `v.union()` for discriminated unions with `v.literal()`

Example schema structure:
```typescript
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  tableName: defineTable({
    field1: v.string(),
    field2: v.optional(v.number()),
  })
    .index("by_field1", ["field1"]),
});
```
