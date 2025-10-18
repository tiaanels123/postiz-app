# Postiz App - Setup Guide

## Prerequisites

Before you begin, ensure you have the following installed:

- **Node.js** (v22.x) - [Download](https://nodejs.org/)
- **pnpm** (v10.6.1+) - Package manager
- **Docker Desktop** - For PostgreSQL and Redis
- **Git** - For version control

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/postiz-app.git
cd postiz-app
```

### 2. Install pnpm (if not already installed)

```bash
npm install -g pnpm@10.6.1
```

### 3. Create Environment File

Copy the example environment file:

```bash
# Windows (PowerShell)
Copy-Item .env.example .env

# macOS/Linux
cp .env.example .env
```

### 4. Update Database Configuration

Edit `.env` file and update the `DATABASE_URL` to match Docker credentials:

```env
DATABASE_URL="postgresql://postiz-local:postiz-local-pwd@localhost:5432/postiz-db-local"
```

**Key settings in `.env`:**
- `DATABASE_URL` - PostgreSQL connection string
- `REDIS_URL` - Redis connection string (default: redis://localhost:6379)
- `JWT_SECRET` - Change to a secure random string
- `FRONTEND_URL` - Frontend URL (default: http://localhost:4200)
- `NEXT_PUBLIC_BACKEND_URL` - Backend API URL (default: http://localhost:3000)
- `STORAGE_PROVIDER` - Set to "local" for development

### 5. Start Docker Services

Start PostgreSQL, Redis, and admin tools:

```bash
docker compose -f docker-compose.dev.yaml up -d
```

**This will start:**
- PostgreSQL on `localhost:5432`
- Redis on `localhost:6379`
- PgAdmin on `http://localhost:8081` (admin@admin.com / admin)
- RedisInsight on `http://localhost:5540`

**To stop services:**
```bash
docker compose -f docker-compose.dev.yaml down
```

### 6. Install Dependencies

```bash
pnpm install
```

This will:
- Install all npm packages
- Generate Prisma client automatically (via postinstall hook)

### 7. Run Database Migrations

Push the database schema to PostgreSQL:

```bash
pnpm run prisma-db-push
```

### 8. Start the Application

**Option A: Start All Services (Recommended)**

Start backend, frontend, workers, and cron:

```bash
# Windows
pnpm --filter ./apps/backend run dev
pnpm --filter ./apps/frontend run dev
pnpm --filter ./apps/workers run dev
pnpm --filter ./apps/cron run dev
```

Run each command in a separate terminal window.

**Option B: Individual Services**

Start services separately:

```bash
# Backend API (port 3000)
pnpm run dev:backend

# Frontend (port 4200)
pnpm run dev:frontend

# Background Workers
pnpm run dev:workers

# Cron Jobs
pnpm run dev:cron
```

**Note for Windows users:** The scripts use Unix `rm` command which won't work. Use the individual service commands above without the cleanup step.

### 9. Access the Application

- **Frontend**: http://localhost:4200
- **Backend API**: http://localhost:3000
- **PgAdmin**: http://localhost:8081
- **RedisInsight**: http://localhost:5540

## Development Workflow

### Available Scripts

```bash
# Start all services in development mode
pnpm run dev

# Build all apps for production
pnpm run build

# Start production builds
pnpm run start:prod:backend
pnpm run start:prod:frontend
pnpm run start:prod:workers
pnpm run start:prod:cron

# Database commands
pnpm run prisma-generate    # Generate Prisma client
pnpm run prisma-db-push     # Push schema changes to database
pnpm run prisma-reset       # Reset database (⚠️ deletes all data)

# Testing
pnpm run test
```

### Project Structure

```
postiz-app/
├── apps/
│   ├── backend/        # NestJS backend API
│   ├── frontend/       # Next.js frontend
│   ├── workers/        # Background job workers
│   ├── cron/          # Scheduled tasks
│   └── extension/     # Browser extension
├── libraries/
│   └── nestjs-libraries/
│       └── src/database/prisma/  # Prisma schema
├── docker-compose.dev.yaml      # Docker services
├── package.json                 # Root package.json
└── pnpm-workspace.yaml         # Workspace configuration
```

## Troubleshooting

### Port Already in Use

If you get "port already in use" errors:

```bash
# Check what's running on port 3000
netstat -ano | findstr :3000  # Windows
lsof -i :3000                 # macOS/Linux

# Kill the process
taskkill /PID <PID> /F        # Windows
kill -9 <PID>                 # macOS/Linux
```

### Database Connection Issues

1. Ensure Docker containers are running: `docker ps`
2. Check database credentials in `.env` match `docker-compose.dev.yaml`
3. Restart Docker containers:
   ```bash
   docker compose -f docker-compose.dev.yaml restart
   ```

### Prisma Client Issues

Regenerate Prisma client:

```bash
pnpm run prisma-generate
```

### Dependencies Issues

Clean install:

```bash
# Remove node_modules and reinstall
rm -rf node_modules
pnpm install
```

### Windows-Specific Issues

Some scripts use Unix commands (`rm`, `mkdir -p`, etc.). If you encounter errors:

1. Use individual service commands instead of combined scripts
2. Install Git Bash or WSL for better Unix command support
3. Or manually clean dist folders before building

## Environment Variables Reference

### Required Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `DATABASE_URL` | PostgreSQL connection string | - |
| `REDIS_URL` | Redis connection string | redis://localhost:6379 |
| `JWT_SECRET` | Secret for JWT tokens | - |
| `FRONTEND_URL` | Frontend application URL | http://localhost:4200 |
| `NEXT_PUBLIC_BACKEND_URL` | Backend API URL | http://localhost:3000 |

### Optional Variables

| Variable | Description |
|----------|-------------|
| `CLOUDFLARE_*` | Cloudflare R2 storage settings |
| `OPENAI_API_KEY` | OpenAI API key for AI features |
| `STRIPE_*` | Stripe payment settings |
| `*_CLIENT_ID/SECRET` | Social media OAuth credentials |
| `RESEND_API_KEY` | Email service API key |

### Storage Configuration

For development, use local storage:

```env
STORAGE_PROVIDER="local"
```

For production, configure Cloudflare R2:

```env
STORAGE_PROVIDER="cloudflare"
CLOUDFLARE_ACCOUNT_ID="your-account-id"
CLOUDFLARE_ACCESS_KEY="your-access-key"
CLOUDFLARE_SECRET_ACCESS_KEY="your-secret"
CLOUDFLARE_BUCKETNAME="your-bucket"
CLOUDFLARE_BUCKET_URL="https://your-bucket-url.r2.cloudflarestorage.com/"
```

## Production Deployment

### Build for Production

```bash
pnpm run build
```

### Start Production Services

```bash
pnpm run start:prod:backend
pnpm run start:prod:frontend
pnpm run start:prod:workers
pnpm run start:prod:cron
```

### Using PM2 (Process Manager)

```bash
pnpm run pm2
```

### Docker Deployment

For production Docker deployment, see: https://docs.postiz.com/installation/docker-compose

**Do NOT use `docker-compose.dev.yaml` for production!**

## Additional Resources

- **Official Documentation**: https://docs.postiz.com
- **Quick Start Guide**: https://docs.postiz.com/quickstart
- **Configuration Reference**: https://docs.postiz.com/configuration/reference
- **Public API Docs**: https://docs.postiz.com/public-api
- **Discord Community**: https://discord.postiz.com
- **YouTube Tutorials**: https://youtube.com/@postizofficial

## Support

If you encounter issues:

1. Check the [official documentation](https://docs.postiz.com)
2. Search existing [GitHub issues](https://github.com/gitroomhq/postiz-app/issues)
3. Join the [Discord community](https://discord.postiz.com)
4. Create a new issue with detailed error information

## License

This project is licensed under AGPL-3.0. See [LICENSE](LICENSE) file for details.
