# Next.js + Supabase + Vercel Starter Kit

A production-ready starter template for building full-stack applications with Next.js, Supabase, and Vercel. 

**Workflow:** Set up cloud Supabase connection first (so deployment is ready), then develop locally using local Supabase. When your code is ready, deploy to production with confidence!

## 🚀 Quick Start

### Prerequisites

- Node.js 18+ and npm
- [Docker](https://www.docker.com/get-started) and Docker Compose
- [Supabase CLI](https://supabase.com/docs/guides/cli/getting-started)
- A Supabase account ([sign up free](https://supabase.com))
- A Vercel account ([sign up free](https://vercel.com)) - for deployment
- Git

### Step 1: Clone and Setup

```bash
# Clone this repository
git clone https://github.com/thechristofferweiss/nextjs-supabase-vercel-starter.git
cd nextjs-supabase-vercel-starter

# Install dependencies
npm install

# Copy environment template
cp .env.example .env.local
```

### Step 2: Set Up Cloud Supabase Connection (Before Development)

Set up your cloud Supabase project first so it's ready for deployment. We'll develop locally, but this ensures your cloud database is configured.

**a.** Create a new project at [supabase.com](https://supabase.com)

**b.** Go to **Settings → API** and copy your project URL and anon key

**c.** Install Supabase CLI (if not already installed):
   ```bash
   # On macOS:
   brew install supabase/tap/supabase
   # Or see: https://supabase.com/docs/guides/cli/getting-started
   ```

**d.** Login to Supabase (opens browser for authentication):
   ```bash
   supabase login
   ```

**e.** Initialize Supabase (creates supabase/ directory if it doesn't exist):
   ```bash
   supabase init
   ```

**f.** Link to your cloud project (replace `<project-id>` with your Project ID):
   ```bash
   # Find your Project ID in: Supabase Dashboard → Settings → General → Project ID
   supabase link --project-ref <project-id>
   ```

**g.** Push migrations to your cloud Supabase project:
   ```bash
   supabase db push
   ```

**h.** Generate TypeScript types from cloud database:
   ```bash
   supabase gen types typescript --project-id <project-id> > src/lib/database.types.ts
   ```

**✅ Cloud Supabase connection is now set up and ready for deployment!**

### Step 3: Set Up Vercel (Ready for Deployment)

Set up Vercel now so deployment is ready when you need it.

**a.** Push your code to GitHub:
   ```bash
   git add .
   git commit -m "Initial setup"
   git push origin main
   ```

**b.** Import your repository in [Vercel](https://vercel.com)

**c.** Add environment variables in Vercel project settings (use your **cloud** Supabase credentials):
   - `NEXT_PUBLIC_SUPABASE_URL` (your cloud Supabase URL)
   - `NEXT_PUBLIC_SUPABASE_ANON_KEY` (your cloud Supabase anon key)
   - `SUPABASE_SERVICE_ROLE_KEY` (your cloud Supabase service role key)

**d.** Configure Supabase redirect URLs:
   - Go to Supabase Dashboard → **Authentication → URL Configuration**
   - Set **Site URL** to your Vercel URL: `https://your-app.vercel.app`
   - Add **Redirect URLs**:
     - `https://your-app.vercel.app/auth/callback`
     - `https://your-app.vercel.app/**`
     - `http://localhost:3000/auth/callback` (for local development)
     - `http://localhost:3000/**`

**✅ Vercel is now configured!** When you push to GitHub, Vercel will automatically deploy.

---

### Step 4: Set Up Local Supabase for Development

Now set up local Supabase - this is where you'll develop and test your application.

**Important:** Run these commands **one at a time** in your terminal.

```bash
# Start local Supabase (runs PostgreSQL, Auth, Storage, etc. in Docker)
supabase start
```

After starting, Supabase will display your local credentials. Update your `.env.local` with **local** credentials:

```bash
# The 'supabase start' output will show:
# - API URL → use for NEXT_PUBLIC_SUPABASE_URL (local)
# - anon key → use for NEXT_PUBLIC_SUPABASE_ANON_KEY (local)
# - service_role key → use for SUPABASE_SERVICE_ROLE_KEY (local)

# Edit .env.local and replace with local credentials for development
```

**Email Authentication:** Local Supabase handles email auth via Inbucket (email testing server). View captured emails at `http://localhost:54324`. Email verification links work locally and redirect to your local app.

### Step 5: Apply Database Migrations Locally

```bash
# Apply migrations to your local Supabase instance
supabase db reset

# Generate TypeScript types from your local database
supabase gen types typescript --local > src/lib/database.types.ts
```

### Step 6: Run the Application Locally

```bash
# Make sure Supabase is running (if not, run: supabase start)
# Then start Next.js with hot reload
npm run dev
```

Visit [http://localhost:3000](http://localhost:3000) to see your app running locally!

**🎉 You're now developing locally!** Make changes, test features, and iterate quickly using your local Supabase instance. Your cloud infrastructure (Supabase + Vercel) is already set up and ready for when you want to deploy.

---

## 🚀 Deploy to Production

When your code is ready and tested locally, deploy to production:

**a.** If you created new migrations locally, push them to cloud first:
   ```bash
   supabase db push
   supabase gen types typescript --project-id <project-id> > src/lib/database.types.ts
   ```

**b.** Push your code to GitHub:
   ```bash
   git add .
   git commit -m "Ready for production"
   git push origin main
   ```

Vercel will automatically deploy your changes! Your cloud Supabase is already configured.

## 📚 Documentation

- **[Complete Setup Guide](./docs/setup-guide.md)** - Step-by-step setup instructions
- **[User Management Guide](./docs/user-management.md)** - Authentication, roles, and user management
- **[Deployment Guide](./docs/deployment.md)** - Vercel deployment and production considerations
- **[Docker Guide](./docs/docker.md)** - Running with Docker and Docker Compose
- **[Troubleshooting](./docs/troubleshooting.md)** - Common issues and solutions

## ✨ Features

- ✅ **Authentication** - Email/password with Supabase Auth
- ✅ **User Roles** - Role-based access control (Admin, User, etc.)
- ✅ **Database** - PostgreSQL with Row Level Security
- ✅ **Type Safety** - Auto-generated TypeScript types from database
- ✅ **Server Actions** - Next.js 14 App Router with server actions
- ✅ **Local Development** - Full local stack with Docker and Supabase CLI
- ✅ **Hot Reload** - Fast iteration with Next.js hot reload
- ✅ **Production Ready** - Configured for Vercel deployment after local testing

## 🏗️ Project Structure

```
├── src/
│   ├── app/              # Next.js App Router pages
│   │   ├── auth/         # Authentication pages
│   │   └── (protected)/  # Protected routes
│   ├── components/       # React components
│   └── lib/              # Utilities and helpers
│       ├── supabase/     # Supabase client helpers
│       └── database.types.ts  # Generated types
├── supabase/
│   ├── migrations/       # Database migrations
│   └── functions/        # Edge functions (optional)
├── docs/                 # Documentation
└── public/               # Static assets
```

## 🛠️ Tech Stack

- **Framework**: [Next.js 14](https://nextjs.org) (App Router)
- **Database**: [Supabase](https://supabase.com) (PostgreSQL)
- **Authentication**: Supabase Auth
- **Deployment**: [Vercel](https://vercel.com)
- **Styling**: Tailwind CSS (optional)
- **Language**: TypeScript

## 📖 Development Workflow

This starter uses a **"Cloud-Ready, Local-Development"** approach:

### Phase 1: Set Up Cloud Infrastructure First
- Create cloud Supabase project
- Link local project to cloud
- Push migrations to cloud
- Set up Vercel deployment
- **Result:** Cloud infrastructure is ready for deployment

### Phase 2: Develop Locally
- Use local Supabase for all development (`supabase start`)
- Update `.env.local` with local credentials
- Develop and test features locally
- Fast iteration with hot reload

### Phase 3: Deploy When Ready
- Push new migrations to cloud (if any)
- Push code to GitHub
- Vercel auto-deploys (already configured!)

**Benefits:**
- ✅ **Cloud ready** - Infrastructure set up upfront
- ✅ **Fast local development** - No cloud latency, work offline
- ✅ **No cloud costs** during development
- ✅ **Confidence** - Deploy when code is ready and tested
- ✅ **Easy deployment** - Just push to GitHub

## 🔐 Environment Variables

### For Local Development

Your `.env.local` file should contain your **local Supabase credentials** (from `supabase start` output):

```env
NEXT_PUBLIC_SUPABASE_URL=http://127.0.0.1:54321
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-local-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-local-service-role-key
```

### For Production (Vercel)

Vercel uses **cloud Supabase credentials** (set in Vercel environment variables):
- `NEXT_PUBLIC_SUPABASE_URL` - Your cloud Supabase URL
- `NEXT_PUBLIC_SUPABASE_ANON_KEY` - Your cloud Supabase anon key
- `SUPABASE_SERVICE_ROLE_KEY` - Your cloud Supabase service role key

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📝 License

MIT License - feel free to use this for your projects!

## 🙏 Acknowledgments

Built with experience from production deployments. This starter includes solutions to common issues encountered when building with Next.js, Supabase, and Vercel.

---

**Need help?** Check the [documentation](./docs/) or [open an issue](https://github.com/thechristofferweiss/nextjs-supabase-vercel-starter/issues).

