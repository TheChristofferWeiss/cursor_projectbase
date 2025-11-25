# Next.js + Supabase + Vercel Starter Kit

A production-ready starter template for building full-stack applications with Next.js, Supabase, and Vercel. 

**Workflow:** Set up cloud Supabase connection first (so deployment is ready), then develop locally using local Supabase. When your code is ready, deploy to production with confidence!

## 🚀 Quick Start

### Prerequisites

- Node.js 18+ and npm
- [Docker](https://www.docker.com/get-started) and Docker Compose
  - **Installation:** Download [Docker Desktop](https://www.docker.com/products/docker-desktop/) for your operating system
  - **macOS:** `brew install --cask docker` or download from docker.com
  - **Windows:** Download Docker Desktop from docker.com
  - **Linux:** Follow [Docker installation guide](https://docs.docker.com/engine/install/)
  - **Verify:** Run `docker --version` and `docker-compose --version` to confirm installation
- [Supabase CLI](https://supabase.com/docs/guides/cli/getting-started)
- A Supabase account ([sign up free](https://supabase.com))
- A Vercel account ([sign up free](https://vercel.com)) - for deployment
- Git

### Step 1: Clone and Setup

**a.** Create your project directory and navigate to it:
   ```bash
   # Create your project directory (replace 'your-project-name' with your actual project name)
   mkdir your-project-name
   
   # Navigate into the directory
   cd your-project-name
   ```
   
   **In Cursor:** You can also create the folder using File → New Folder, then open it in Cursor.

**b.** Clone this repository directly into your project folder:
   ```bash
   # Clone directly into current directory (note the . at the end)
   git clone https://github.com/thechristofferweiss/nextjs-supabase-vercel-starter.git .
   ```
   The `.` tells git to clone into the current directory instead of creating a nested subfolder.

**c.** Set up your own Git repository (the cloned repo is connected to the starter template's repository):
   
   **Important:** Create a new GitHub repository for your project first, then run the commands below **one at a time**. Copy only the command lines (`git ...`) into your terminal:
   
   ```bash
   git remote remove origin
   ```
   
   ```bash
   git remote add origin https://github.com/yourusername/your-project-name.git
   ```
   
   ```bash
   git remote -v
   ```

**d.** Install dependencies:
   ```bash
npm install
   ```

**e.** Copy environment template:
   ```bash
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

**e.** Initialize Supabase (creates `supabase/` directory). **Skip this if the folder already contains `supabase/config.toml` (i.e., you cloned this starter):**
   ```bash
   supabase init
   ```

**f.** Link to your cloud project (replace `<project-id>` with your Project ID):
   ```bash
   # Find your Project ID in: Supabase Dashboard → Settings → General → Project ID
   supabase link --project-ref <project-id>
   ```
   
   **Important:** After linking, you may see a warning about database version mismatch. If so, check your cloud PostgreSQL version:
   - Go to Supabase Dashboard → **Settings → Database**
   - Note the PostgreSQL version (e.g., 17.x)
   - Update `supabase/config.toml`: change `major_version = 15` to match your cloud version (e.g., `major_version = 17`)

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

**a.** Verify your Git repository is set up correctly:
   ```bash
   # Check that your remote is pointing to your repository (not the starter template)
   git remote -v
   ```
   You should see your repository URL, not the starter template URL. If you see the starter template URL, go back to Step 1c.

**b.** Ensure your GitHub repository exists:
   - Go to [GitHub](https://github.com) and verify your repository exists
   - If it doesn't exist, create it now (don't initialize with README, .gitignore, or license)

**c.** Push your code to GitHub:
   ```bash
   git add .
   git commit -m "Initial setup"
   git push -u origin main
   ```
   
   **If push fails:** Make sure your GitHub repository exists and you have push access. If you get "repository not found", verify the repository URL in Step 1c.

**d.** Import your repository in [Vercel](https://vercel.com):
   - Go to [vercel.com](https://vercel.com) and sign in
   - Click **"Add New..."** → **"Project"**
   - Import your GitHub repository (e.g., `yourusername/yourprojectname`)
   - Vercel will auto-detect Next.js settings

**e.** Add environment variables in Vercel project settings:
   
   **Important:** Use your **cloud** Supabase credentials (NOT the local ones from `.env.local`). Your `.env.local` file has local development credentials (`http://127.0.0.1:54321`), but Vercel needs production/cloud credentials.
   
   - Go to Vercel project settings → **Environment Variables**
   - Get the values from your Supabase Dashboard:
     - **URL:** Go to **Settings → Data API** and copy the **Project URL**
     - **Keys:** Go to **Settings → API Keys** and copy:
       - `anon` `public` key (for `NEXT_PUBLIC_SUPABASE_ANON_KEY`)
       - `service_role` `secret` key (for `SUPABASE_SERVICE_ROLE_KEY`)
   - Add these variables in Vercel:
     - `NEXT_PUBLIC_SUPABASE_URL` - Your cloud Supabase project URL (from Data API)
     - `NEXT_PUBLIC_SUPABASE_ANON_KEY` - Your cloud Supabase anon/public key (from API Keys)
     - `SUPABASE_SERVICE_ROLE_KEY` - Your cloud Supabase service role key (from API Keys)
   
   **Tip:** You can use Vercel's "Import .env" button if you create a `.env` file with your cloud credentials, or add them manually.

**f.** Configure Supabase redirect URLs:
   - Go to Supabase Dashboard → **Authentication → URL Configuration**
   - Set **Site URL** to your Vercel URL: `https://your-app.vercel.app`
   - Add **Redirect URLs**:
     - `https://your-app.vercel.app/auth/callback`
     - `https://your-app.vercel.app/**`
     - `http://localhost:3000/auth/callback` (for local development)
     - `http://localhost:3000/**`

**✅ Vercel is now configured!** When you push to GitHub, Vercel will automatically deploy.

**Note:** During deployment, you may see yellow or red lines in the Vercel build logs. Don't be alarmed - these are often warnings or informational messages. The deployment will complete successfully as long as the build finishes.

**What to expect:** Once deployed, your site will show a landing page with:
- Title: "Next.js + Supabase + Vercel Starter"
- A "Get Started" card with Sign In and Sign Up buttons
- A Features section listing the key features
- Authentication pages for sign in and sign up

---

### Step 4: Set Up Local Supabase for Development

Now set up local Supabase - this is where you'll develop and test your application.

**Important:** Run these commands **one at a time** (copy only the lines that start with actual commands, never the explanatory text).

1. Start the local stack (PostgreSQL, Auth, Storage, etc.):
   ```bash
   supabase start
   ```

2. If you see a port conflict, diagnose and restart:
   - List running Supabase containers:
     ```bash
     docker ps --filter "name=supabase"
     ```
   - Stop the conflicting project (replace `<project-id>` with the one shown in the previous command):
     ```bash
     supabase stop --project-id <project-id>
     ```
   - Start again:
     ```bash
     supabase start
     ```

After `supabase start` finishes, it prints the local credentials you need for development. Update `.env.local` with:

- `NEXT_PUBLIC_SUPABASE_URL` → the printed **API URL**
- `NEXT_PUBLIC_SUPABASE_ANON_KEY` → the printed **Publishable key**
- `SUPABASE_SERVICE_ROLE_KEY` → the printed **Secret key**

Edit `.env.local` and replace the placeholder values with these local credentials.

**Email Authentication:** Local Supabase handles email auth via Inbucket (email testing server). View captured emails at `http://localhost:54324`. Email verification links work locally and redirect to your local app.

### Step 5: Apply Database Migrations Locally

1. Apply migrations to your local Supabase instance:
   ```bash
   supabase db reset
   ```

2. Generate TypeScript types from the local database:
   ```bash
   supabase gen types typescript --local > src/lib/database.types.ts
   ```

### Step 6: Run the Application Locally

Start the Next.js dev server (ensure Supabase is still running in another terminal):
```bash
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
You can also contact me directly at indigovira@gmail.com

