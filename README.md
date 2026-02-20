# TanStack and Axum Claude Code Skills

Claude Code skills that enforce best practices for fullstack development with TanStack (React) frontend and Axum (Rust) backend.

## Overview

This repository contains two auto-invoking skills for Claude Code:

| Skill | Purpose |
|-------|---------|
| `tanstack-best-practice` | Enforces React/TypeScript patterns using TanStack ecosystem |
| `axum-best-practice` | Enforces Rust/Axum backend patterns with Clean Architecture |

## Installation

Copy the `.claude/skills` directory to your project:

```bash
cp -r .claude/skills /path/to/your/project/.claude/
```

Or copy to your global Claude Code configuration:

```bash
cp -r .claude/skills ~/.claude/
```

## Skills

### TanStack Best Practice

Automatically applies when writing React/TypeScript code.

**Key Patterns Enforced:**

- API Layer Pattern (4-file structure per feature)
  - `types.ts` - TypeScript interfaces
  - `service.ts` - API calls via axios
  - `hooks.ts` - TanStack Query hooks with key factories
  - `index.ts` - Barrel exports
- Component Organization
  - `components/ui/` - Reusable UI components
  - `components/features/` - Domain-specific components
  - `components/layout/` - Layouts and route guards
- State Management
  - Server state via TanStack Query
  - Client state via TanStack Store
  - Forms via TanStack Form with Zod validation

**Code Style:**

- No semicolons
- Single quotes
- 2-space indentation
- Strict TypeScript (no `any`)
- Max 200 lines per file

### Axum Best Practice

Automatically applies when writing Rust code.

**Key Patterns Enforced:**

- Clean Architecture (domain -> application -> infrastructure)
- Module Structure
  - `domain/` - Entities, repository traits, value objects
  - `application/` - Use cases (one per file)
  - `infrastructure/http/` - Handlers, routes, DTOs
  - `infrastructure/persistence/` - Repository implementations
- Repository Pattern
  - Traits defined in domain layer
  - Implementations in infrastructure layer
- Centralized Error Handling
  - `AppError` enum with `IntoResponse` implementation
  - Semantic HTTP status codes

**Code Style:**

- No `unwrap()` in production code
- Use `?` operator for error propagation
- Max 200 lines per file
- No code comments

## Project Structure

```
.claude/skills/
├── axum-best-practice/
│   └── SKILL.md
└── tanstack-best-practice/
    └── SKILL.md
```

## Common Principles

Both skills enforce:

- **DRY** - Extract shared patterns into reusable modules
- **Atomic** - Single responsibility per file/function
- **Reusability** - Design for composition
- **Type Safety** - Strict typing throughout
- **Max 200 LOC** - Keep files focused and readable
- **No Comments** - Code should be self-documenting

## Reference Projects

These skills are derived from:

- [axum-backend-best-practice](https://github.com/maulanasdqn/axum-backend-best-practice) - Rust/Axum backend reference
- [tanstack-frontend-best-practice](https://github.com/maulanasdqn/tanstack-frontend-best-practice) - React/TanStack frontend reference

## Usage

Once installed, the skills automatically activate when Claude Code detects relevant file types:

- `.rs` files trigger the Axum skill
- `.ts`, `.tsx` files trigger the TanStack skill

You can also invoke them manually:

```
/axum-best-practice
/tanstack-best-practice
```

## License

MIT
