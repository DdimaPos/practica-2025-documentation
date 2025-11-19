# Data Flow Architecture

This document describes how data flows between the **Client**, **Next.js Server**, and **Supabase** in this application.

## Overview

The application uses a three-tier architecture:
- **Client** (React components in the browser)
- **Next.js Server** (API routes, server actions, server components)
- **Supabase** (Authentication + PostgreSQL Database)

## Architecture Diagram

```
┌─────────────────┐
│  Client (React) │
│  'use client'   │
└────────┬────────┘
         │
         │ HTTP Requests (fetch)
         │
┌────────▼─────────────────────┐
│   Next.js Server             │
│   ┌─────────────────────┐   │
│   │  API Routes         │   │
│   │  /api/posts         │   │
│   │  /api/comments      │   │
│   │  /api/search/posts  │   │
│   │  /api/user          │   │
│   └──────────┬──────────┘   │
│              │               │
│   ┌──────────▼──────────┐   │
│   │  Server Actions     │   │
│   │  (Server Functions) │   │
│   └──────────┬──────────┘   │
│              │               │
│   ┌──────────▼──────────┐   │
│   │  Supabase Client    │   │
│   │  (Server-side)      │   │
│   └──────────┬──────────┘   │
│              │               │
│   ┌──────────▼──────────┐   │
│   │  Drizzle ORM        │   │
│   │  (Database Layer)   │   │
│   └──────────┬──────────┘   │
└──────────────┼──────────────┘
               │
               │ PostgreSQL Protocol
               │ (DATABASE_URL)
               │
┌──────────────▼──────────────┐
│      Supabase               │
│  ┌──────────────────────┐   │
│  │  PostgreSQL Database │   │
│  └──────────────────────┘   │
│  ┌──────────────────────┐   │
│  │  Auth Service        │   │
│  └──────────────────────┘   │
└─────────────────────────────┘
```

## Data Flow Patterns

### 1. Authentication Flow

#### Client → Next.js Server → Supabase Auth

**Flow:**
1. Client components use browser-based Supabase client (`@/utils/supabase/client.ts`)
2. Authentication requests go through Next.js middleware (`middleware.ts`)
3. Middleware creates a Supabase server client and validates sessions
4. Server actions use server-side Supabase client to authenticate users

**Example Files:**
- `src/utils/supabase/client.ts` - Browser client for auth
- `src/utils/supabase/server.ts` - Server client for auth
- `src/utils/supabase/middleware.ts` - Middleware session validation
- `middleware.ts` - Next.js middleware wrapper

**Key Code:**
```typescript
// Middleware validates sessions on every request
const { data: { user } } = await supabase.auth.getUser();
if (!user && !isAuthRoute) {
  return NextResponse.redirect('/login');
}
```

### 2. Data Fetching Flow (Read Operations)

#### Client → API Route → Drizzle ORM → Supabase PostgreSQL

**Flow:**
1. Client component makes `fetch()` call to Next.js API route
2. API route handler processes the request
3. API route uses Drizzle ORM to query the database
4. Drizzle connects to Supabase's PostgreSQL database via `DATABASE_URL`
5. Results are returned as JSON to the client

**Example: Fetching Posts**

**Client Side** (`src/features/postsContainer/hooks/usePosts.tsx`):
```typescript
const response = await fetch(`/api/posts?limit=${POSTS_PER_PAGE}&offset=${currentOffset}`);
const { posts: newPosts, hasMore } = await response.json();
```

**API Route** (`src/app/api/posts/route.ts`):
```typescript
const results = await db
  .select({ id: posts.id, title: posts.title, ... })
  .from(posts)
  .innerJoin(users, eq(users.id, posts.authorId))
  .where(eq(posts.isActive, true))
  .orderBy(desc(posts.createdAt))
  .limit(limit)
  .offset(offset);

return NextResponse.json({ posts: transformedResults, hasMore });
```

**Database Connection** (`src/db/index.ts`):
```typescript
const client = postgres(process.env.DATABASE_URL!, { prepare: false });
const db = drizzle({ client: client, schema });
```

**Flow Sequence:**
1. Client: `usePosts` hook calls `fetch('/api/posts')`
2. Server: `GET /api/posts` route handler receives request
3. Server: Drizzle ORM queries PostgreSQL database
4. Database: Supabase PostgreSQL executes query
5. Server: Results transformed and returned as JSON
6. Client: Response parsed and state updated

### 3. Data Mutation Flow (Write Operations)

#### Client → API Route → Drizzle ORM → Supabase PostgreSQL

**Flow:**
1. Client component sends POST/PUT/DELETE request with data
2. API route validates and processes the request
3. API route uses Drizzle ORM to insert/update/delete data
4. Database changes are committed to Supabase PostgreSQL
5. Success response returned to client

**Example: Creating Comments**

**Client Side** (`src/features/PostContainer/components/ReplyContainer.tsx`):
```typescript
const res = await fetch('/api/comments', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    postId, authorId, parentCommentId, content, isAnonymous
  }),
});
const responseData = await res.json();
```

