---
name: tanstack-best-practice
description: Enforce TanStack frontend best practices when writing React/TypeScript code
---

# TanStack Frontend Best Practices

## Core Principles
- Max 200 LOC per file
- No code comments
- No semicolons
- Single quotes
- 2-space indentation
- Strict TypeScript (no `any`)
- Reusability-first design
- Single responsibility per file

## API Layer Pattern

Every feature API must follow this 4-file structure in `src/apis/{feature}/`:

### types.ts
```typescript
export type User = {
  id: string
  email: string
  created_at: string
}

export type CreateUserPayload = {
  email: string
  password: string
}
```

### service.ts
```typescript
import { apiClient } from '@/libs/axios'
import type { User, CreateUserPayload } from './types'

const BASE = '/api/v1/users'

export const usersService = {
  list: (params?: ListParams) =>
    apiClient.get<ListResponse<User>>(BASE, { params }),
  get: (id: string) =>
    apiClient.get<SingleResponse<User>>(`${BASE}/${id}`),
  create: (payload: CreateUserPayload) =>
    apiClient.post<SingleResponse<User>>(BASE, payload),
}
```

### hooks.ts
```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import { usersService } from './service'

export const userKeys = {
  all: ['users'] as const,
  lists: () => [...userKeys.all, 'list'] as const,
  list: (params?: ListParams) => [...userKeys.lists(), params] as const,
  details: () => [...userKeys.all, 'detail'] as const,
  detail: (id: string) => [...userKeys.details(), id] as const,
}

export function useUsers(params?: ListParams) {
  return useQuery({
    queryKey: userKeys.list(params),
    queryFn: () => usersService.list(params).then((res) => res.data),
  })
}

export function useCreateUser() {
  const queryClient = useQueryClient()
  return useMutation({
    mutationFn: usersService.create,
    onSuccess: () => queryClient.invalidateQueries({ queryKey: userKeys.lists() }),
  })
}
```

### index.ts
```typescript
export * from './types'
export * from './service'
export * from './hooks'
```

## Component Organization

```
src/components/
├── ui/           # Reusable UI (Button, Input, Card, Table)
├── features/     # Domain-specific (UserForm, AccountTable)
└── layout/       # Layouts, guards (DashboardLayout, ProtectedRoute)
```

## State Management
- Server state: TanStack Query
- Client state: TanStack Store with selectors
- Forms: TanStack Form + Zod validation

## File Patterns

| Type | Location | Naming |
|------|----------|--------|
| API module | `src/apis/{feature}/` | lowercase |
| UI component | `src/components/ui/` | PascalCase.tsx |
| Feature component | `src/components/features/` | PascalCase.tsx |
| Route | `src/routes/` | file-based routing |
| Lib config | `src/libs/{lib}/` | index.ts |

## Response Types
```typescript
type SingleResponse<T> = { data: T; message: string }
type ListResponse<T> = { data: T[]; pagination: PaginationMeta }
```

## Rules
1. Always use query key factories
2. Invalidate queries on mutations
3. Separate types from implementation
4. Use barrel exports (index.ts)
5. Keep components pure when possible
6. Extract reusable logic to hooks
7. Use Zod for all form validation
