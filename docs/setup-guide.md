# Ultimate Guide: Next.js, Vercel, and Supabase Starter Kit

This document is a production-focused guide for bootstrapping a new, fully serverless project using Next.js, Vercel, and Supabase. It is designed to be a reusable template that anticipates common issues and establishes a fast, modern development workflow.

---

## 1. The Development Workflow: Local Frontend, Live Backend

This guide establishes a powerful "local frontend, live backend" development workflow.
-   **Frontend:** You will run the Next.js development server locally (`npm run dev`). This gives you instant feedback and hot-reloading for UI changes.
-   **Backend:** Your local frontend will connect directly to your **live, cloud-hosted Supabase project**. Database changes are pushed to the cloud and are immediately available to your local server.
-   **Deployment:** You only need to deploy to Vercel when a feature is complete and tested locally.

---

## 2. Local Setup & Project Initialization

### 1.1. Create the Next.js App & Git Repo
From your projects folder, create the app, navigate into it, and initialize a Git repository.

```bash
npx create-next-app@latest my-project
cd my-project
git init
```

### 1.2. Proactive Git Branch Check (Important!)
Vercel and GitHub work best with a default branch named `main`. Check your current branch name and rename it if necessary to avoid future deployment issues.

```bash
git branch # Check the current branch (often 'master' by default)
git branch -M main # If not 'main', rename it
```

### 1.3. Install Supabase Auth Helpers
Install the necessary packages for connecting Next.js to Supabase authentication.

```bash
npm install @supabase/auth-helpers-nextjs @supabase/supabase-js
```

### 1.4. Initialize Supabase

**Important:** Run these commands **one at a time** in your terminal. Do not copy-paste them all at once.

1.  **Install/Verify CLI:** `supabase --version` (If not installed, use `brew install supabase/tap/supabase` on macOS or see official docs).
2.  **Log In:** `supabase login` (authorizes via browser).
3.  **Initialize:** `supabase init` (creates the `supabase/` directory).

---

## 3. Connect to Your Supabase Project

### 3.1. Create a `.env.local` File
This file stores your Supabase credentials for local development. It is crucial that this file is listed in your `.gitignore` and never committed.

`/.env.local`:
```
NEXT_PUBLIC_SUPABASE_URL=<your-project-url>
NEXT_PUBLIC_SUPABASE_ANON_KEY=<your-anon-key>
```

### 3.2. Link to Remote Project
Link your local setup to your remote Supabase project. 

