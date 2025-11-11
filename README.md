# Next.js + Supabase + Vercel Starter Kit

A production-ready starter template for building full-stack applications with Next.js, Supabase, and Vercel. 

**Workflow:** Set up cloud infrastructure (Supabase + Vercel) first so deployment is ready, then develop locally. When your code is ready, deploy with confidence!

## 🚀 Quick Start

### Prerequisites

- Node.js 18+ and npm
- [Docker](https://www.docker.com/get-started) and Docker Compose
- [Supabase CLI](https://supabase.com/docs/guides/cli/getting-started)
- A Supabase account ([sign up free](https://supabase.com))
- A Vercel account ([sign up free](https://vercel.com))
- Git

### Phase 1: Set Up Cloud Infrastructure (Ready for Deployment)

Set up your cloud Supabase and Vercel projects first so deployment is ready when you need it.

#### Step 1: Clone and Setup

```bash
# Clone this repository
git clone https://github.com/thechristofferweiss/nextjs-supabase-vercel-starter.git
cd nextjs-supabase-vercel-starter

# Install dependencies
npm install

# Copy environment template
cp .env.example .env.local
```

#### Step 2: Set Up Cloud Supabase

1. Create a new project at [supabase.com](https://supabase.com)
2. Go to **Settings → API** and copy your project URL and anon key
3. Link your local project to cloud Supabase:

**Important:** Run these commands **one at a time** in your terminal. Do not copy-paste them all at once.

```bash
# Install Supabase CLI (if not already installed)
# On macOS: brew install supabase/tap/supabase
# Or see: https://supabase.com/docs/guides/cli/getting-started

# Login to Supabase (opens browser for authentication)
supabase login

# Initialize Supabase (creates supabase/ directory if it doesn't exist)
supabase init

# Link to your cloud project (replace <project-id> with your Project ID)
# Find your Project ID in: Supabase Dashboard → Settings → General → Project ID
supabase link --project-ref <project-id>

# Push migrations to cloud Supabase
supabase db push

# Generate TypeScript types from cloud database
supabase gen types typescript --project-id <project-id> > src/lib/database.types.ts
```

4. Update `.env.local` with your cloud Supabase credentials:

```env
NEXT_PUBLIC_SUPABASE_URL=your-cloud-project-url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-cloud-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-cloud-service-role-key
```

#### Step 3: Set Up Vercel (Ready for Deployment)

1. Push your code to GitHub:
   ```bash
   git add .
   git commit -m "Initial setup"
   git push origin main
   ```

2. Import your repository in [Vercel](https://vercel.com)
3. Add environment variables in Vercel project settings:
   - `NEXT_PUBLIC_SUPABASE_URL` (your cloud Supabase URL)
   - `NEXT_PUBLIC_SUPABASE_ANON_KEY` (your cloud Supabase anon key)
   - `SUPABASE_SERVICE_ROLE_KEY` (your cloud Supabase service role key)

4. Configure Supabase redirect URLs:
   - Go to Supabase Dashboard → **Authentication → URL Configuration**
   - Set **Site URL** to your Vercel URL: `https://your-app.vercel.app`
   - Add **Redirect URLs**:
     - `https://your-app.vercel.app/auth/callback`
     - `https://your-app.vercel.app/**`
     - `http://localhost:3000/auth/callback` (for local development)
     - `http://localhost:3000/**`

**✅ Cloud infrastructure is now set up and ready for deployment!**

---

### Phase 2: Develop Locally

Now that cloud infrastructure is ready, develop locally using local Supabase for faster iteration.

#### Step 4: Set Up Local Supabase

**Important:** Run these commands **one at a time** in your terminal.

```bash
# Start local Supabase (runs PostgreSQL, Auth, Storage, etc. in Docker)
supabase start
```

After starting, Supabase will display your local credentials. Update your `.env.local` with local credentials for development:

```bash
# The 'supabase start' output will show:
# - API URL → use for NEXT_PUBLIC_SUPABASE_URL (local)
# - anon key → use for NEXT_PUBLIC_SUPABASE_ANON_KEY (local)
# - service_role key → use for SUPABASE_SERVICE_ROLE_KEY (local)

# Edit .env.local and replace with local credentials for development
```

**Email Authentication:** Local Supabase handles email auth completely! Emails are captured by Inbucket (email testing server) instead of being sent. View captured emails at `http://localhost:54324`. Email verification links work locally and redirect to your local app.

#### Step 5: Apply Database Migrations Locally

```bash
# Apply migrations to your local Supabase instance
supabase db reset

# Generate TypeScript types from your local database
supabase gen types typescript --local > src/lib/database.types.ts
```

#### Step 6: Run the Application Locally

```bash
# Make sure Supabase is running (if not, run: supabase start)
# Then start Next.js with hot reload
npm run dev
```

Visit [http://localhost:3000](http://localhost:3000) to see your app running locally!

**🎉 You're now developing locally!** Make changes, test features, and iterate quickly. Your cloud infrastructure is ready for when you want to deploy.

---

### Phase 3: Deploy to Production

When your code is ready and tested locally, deploy to production:

```bash
# Push your code to GitHub
git add .
git commit -m "Ready for production"
git push origin main
```

Vercel will automatically deploy your changes! Your cloud Supabase and Vercel are already configured, so deployment happens automatically.

**Note:** Before deploying, you may want to push any new migrations to cloud Supabase:
```bash
supabase db push
```

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

This starter uses a **"cloud-ready, local-development"** approach:

### Phase 1: Set Up Cloud Infrastructure First
- Set up cloud Supabase project
- Configure Vercel deployment
- Get everything ready for production deployment

### Phase 2: Develop Locally
- Use local Supabase for development (`supabase start`)
- Develop and test features locally
- Fast iteration with hot reload
- No cloud costs during development

### Phase 3: Deploy When Ready
- Push code to GitHub
- Vercel automatically deploys (already configured!)
- Cloud Supabase is ready (already set up!)

**Benefits:**
- ✅ **Deployment ready** - Infrastructure set up upfront
- ✅ **Fast local development** - No cloud latency
- ✅ **No cloud costs** during development
- ✅ **Confidence** - Deploy when code is ready
- ✅ **Easy deployment** - Just push to GitHub

## 🔐 Environment Variables

Your `.env.local` file should contain:

### For Local Development

Use local Supabase credentials (from `supabase start` output):
- `NEXT_PUBLIC_SUPABASE_URL` - Your local Supabase URL
- `NEXT_PUBLIC_SUPABASE_ANON_KEY` - Your local Supabase anon key
- `SUPABASE_SERVICE_ROLE_KEY` - Your local Supabase service role key

### For Production (Vercel)

Vercel uses cloud Supabase credentials (set in Vercel environment variables):
- `NEXT_PUBLIC_SUPABASE_URL` - Your cloud Supabase URL
- `NEXT_PUBLIC_SUPABASE_ANON_KEY` - Your cloud Supabase anon key
- `SUPABASE_SERVICE_ROLE_KEY` - Your cloud Supabase service role key

**Note:** Switch between local and cloud credentials in `.env.local` as needed. Vercel uses its own environment variables for production.

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📝 License

MIT License - feel free to use this for your projects!

## 🙏 Acknowledgments

Built with experience from production deployments. This starter includes solutions to common issues encountered when building with Next.js, Supabase, and Vercel.

---

**Need help?** Check the [documentation](./docs/) or [open an issue](https://github.com/thechristofferweiss/nextjs-supabase-vercel-starter/issues).

