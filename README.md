# Convex Development Plugin for Claude Code

A comprehensive Claude Code plugin that provides Convex development guidelines, best practices, and slash commands based on [Chef's](https://github.com/get-convex/chef) production patterns.

## Features

- **Skill**: Complete Convex development guidelines automatically applied when working with `.ts`/`.tsx` files in Convex projects
- **Slash Commands**: Quick commands for creating queries, mutations, actions, and schemas
- **Best Practices**: Enforces correct patterns from the official Chef project

## Installation

### Local Installation (Development)

```bash
claude --plugin-dir /path/to/convex-claude-plugin
```

### From Marketplace

```bash
claude plugin install convex-dev@your-marketplace
```

## Slash Commands

| Command | Description |
|---------|-------------|
| `/convex-dev:convex-query` | Create a new query with proper validators and index usage |
| `/convex-dev:convex-mutation` | Create a new mutation with auth and validators |
| `/convex-dev:convex-action` | Create an action for external API calls |
| `/convex-dev:convex-schema` | Create or update schema with proper indexes |

### Usage Examples

```
/convex-dev:convex-query list all messages for a channel with pagination
/convex-dev:convex-mutation create a new post with authentication
/convex-dev:convex-action generate text using OpenAI
/convex-dev:convex-schema add a comments table with user reference
```

## Skill Coverage

The plugin's skill automatically provides guidance for:

### Functions
- New function syntax with `args` and `returns` validators
- Public vs internal functions (`query` vs `internalQuery`)
- Function references with `api` and `internal` objects
- Proper error handling and authentication

### Schema & Database
- Schema definition with `defineTable` and validators
- Index best practices (no `_creationTime` in indexes)
- Query patterns with `withIndex()` (never `filter()`)
- Full-text search indexes

### Actions & Scheduling
- Node.js actions with `"use node";`
- Scheduler usage and limitations
- Cron job patterns
- Auth propagation in scheduled jobs

### React Integration
- `useQuery`, `useMutation`, `useAction` hooks
- Conditional queries with `"skip"`
- Loading state handling
- File upload patterns

### Authentication
- Convex Auth integration
- Protected functions with `getAuthUserId`
- User table schema

### File Storage
- Upload URL generation
- File URL retrieval
- Storage ID patterns

### AI Integration
- OpenAI action patterns
- Context loading for chat
- Rate limiting

## System Limits Reference

| Limit | Value |
|-------|-------|
| Function args/returns | 8 MiB |
| Array elements | 8192 |
| Query/Mutation timeout | 1 second |
| Action timeout | 10 minutes |

## Based On

This plugin is based on the guidelines from [Chef](https://github.com/get-convex/chef), Convex's official AI-powered full-stack app builder.

## License

MIT
