# Docker Setup Guide

This guide explains how to run the Next.js + Supabase application using Docker. Docker provides an alternative to the default "local frontend, cloud backend" workflow, allowing you to run everything locally in containers.

## When to Use Docker

Docker is useful when you want to:
- **Run everything locally** - Full local development environment without cloud dependencies
- **Test in isolation** - Isolated environment that matches production
- **Deploy to other platforms** - Self-hosted deployments or platforms like Railway, Render, etc.
- **Team consistency** - Ensure everyone has the same development environment
- **CI/CD pipelines** - Containerized builds and tests

## Prerequisites

- [Docker](https://www.docker.com/get-started) installed (Docker Desktop recommended)
- [Docker Compose](https://docs.docker.com/compose/install/) (usually included with Docker Desktop)

## Quick Start

### Option 1: Next.js with Docker Compose

This runs Next.js in Docker while connecting to your Supabase (cloud or local):

```bash
# Make sure .env.local is configured with your Supabase credentials
# Then start the Next.js container
npm run docker:up

# View logs
npm run docker:logs

# Stop services
npm run docker:down
```

The application will be available at:
- **Next.js App**: http://localhost:3000

**Note**: For local Supabase, use `supabase start` separately (Supabase CLI handles Docker internally).

### Option 2: Next.js Only in Docker

If you want to run only Next.js in Docker but connect to cloud Supabase:

```bash
# Build the Docker image
DOCKER_BUILD=true npm run build

# Build Docker image
docker build -t nextjs-supabase-app .

# Run container
docker run -p 3000:3000 \
  -e NEXT_PUBLIC_SUPABASE_URL=your-cloud-url \
  -e NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key \
  nextjs-supabase-app
```

## Configuration

### Environment Variables

Create a `.env.local` file (or set environment variables) with your Supabase credentials:

```env
NEXT_PUBLIC_SUPABASE_URL=http://localhost:54321
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
```

**Note**: When using Docker Compose with local Supabase, you'll need to:
1. Initialize Supabase locally: `supabase start`
2. Get the local keys from `supabase status`
3. Update your `.env.local` with local Supabase credentials

### Docker Compose Services

The `docker-compose.yml` includes:

- **supabase**: PostgreSQL database (port 54322)
- **supabase-studio**: Database management UI (port 54323) - optional
- **nextjs**: Next.js application (port 3000)

### Customizing Docker Compose

You can modify `docker-compose.yml` to:
- Change ports
- Add additional services
- Configure volumes
- Set resource limits

## Development Workflow

### Using Docker Compose

1. **Configure environment**:
   ```bash
   # Ensure .env.local exists with Supabase credentials
   cp .env.example .env.local
   # Edit .env.local with your Supabase URL and keys
   ```

2. **Start Next.js container**:
   ```bash
   npm run docker:up
   ```

3. **Access the application**:
   - App: http://localhost:3000

4. **View logs**:
   ```bash
   npm run docker:logs
   # Or for specific service:
   docker-compose logs -f nextjs
   ```

5. **Stop services**:
   ```bash
   npm run docker:down
   ```

**Note**: If you want to run Supabase locally, use `supabase start` separately (before or after starting Docker Compose).

### Hot Reloading

For development with hot reloading, run Next.js locally (not in Docker):

```bash
# Start Supabase locally (uses Docker internally)
supabase start

# Run Next.js locally (connects to local Supabase)
npm run dev
```

Docker is best used for production-like testing or deployment scenarios.

## Production Build

### Building for Production

```bash
# Build with Docker support
DOCKER_BUILD=true npm run build

# Build Docker image
docker build -t nextjs-supabase-app:latest .
```

### Running Production Container

```bash
docker run -d \
  -p 3000:3000 \
  --name nextjs-app \
  -e NEXT_PUBLIC_SUPABASE_URL=your-production-url \
  -e NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key \
  nextjs-supabase-app:latest
```

## Troubleshooting

### Port Already in Use

If ports 3000, 54322, or 54323 are already in use:

1. Stop the conflicting service
2. Or modify ports in `docker-compose.yml`:
   ```yaml
   ports:
     - "3001:3000"  # Use 3001 instead of 3000
   ```

### Database Connection Issues

If Next.js can't connect to Supabase:

1. Check Supabase is running: `docker-compose ps`
2. Verify environment variables are set correctly
3. Check logs: `docker-compose logs supabase`

### Build Failures

If Docker build fails:

1. Ensure `DOCKER_BUILD=true` is set during build
2. Check Next.js config has `output: 'standalone'` enabled
3. Verify all dependencies are in `package.json`

### Volume Issues

If you need to reset the database:

```bash
# Remove volumes (WARNING: deletes all data)
npm run docker:clean

# Start fresh
npm run docker:up
```

## Docker vs Default Workflow

### Default Workflow (Recommended for Most Users)
- ✅ Faster setup
- ✅ No Docker required
- ✅ Connects to cloud Supabase (free tier available)
- ✅ Easy to share with team
- ❌ Requires internet connection
- ❌ Uses cloud resources

### Docker Workflow
- ✅ Fully local (no internet needed after setup)
- ✅ Isolated environment
- ✅ Matches production setup
- ✅ Good for CI/CD
- ❌ Requires Docker installation
- ❌ More complex setup
- ❌ Uses local resources

## Best Practices

1. **Use `.dockerignore`** - Already configured to exclude unnecessary files
2. **Multi-stage builds** - Dockerfile uses multi-stage builds for smaller images
3. **Environment variables** - Never commit secrets, use `.env.local`
4. **Health checks** - Docker Compose includes health checks for services
5. **Volume persistence** - Database data persists in Docker volumes

## Additional Resources

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Next.js Docker Deployment](https://nextjs.org/docs/deployment#docker-image)
- [Supabase Local Development](https://supabase.com/docs/guides/cli/local-development)