**API Route** (`src/app/api/comments/route.ts`):
```typescript
const result = await db
  .insert(comments)
  .values({
    postId, parentCommentId, authorId,
    content: content.trim(),
    isAnonymous: Boolean(isAnonymous),
    createdAt: new Date(),
  })
  .returning();

return NextResponse.json({ ok: true, comment: insertedComment }, { status: 201 });
```

### 4. Search Flow

#### Client → API Route → Drizzle ORM → Supabase PostgreSQL

**Flow:**
1. User types in search input (client component)
2. Search context debounces input and makes suggestion API call
3. Full search triggers main search API call
4. API route uses Drizzle ORM with `ilike` queries
5. Results filtered and returned from Supabase PostgreSQL

**Client Side** (`src/features/search/context/SearchContext.tsx`):
```typescript
// Debounced suggestion search
const url = `/api/search/posts?q=${encodeURIComponent(searchQuery)}&limit=5`;
const response = await fetch(url);
const data = await response.json();
setSuggestions(data);
```

**API Route** (`src/app/api/search/posts/route.ts`):
```typescript
const result = await searchPosts(query, limit);
return NextResponse.json(result);
```

**Search Query** (`src/lib/search/queries.ts`):
```typescript
const results = await db
  .select({ id: posts.id, title: posts.title, ... })
  .from(posts)
  .innerJoin(users, eq(users.id, posts.authorId))
  .where(and(
    eq(posts.isActive, true),
    or(
      ilike(posts.title, `%${sanitizedQuery}%`),
      ilike(posts.content, `%${sanitizedQuery}%`)
    )
  ))
  .orderBy(desc(posts.createdAt))
  .limit(limit);
```

### 5. User Data Flow

#### Client → API Route → Supabase Auth + Drizzle ORM → Supabase

**Flow:**
1. Client requests user data from `/api/user`
2. Server gets authenticated user from Supabase Auth
3. Server queries user details from PostgreSQL using Drizzle
4. Combined user data returned to client

**API Route** (`src/app/api/user/route.ts`):
```typescript
const user = await getUser();
return NextResponse.json(user);
```

**Server Action** (`src/utils/getUser.ts`):
```typescript
// 1. Get authenticated user from Supabase Auth
const supabase = await createClient();
const { data } = await supabase.auth.getUser();

// 2. Query user details from PostgreSQL
const uRes = await db
  .select()
  .from(users)
  .where(eq(users.authId, data.user.id))
  .limit(1);

// 3. Combine and return
return { ...data.user, ...user };
```

## Key Technologies

### Client-Side
- **React Hooks**: `useState`, `useEffect`, `useCallback` for state management
- **Fetch API**: Native browser API for HTTP requests
- **Supabase SSR Client**: Browser client for authentication

### Server-Side
- **Next.js API Routes**: RESTful endpoints at `/api/*`
- **Server Actions**: Server functions with `'use server'` directive
- **Drizzle ORM**: Type-safe SQL query builder
- **Supabase SSR Server Client**: Server-side Supabase client for auth

### Database
- **Supabase PostgreSQL**: Managed PostgreSQL database
- **Connection**: Via `postgres-js` using `DATABASE_URL` environment variable

## Data Flow Patterns Summary

| Operation | Client → Server | Server → Database | Database |
|-----------|----------------|-------------------|----------|
| **Read Posts** | `fetch('/api/posts')` | Drizzle `.select()` | PostgreSQL SELECT |
| **Create Comment** | `fetch('/api/comments', { method: 'POST' })` | Drizzle `.insert()` | PostgreSQL INSERT |
| **Search** | `fetch('/api/search/posts?q=...')` | Drizzle with `ilike()` | PostgreSQL SELECT with LIKE |
| **Get User** | `fetch('/api/user')` | Supabase Auth + Drizzle | Auth + PostgreSQL SELECT |
| **Authenticate** | Supabase client auth | Middleware validation | Supabase Auth Service |

## Security Considerations

1. **Middleware Protection**: All routes go through middleware that validates Supabase sessions
2. **Server-Side Validation**: API routes validate all inputs before database operations
3. **SQL Injection Prevention**: Drizzle ORM uses parameterized queries
4. **HTTP-Only Cookies**: Auth cookies set with `httpOnly: true` flag
5. **Environment Variables**: Database credentials stored in server-only environment variables

## State Management

- **Client State**: React hooks manage component-level state
- **Server State**: API routes handle data fetching and mutations
- **Shared Context**: `SearchContext` provides global search state across components
- **Optimistic Updates**: Components may show optimistic UI updates before server confirmation

## Caching Strategy

- **Comments Cache**: Server-side caching with revalidation (`revalidateCommentsCache`)
- **Next.js Cache**: Leverages Next.js built-in caching for static and dynamic routes
- **Client Cache**: React state holds fetched data until explicit refresh

