# Convex Development Skills

A collection of agent skills for Convex development, providing guidelines, best practices, and patterns based on [Chef's](https://github.com/get-convex/chef) production patterns.

## Installation

### Using add-skill (Recommended)

Works with Claude Code, Cursor, Codex, and 15+ other agents:

```bash
# Install all skills
npx add-skill geolize/convex-claude-plugin

# List available skills
npx add-skill geolize/convex-claude-plugin --list

# Install specific skill
npx add-skill geolize/convex-claude-plugin -s convex-core

# Install to a specific agent
npx add-skill geolize/convex-claude-plugin -s convex-core -a claude-code

# Install globally
npx add-skill geolize/convex-claude-plugin -g
```

### Manual Installation

Copy the desired skill directory from `skills/` into your agent's skills directory:

- **Claude Code**: `.claude/skills/<skill-name>/`
- **Cursor**: `.cursor/skills/<skill-name>/`
- **Codex**: `.codex/skills/<skill-name>/`
- **VS Code/Copilot**: `.github/skills/`

## Available Skills

| Skill | Description |
|-------|-------------|
| `convex-core` | Core Convex development - functions, validators, schema, queries, mutations, and database patterns |
| `convex-auth` | Authentication - user management, protected functions, session handling with @convex-dev/auth |
| `convex-react` | React client - hooks, real-time updates, optimistic updates, pagination, and UI patterns |
| `convex-ai` | AI Integration - OpenAI, actions, streaming, and AI patterns with database integration |
| `convex-components` | Components - Presence, ProseMirror/BlockNote collaborative editing, and Resend email |

## Skill Coverage

### convex-core

- New function syntax with `args` and `returns` validators
- Public vs internal functions (`query` vs `internalQuery`)
- Schema definition with `defineTable` and validators
- Index best practices (no `_creationTime` in indexes)
- Query patterns with `withIndex()` (never `filter()`)
- Full-text search indexes
- Pagination patterns

### convex-auth

- Convex Auth integration with `@convex-dev/auth`
- Protected functions with `getAuthUserId`
- User table schema and extension
- Auth in scheduled jobs (userId propagation)
- HTTP endpoints with auth

### convex-react

- `useQuery`, `useMutation`, `useAction` hooks
- Conditional queries with `"skip"`
- Loading state handling (`undefined` vs `null`)
- Optimistic updates
- File upload patterns
- Provider setup

### convex-ai

- OpenAI action patterns with `"use node";`
- Context loading for chat
- Scheduling AI responses
- Error handling and rate limiting
- Environment variables and secrets

### convex-components

- Presence component for real-time user tracking
- ProseMirror/BlockNote collaborative editing
- Resend email integration
- Multi-component configuration

## System Limits Reference

| Limit | Value |
|-------|-------|
| Function args/returns | 8 MiB |
| Array elements | 8192 |
| Query/Mutation timeout | 1 second |
| Action timeout | 10 minutes |

## Based On

These skills are based on the guidelines from [Chef](https://github.com/get-convex/chef), Convex's official AI-powered full-stack app builder.

## License

MIT
