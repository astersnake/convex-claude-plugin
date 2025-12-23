---
name: convex-development
description: Complete Convex development guidelines and best practices for building production-ready full-stack applications
globs:
  - "**/*.ts"
  - "**/*.tsx"
  - "convex/**/*"
triggers:
  - convex
  - database
  - query
  - mutation
  - action
  - schema
  - realtime
---

# Convex Development Skill

Complete guidelines and best practices for building Convex projects, including database schema design, queries, mutations, actions, authentication, React integration, and AI capabilities.

---

# CRITICAL RULES

## Function Syntax (REQUIRED)

ALWAYS use the new function syntax for Convex functions:

```typescript
import { query } from "./_generated/server";
import { v } from "convex/values";

export const myFunction = query({
  args: {
    // Arguments with validators
  },
  returns: v.null(), // Return validator
  handler: async (ctx, args) => {
    // Function body
  },
});
```

## Validators

ALWAYS include argument AND return validators for ALL Convex functions:
- `query`, `internalQuery`
- `mutation`, `internalMutation`
- `action`, `internalAction`

If a function doesn't return anything, use `returns: v.null()`.

## Index Rules

- Do NOT use `filter()` in queries - use `withIndex()` instead
- Do NOT include `_creationTime` in index definitions - it's automatic
- Do NOT create `.index("by_creation_time", ["_creationTime"])` - it's built-in
- Index names should describe all fields: `.index("by_field1_and_field2", ["field1", "field2"])`

## Actions

- Add `"use node";` at the top of files using Node.js modules
- Files with `"use node"` should ONLY contain actions, not queries or mutations
- Actions do NOT have access to `ctx.db`

## Scheduling

- Auth does NOT propagate to scheduled jobs - pass userId explicitly
- Don't schedule more than once every 10 seconds

---

# Valid Convex Types

| Type | TS/JS | Validator | Notes |
|------|-------|-----------|-------|
| Id | string | `v.id(tableName)` | Document ID |
| Null | null | `v.null()` | Use instead of undefined |
| Int64 | bigint | `v.int64()` | NOT v.bigint() |
| Float64 | number | `v.number()` | |
| Boolean | boolean | `v.boolean()` | |
| String | string | `v.string()` | Max 1MB UTF-8 |
| Bytes | ArrayBuffer | `v.bytes()` | Max 1MB |
| Array | Array | `v.array(values)` | Max 8192 elements |
| Object | Object | `v.object({...})` | Max 1024 entries |
| Record | Record | `v.record(keys, values)` | Dynamic keys |

**NOT SUPPORTED:** `v.map()`, `v.set()`, `v.bigint()` (deprecated)

---

# Function Types

## Public Functions (API)
```typescript
import { query, mutation, action } from "./_generated/server";
import { api } from "./_generated/api";
// Reference: api.filename.functionName
```

## Internal Functions (Private)
```typescript
import { internalQuery, internalMutation, internalAction } from "./_generated/server";
import { internal } from "./_generated/api";
// Reference: internal.filename.functionName
```

## Calling Functions

```typescript
// From mutation or action
await ctx.runQuery(api.users.get, { id: userId });
await ctx.runMutation(internal.users.update, { id: userId, name });

// From action only
await ctx.runAction(internal.ai.generate, { prompt });
```

---

# Schema Definition

```typescript
// convex/schema.ts
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  users: defineTable({
    name: v.string(),
    email: v.string(),
  })
    .index("by_email", ["email"]),

  messages: defineTable({
    userId: v.id("users"),
    content: v.string(),
    channelId: v.id("channels"),
  })
    .index("by_channel", ["channelId"])
    .index("by_user_and_channel", ["userId", "channelId"]),
});
```

## System Fields (Automatic)
- `_id`: `v.id(tableName)` - Document ID
- `_creationTime`: `v.number()` - Creation timestamp

---

# Query Patterns

## Correct Query Usage
```typescript
// Using index (CORRECT)
const messages = await ctx.db
  .query("messages")
  .withIndex("by_channel", (q) => q.eq("channelId", channelId))
  .order("desc")
  .take(10);

// WRONG - Never use filter()
const messages = await ctx.db
  .query("messages")
  .filter((q) => q.eq(q.field("channelId"), channelId)) // BAD!
  .collect();
```

## Delete Pattern
```typescript
// Convex doesn't support .delete() on queries
const items = await ctx.db
  .query("items")
  .withIndex("by_user", (q) => q.eq("userId", userId))
  .collect();

for (const item of items) {
  await ctx.db.delete(item._id);
}
```

---

# Mutations

```typescript
// Insert
const id = await ctx.db.insert("users", { name: "Alice", email: "alice@example.com" });

// Patch (partial update)
await ctx.db.patch(userId, { name: "Bob" });

// Replace (full replacement)
await ctx.db.replace(userId, { name: "Bob", email: "bob@example.com" });

// Delete
await ctx.db.delete(userId);
```

---

# Actions with External APIs

```typescript
// convex/ai.ts
"use node";

import { action } from "./_generated/server";
import { v } from "convex/values";
import OpenAI from "openai";

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export const generate = action({
  args: { prompt: v.string() },
  returns: v.string(),
  handler: async (ctx, args) => {
    const response = await openai.chat.completions.create({
      model: "gpt-4o-mini",
      messages: [{ role: "user", content: args.prompt }],
    });
    return response.choices[0].message.content ?? "";
  },
});
```

