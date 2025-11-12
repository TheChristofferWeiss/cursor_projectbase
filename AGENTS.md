# AGENTS.md - AI Assistant Guide

This document provides guidance for AI assistants working with this codebase. It explains the project structure, conventions, and common patterns to help you understand and modify the code effectively.

## Project Overview

This is a **Next.js 14 + Supabase + Vercel** starter template with:
- **Framework**: Next.js 14 with App Router
- **Database**: Supabase (PostgreSQL)
- **Authentication**: Supabase Auth with email/password
- **Deployment**: Vercel
- **Language**: TypeScript
- **Styling**: Tailwind CSS

## Key Architecture Patterns

### 1. Supabase Client Usage

**Client Components** (browser):
```typescript
import { createClient } from '@/lib/supabase/client'

// Use in Client Components ('use client')
const supabase = createClient()
```

**Server Components & Server Actions**:
```typescript
import { createClient } from '@/lib/supabase/server'

// Use in Server Components, Server Actions, Route Handlers
const supabase = createClient()
```

**Admin Operations** (service role):
```typescript
import { createAdminClient } from '@/lib/supabase/admin'

// Use for admin operations requiring service role key
const supabase = createAdminClient()
```

### 2. Authentication Flow

1. **Sign Up**: `src/app/auth/signup/page.tsx` → creates user → redirects to `/auth/callback`
2. **Sign In**: `src/app/auth/login/page.tsx` → authenticates → redirects to `/dashboard`
3. **Callback**: `src/app/auth/callback/route.ts` → exchanges code for session → redirects to `/dashboard`
4. **Sign Out**: `src/app/auth/signout/route.ts` → clears session → redirects to `/auth/login`

**Important**: Always use the server client (`@/lib/supabase/server`) in route handlers and server components. Use the browser client (`@/lib/supabase/client`) only in Client Components.

### 3. Database Schema

- **`auth.users`**: Managed by Supabase Auth (don't modify directly)
- **`public.profiles`**: User profiles with role-based access
  - `id` (uuid, references `auth.users`)
  - `full_name` (text)
  - `role` (enum: 'admin' | 'user')
  - `created_at` (timestamptz)

**Row Level Security (RLS)**:
- Profiles are viewable by everyone
- Users can only insert/update their own profile
- Admin operations require service role key

### 4. Type Safety

Database types are auto-generated:
```bash
# Generate types from local database
supabase gen types typescript --local > src/lib/database.types.ts

# Generate types from cloud database
supabase gen types typescript --project-id <project-id> > src/lib/database.types.ts
```

Always regenerate types after database migrations!

## Project Structure

```
src/
├── app/                    # Next.js App Router
│   ├── auth/              # Authentication pages
│   │   ├── login/         # Sign in page
│   │   ├── signup/        # Sign up page
│   │   ├── callback/      # Auth callback handler
│   │   └── signout/       # Sign out handler
│   ├── dashboard/         # Protected dashboard page
│   ├── layout.tsx          # Root layout
│   ├── page.tsx            # Landing page
│   └── globals.css         # Global styles
├── components/             # React components (currently empty)
└── lib/
    ├── supabase/
    │   ├── client.ts       # Browser client (Client Components)
    │   ├── server.ts       # Server client (Server Components/Actions)
    │   └── admin.ts        # Admin client (service role)
    └── database.types.ts   # Generated TypeScript types

supabase/
├── migrations/             # Database migrations
│   └── 20240101000000_setup_user_roles.sql
└── config.toml            # Supabase local config

docs/                       # Documentation
```

## Common Tasks

### Adding a New Page

1. Create a new directory in `src/app/`:
   ```typescript
   // src/app/new-page/page.tsx
   export default function NewPage() {
     return <div>New Page</div>
   }
   ```

2. For protected pages, check authentication:
   ```typescript
   import { createClient } from '@/lib/supabase/server'
   import { redirect } from 'next/navigation'
   
   export default async function ProtectedPage() {
     const supabase = createClient()
     const { data: { user } } = await supabase.auth.getUser()
     
     if (!user) {
       redirect('/auth/login')
     }
     
     return <div>Protected Content</div>
   }
   ```

### Adding a Database Table

1. Create a migration in `supabase/migrations/`:
   ```sql
   -- supabase/migrations/YYYYMMDDHHMMSS_table_name.sql
   create table public.table_name (
     id uuid primary key default gen_random_uuid(),
     created_at timestamptz default now()
   );
   ```

2. Apply migration locally:
   ```bash
   supabase db reset  # Resets and applies all migrations
   ```

3. Generate types:
   ```bash
   supabase gen types typescript --local > src/lib/database.types.ts
   ```

4. Push to cloud when ready:
   ```bash
   supabase db push
   supabase gen types typescript --project-id <project-id> > src/lib/database.types.ts
   ```

### Creating a Server Action

```typescript
'use server'

import { createClient } from '@/lib/supabase/server'

export async function myServerAction(formData: FormData) {
  const supabase = createClient()
  const { data: { user } } = await supabase.auth.getUser()
  
  if (!user) {
    return { error: 'Unauthorized' }
  }
  
  // Your logic here
  return { success: true }
}
```

### Creating a Route Handler

```typescript
// src/app/api/example/route.ts
import { createClient } from '@/lib/supabase/server'
import { NextResponse } from 'next/server'

export async function GET(request: Request) {
  const supabase = createClient()
  const { data: { user } } = await supabase.auth.getUser()
  
  if (!user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }
  
  return NextResponse.json({ data: 'example' })
}
```

## Environment Variables

### Local Development (`.env.local`)
```env
NEXT_PUBLIC_SUPABASE_URL=http://127.0.0.1:54321
NEXT_PUBLIC_SUPABASE_ANON_KEY=<local-anon-key>
SUPABASE_SERVICE_ROLE_KEY=<local-service-role-key>
```

**Important**: Use **local** Supabase credentials for development (from `supabase start` output).

### Production (Vercel)
```env
NEXT_PUBLIC_SUPABASE_URL=https://<project>.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=<cloud-anon-key>
SUPABASE_SERVICE_ROLE_KEY=<cloud-service-role-key>
```

**Important**: Vercel uses **cloud** Supabase credentials (set in Vercel dashboard).

## Development Workflow

1. **Start local Supabase**: `supabase start`
2. **Update `.env.local`** with local credentials
3. **Apply migrations**: `supabase db reset`
4. **Generate types**: `supabase gen types typescript --local > src/lib/database.types.ts`
5. **Start dev server**: `npm run dev`
6. **Make changes** and test locally
7. **When ready**: Push migrations to cloud, then push code to GitHub (Vercel auto-deploys)

## Important Notes

- **Never commit** `.env.local` or service role keys
- **Always regenerate** database types after migrations
- **Use server client** in Server Components, Server Actions, and Route Handlers
- **Use browser client** only in Client Components
- **RLS policies** are enabled on `profiles` table - check policies before querying
- **Email auth** works locally via Inbucket at `http://localhost:54324`
- **Cloud infrastructure** is set up first, then develop locally, then deploy

## Code Conventions

- Use TypeScript for all files
- Use async/await for async operations
- Always handle errors in auth operations
- Use Tailwind CSS for styling
- Follow Next.js App Router conventions
- Use server components by default, add `'use client'` only when needed

## Troubleshooting

- **Auth not working**: Check redirect URLs in Supabase Dashboard
- **Type errors**: Regenerate database types
- **RLS errors**: Check policies and ensure user is authenticated
- **Build fails**: Check environment variables in Vercel
- **Port conflicts**: Stop other Supabase instances with `supabase stop`

For more details, see the [documentation](./docs/) directory.