**Find your Project ID:** Go to Supabase Dashboard → Settings → General → Project ID (it's a 20-character string).

**Important:** Run this command separately after completing the previous steps.

```bash
supabase link --project-ref <your-project-id>
# You will be prompted for your database password.
```

### 3.3. Generate Initial Database Types
Create the TypeScript definitions for your database schema. This is crucial for type safety and autocompletion. The file will be empty for now, but we'll regenerate it later.

```bash
supabase gen types typescript --project-id <your-project-ref> > src/lib/database.types.ts
```

---

## 4. Database Migrations & User Authentication

### 3.1. Create User Roles & Profiles Table
This migration sets up a `profiles` table that is automatically populated when a new user signs up, and includes a `role` for managing permissions.

1.  **Create Migration File:**
    ```bash
    supabase migrations new setup_user_roles
    ```
2.  **Add SQL to the new file:** (`supabase/migrations/<timestamp>_setup_user_roles.sql`)
    ```sql
    -- Create a custom type for user roles
    create type public.app_role as enum ('admin', 'user');

    -- Create a table for user profiles
    create table public.profiles (
      id uuid not null primary key references auth.users on delete cascade,
      full_name text,
      role public.app_role not null default 'user'
    );
    alter table public.profiles enable row level security;

    -- RLS Policies: Users can view all profiles, but only manage their own.
    create policy "Public profiles are viewable by everyone." on public.profiles for select using (true);
    create policy "Users can insert their own profile." on public.profiles for insert with check (auth.uid() = id);
    create policy "Users can update own profile." on public.profiles for update using (auth.uid() = id);

    -- Trigger to automatically create a profile for new sign-ups
    create function public.handle_new_user() returns trigger as $$
    begin
      insert into public.profiles (id) values (new.id);
      return new;
    end;
    $$ language plpgsql security definer;
    create trigger on_auth_user_created after insert on auth.users for each row execute procedure public.handle_new_user();
    ```

### 3.2. Create a "Test the Connection" Table
This migration creates a simple `items` table to verify that your frontend can read and write data.

1.  **Create Migration File:**
    ```bash
    supabase migrations new create_items_table
    ```
2.  **Add SQL to the new file:**
    ```sql
    -- Create a "items" table
    create table public.items (
      id bigint generated by default as identity primary key,
      created_at timestamptz default now(),
      name text,
      user_id uuid references public.profiles
    );
    alter table public.items enable row level security;

    -- RLS Policies: Users can only manage their own items.
    create policy "Users can manage their own items." on public.items for all using (auth.uid() = user_id);
    ```

### 3.3. Apply Migrations & Regenerate Types
Push the new tables to your Supabase database and then regenerate the TypeScript definitions to include them.

```bash
supabase db push
supabase gen types typescript --project-id <your-project-ref> > src/lib/database.types.ts
```

---

## 5. Frontend Setup

### 4.1. Create Supabase Client Helpers
Create two files to easily initialize the Supabase client in either server or client components.

`src/lib/supabase/client.ts`:
```typescript
import { createPagesBrowserClient } from '@supabase/auth-helpers-nextjs'
export const createClient = () => createPagesBrowserClient()
```

`src/lib/supabase/server.ts`:
```typescript
import { createServerComponentClient } from '@supabase/auth-helpers-nextjs'
import { cookies } from 'next/headers'
export const createClient = () => createServerComponentClient({ cookies })
```

### 4.2. Create Authentication Pages
-   **Sign Up (`src/app/signup/page.tsx`):** A client component with a form to sign up new users.
-   **Login (`src/app/login/page.tsx`):** A client component with a form to log in existing users.

### 4.3. Create the Auth Callback Route
This is a critical route that handles the redirect from Supabase's email confirmation link.

`src/app/auth/callback/route.ts`:
```typescript
import { createClient } from '@/lib/supabase/server';
import { NextResponse } from 'next/server';

export async function GET(request: Request) {
  const requestUrl = new URL(request.url);
  const code = requestUrl.searchParams.get('code');

  if (code) {
    const supabase = createClient();
    await supabase.auth.exchangeCodeForSession(code);
  }

  // URL to redirect to after sign in process completes
  return NextResponse.redirect(requestUrl.origin);
}
```

---

## 6. Deployment to Vercel & Production Workflow

### 6.1. Configure the Project in Vercel
1.  Push your code to a new GitHub repository.
2.  In Vercel, import the Git repository.
3.  In the project settings (**Settings -> Environment Variables**), add your Supabase credentials:
    -   `NEXT_PUBLIC_SUPABASE_URL`
    -   `NEXT_PUBLIC_SUPABASE_ANON_KEY`

### 6.2. Configure Supabase Auth URLs
To ensure email confirmation links work correctly for both local development and your live site, configure the URLs in Supabase.
1.  Go to your Supabase Dashboard -> **Authentication -> URL Configuration**.
2.  Set **Site URL** to your Vercel deployment URL (e.g., `https://my-project.vercel.app`).
3.  Add **both** your local and production URLs to **Redirect URLs**:
    -   `https://my-project.vercel.app/auth/callback`
    -   `https://my-project.vercel.app/**`
    -   `http://localhost:3000/auth/callback`
    -   `http://localhost:3000/**`

### 6.3. Pushing to Production
Once you have tested your features locally, you can deploy them to Vercel with a simple `git push`.

```bash
git add .
git commit -m "feat: A descriptive message of the new feature"
git push origin main
```
Vercel will automatically detect the push and start a new deployment.

---

## 7. Common Gotchas & Troubleshooting

-   **Zsh `no matches found` error:** When using `git add` with a file path that includes square brackets (like `[stageId]`), your terminal may try to interpret it. **Solution:** Wrap the filename in quotes: `git add "src/app/stage/[stageId]/page.tsx"`.
-   **Vercel Build Fails (`database.types.ts` not found):** You forgot to generate or commit the database types file. Run `supabase gen types ...` and commit the file.
-   **Confirmation Links Go to `localhost`:** You haven't configured the **Site URL** and **Redirect URLs** in your Supabase project's auth settings (Step 5.2).
-   **`Dynamic server usage` Messages in Build Log:** These are not errors. They are informational messages from Next.js telling you that a page which uses `cookies()` cannot be statically generated and will be rendered dynamically on the server. This is the expected and correct behavior for authenticated pages.

This generalized guide provides a solid, repeatable foundation for starting any new full-stack project with Next.js and Supabase.