---

# Scheduling

```typescript
// Schedule immediately (async)
await ctx.scheduler.runAfter(0, internal.tasks.process, { taskId });

// Schedule with delay
await ctx.scheduler.runAfter(60000, internal.tasks.cleanup, {}); // 1 minute

// IMPORTANT: Pass userId explicitly - auth doesn't propagate
await ctx.scheduler.runAfter(0, internal.tasks.process, {
  userId: currentUserId, // Must pass this!
  taskId,
});
```

---

# Cron Jobs

```typescript
// convex/crons.ts
import { cronJobs } from "convex/server";
import { internal } from "./_generated/api";

const crons = cronJobs();

crons.interval("cleanup", { hours: 1 }, internal.tasks.cleanup, {});
crons.cron("daily-report", "0 9 * * *", internal.reports.generate, {});

export default crons;
```

---

# Authentication (Convex Auth)

```typescript
import { getAuthUserId } from "@convex-dev/auth/server";
import { query, mutation } from "./_generated/server";

export const currentUser = query({
  args: {},
  handler: async (ctx) => {
    const userId = await getAuthUserId(ctx);
    if (!userId) return null;
    return await ctx.db.get(userId);
  },
});

export const protectedAction = mutation({
  args: { content: v.string() },
  returns: v.null(),
  handler: async (ctx, args) => {
    const userId = await getAuthUserId(ctx);
    if (!userId) throw new Error("Not authenticated");

    await ctx.db.insert("posts", {
      authorId: userId,
      content: args.content,
    });
    return null;
  },
});
```

---

# File Storage

```typescript
// Generate upload URL
export const generateUploadUrl = mutation({
  args: {},
  handler: async (ctx) => {
    return await ctx.storage.generateUploadUrl();
  },
});

// Get file URL
export const getFileUrl = query({
  args: { storageId: v.id("_storage") },
  returns: v.union(v.string(), v.null()),
  handler: async (ctx, args) => {
    return await ctx.storage.getUrl(args.storageId);
  },
});

// Save file reference
export const saveFile = mutation({
  args: { storageId: v.id("_storage"), fileName: v.string() },
  returns: v.null(),
  handler: async (ctx, args) => {
    await ctx.db.insert("files", {
      storageId: args.storageId,
      fileName: args.fileName,
    });
    return null;
  },
});
```

---

# React Client

## Basic Hooks
```tsx
import { useQuery, useMutation, useAction } from "convex/react";
import { api } from "../convex/_generated/api";

function Component() {
  // Query (reactive, auto-updates)
  const messages = useQuery(api.messages.list, { channelId }) || [];

  // Mutation
  const sendMessage = useMutation(api.messages.send);

  // Action
  const generateAI = useAction(api.ai.generate);
}
```

## Conditional Queries (CORRECT)
```tsx
// WRONG - Never call hooks conditionally
const data = userId ? useQuery(api.users.get, { userId }) : null;

// CORRECT - Use "skip"
const data = useQuery(api.users.get, userId ? { userId } : "skip");
```

## Loading States
```tsx
const data = useQuery(api.data.get);

if (data === undefined) return <Loading />; // Loading
if (data === null) return <NotFound />;     // Not found
return <Content data={data} />;             // Data ready
```

---

# System Limits

| Limit | Value |
|-------|-------|
| Function args/returns | 8 MiB |
| Array elements | 8192 |
| Object entries | 1024 |
| Document size | 1 MiB |
| Query/Mutation timeout | 1 second |
| Action timeout | 10 minutes |
| DB read per query | 8 MiB / 16384 docs |
| DB write per mutation | 8 MiB / 8192 docs |

---

# Full Text Search

```typescript
// Schema
defineTable({
  body: v.string(),
  channel: v.string(),
}).searchIndex("search_body", {
  searchField: "body",
  filterFields: ["channel"],
})

// Query
const results = await ctx.db
  .query("messages")
  .withSearchIndex("search_body", (q) =>
    q.search("body", "hello").eq("channel", "#general")
  )
  .take(10);
```

---

# Pagination

```typescript
import { paginationOptsValidator } from "convex/server";

export const list = query({
  args: { paginationOpts: paginationOptsValidator },
  handler: async (ctx, args) => {
    return await ctx.db
      .query("messages")
      .order("desc")
      .paginate(args.paginationOpts);
  },
});
// Returns: { page, isDone, continueCursor }
```

---

# HTTP Endpoints

```typescript
// convex/http.ts
import { httpRouter } from "convex/server";
import { httpAction } from "./_generated/server";

const http = httpRouter();

http.route({
  path: "/webhook",
  method: "POST",
  handler: httpAction(async (ctx, req) => {
    const body = await req.json();
    return new Response(JSON.stringify({ ok: true }), {
      status: 200,
      headers: { "Content-Type": "application/json" },
    });
  }),
});

export default http;
```

---

# TypeScript Best Practices

```typescript
import { Id, Doc } from "./_generated/dataModel";

// Use Id<> for document IDs
function getUser(userId: Id<"users">): Promise<Doc<"users"> | null> {
  // ...
}

// Record with Id keys
const map: Record<Id<"users">, string> = {};

// Arrays with explicit types
const items: Array<{ id: Id<"items">; name: string }> = [];
```
