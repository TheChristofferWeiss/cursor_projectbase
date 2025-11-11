# Next.js + Supabase + Vercel Starter Kit

A production-ready starter template and comprehensive tutorial for building full-stack applications with Next.js, Supabase, and Vercel. This repository provides everything you need to quickly bootstrap a new project with authentication, database, and deployment configured.

## 🚀 Quick Start

### Prerequisites

- Node.js 18+ and npm
- A Supabase account ([sign up free](https://supabase.com))
- A Vercel account ([sign up free](https://vercel.com))
- Git

### 1. Clone and Setup

```bash
# Clone this repository
git clone https://github.com/thechristofferweiss/nextjs-supabase-vercel-starter.git
cd nextjs-supabase-vercel-starter

# Install dependencies
npm install

# Copy environment template
cp .env.example .env.local
```

### 2. Supabase Setup

1. Create a new project at [supabase.com](https://supabase.com)
2. Go to **Settings → API** and copy your project URL and anon key
3. Update `.env.local` with your Supabase credentials:

```env
NEXT_PUBLIC_SUPABASE_URL=your-project-url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
```

4. Link your local project to Supabase:

```bash
# Install Supabase CLI (if not already installed)
npm install -g supabase

# Login to Supabase
supabase login

# Initialize Supabase
supabase init

# Link to your project (replace <project-ref> with your project reference)
supabase link --project-ref <project-ref>
```

5. Run database migrations:

```bash
supabase db push
```

6. Generate TypeScript types:

```bash
supabase gen types typescript --project-id <project-ref> > src/lib/database.types.ts
```

### 3. Run Locally

```bash
npm run dev
```

Visit [http://localhost:3000](http://localhost:3000) to see your app.

### 4. Deploy to Vercel

1. Push your code to GitHub
2. Import your repository in [Vercel](https://vercel.com)
3. Add environment variables in Vercel project settings:
   - `NEXT_PUBLIC_SUPABASE_URL`
   - `NEXT_PUBLIC_SUPABASE_ANON_KEY`
   - `SUPABASE_SERVICE_ROLE_KEY`
4. Configure Supabase redirect URLs:
   - Go to Supabase Dashboard → Authentication → URL Configuration
   - Add your Vercel URL: `https://your-app.vercel.app/auth/callback`
   - Add localhost: `http://localhost:3000/auth/callback`

### 5. Docker Support (Optional)

This starter also includes Docker support for running everything locally:

```bash
# Start full stack (Next.js + Supabase) with Docker Compose
npm run docker:up

# View logs
npm run docker:logs

# Stop services
npm run docker:down
```

See the [Docker Guide](./docs/docker.md) for complete Docker setup instructions.

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
- ✅ **Deployment Ready** - Configured for Vercel deployment
- ✅ **Local Development** - Hot reload with live Supabase backend
- ✅ **Docker Support** - Optional Docker setup for full local development

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

This starter uses a **"local frontend, live backend"** approach by default:

- **Frontend**: Run locally with `npm run dev` for instant feedback
- **Backend**: Connect directly to your cloud Supabase project
- **Deploy**: Push to GitHub and Vercel auto-deploys

**Alternative**: Use Docker for fully local development (see [Docker Guide](./docs/docker.md))

## 🔐 Environment Variables

### Required

- `NEXT_PUBLIC_SUPABASE_URL` - Your Supabase project URL
- `NEXT_PUBLIC_SUPABASE_ANON_KEY` - Your Supabase anon/public key

### Optional (for admin operations)

- `SUPABASE_SERVICE_ROLE_KEY` - Service role key (server-side only!)
- `NEXT_PUBLIC_SITE_URL` - Your production URL (for email links)

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📝 License

MIT License - feel free to use this for your projects!

## 🙏 Acknowledgments

Built with experience from production deployments. This starter includes solutions to common issues encountered when building with Next.js, Supabase, and Vercel.

---

**Need help?** Check the [documentation](./docs/) or [open an issue](https://github.com/yourusername/nextjs-supabase-vercel-starter/issues).

