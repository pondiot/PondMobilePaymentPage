# Deployment Guide

## Automated Deployment via GitHub Actions

### Deployment Environments

Three environment configurations are available:

| Environment | Config File | CORS Settings | Use Case |
|-------------|-------------|---------------|----------|
| **production** | `deploy/.env.production` | Strict (pondmobile.com only) | Live production deployment |
| **staging** | `deploy/.env.staging` | Relaxed (+localhost) | Testing production API locally |
| **development** | `deploy/.env.development` | Localhost only | Local development |

**Default:** Automatic deploys use `staging` (allows localhost testing)

### Required GitHub Secrets

Configure these secrets in: `Repository Settings > Secrets and variables > Actions`

**Note:** All secrets use `POND_PAYMENT_` prefix to avoid conflicts with other repositories.

| Secret | Description | Example |
|--------|-------------|---------|
| `POND_PAYMENT_SERVER_HOST` | Server IP or domain | `123.45.67.89` or `payment.example.com` |
| `POND_PAYMENT_SERVER_USER` | SSH user | `root` or `deploy` |
| `POND_PAYMENT_SERVER_SSH_KEY` | Private SSH key | Copy contents of `~/.ssh/id_rsa` |
| `POND_PAYMENT_SERVER_PORT` | SSH port (optional) | `22` (default) |
| `POND_PAYMENT_DEPLOY_PATH` | Deployment directory | `/opt/pondmobile-payment` |
| `POND_PAYMENT_REPO_URL` | Git repository URL | `git@github.com:bpdu/PondMobilePaymentPage.git` |

### Setting up SSH Key

```bash
# Generate SSH key on your machine
ssh-keygen -t ed25519 -C "github-deploy" -f ~/.ssh/github_deploy

# Copy PUBLIC key to server
ssh-copy-id -i ~/.ssh/github_deploy.pub user@your-server.com

# Copy PRIVATE key content to GitHub Secret SERVER_SSH_KEY
cat ~/.ssh/github_deploy
```

### Environment Configuration

Three pre-configured environments are available:

**1. Production** (`deploy/.env.production`)
```bash
# Authorize.net credentials (required)
AUTHORIZE_API_LOGIN_ID=your-actual-api-login-id
AUTHORIZE_TRANSACTION_KEY=your-actual-transaction-key

# Production domains only - strict CORS
APP_BASE_URL=https://www.pondmobile.com
ALLOWED_ORIGINS=https://www.pondmobile.com,https://pondmobile.com

FLASK_ENV=production
DOCKER_ENV=true
```

**2. Staging** (`deploy/.env.staging`)
```bash
# Same as production, but allows localhost testing
AUTHORIZE_API_LOGIN_ID=your-actual-api-login-id
AUTHORIZE_TRANSACTION_KEY=your-actual-transaction-key

APP_BASE_URL=https://www.pondmobile.com
ALLOWED_ORIGINS=https://www.pondmobile.com,https://pondmobile.com,http://localhost:5001,http://127.0.0.1:5001

FLASK_ENV=production
DOCKER_ENV=true
```

**3. Development** (`deploy/.env.example`)
```bash
# Local development only
AUTHORIZE_API_LOGIN_ID=test-id
AUTHORIZE_TRANSACTION_KEY=test-key

APP_BASE_URL=http://localhost:5001
ALLOWED_ORIGINS=http://localhost:5001,http://127.0.0.1:5001

FLASK_ENV=development
DOCKER_ENV=true
```

**Setup on server:**
```bash
cd /opt/pondmobile-payment
# Choose which environment to use
cp deploy/.env.staging deploy/.env.active
# Or edit directly
nano deploy/.env.staging
```
```

### Manual Deployment Trigger

1. Go to `Actions` tab in GitHub
2. Select `Deploy to Production`
3. Choose environment:
   - **staging** (default) - Production API with localhost access
   - **production** - Strict production deployment
   - **development** - Localhost only
4. Click `Run workflow` button

**Automatic deployment** (push to main) uses `staging` environment.

---

## Manual Deployment

### Prerequisites

- Docker installed on server
- Git access to repository
- Environment files configured (see below)

### Quick Deploy

```bash
# Clone repo
git clone git@github.com:bpdu/PondMobilePaymentPage.git
cd PondMobilePaymentPage

# Deploy to staging (default - allows localhost testing)
cd deploy
./deploy.sh staging

# Deploy to production (strict CORS)
./deploy.sh production

# Deploy to development (localhost only)
./deploy.sh development
```

### Manual Docker Commands

```bash
# Build image with version
docker build \
  -f deploy/Dockerfile \
  --build-arg BUILD_VERSION=$(git rev-parse --short HEAD) \
  -t pondmobile-payment:latest \
  .

# Run container with staging environment (allows localhost)
docker run -d \
  --name pondmobile-payment \
  --restart unless-stopped \
  -p 5001:5001 \
  -v pondmobile-logs:/app/logs \
  --env-file deploy/.env.staging \
  pondmobile-payment:latest

# Or with production environment (strict CORS)
docker run -d \
  --name pondmobile-payment \
  --restart unless-stopped \
  -p 5001:5001 \
  -v pondmobile-logs:/app/logs \
  --env-file deploy/.env.production \
  pondmobile-payment:latest
```

---

## Health Check

After deployment, verify the service:

```bash
curl http://localhost:5001/health
# Response: {"status": "healthy [abc1234]"}
```

The response includes:
- `healthy` - service status
- `[abc1234]` - short SHA of deployed commit

---

## Logs

View application logs:

```bash
# Docker logs
docker logs -f pondmobile-payment

# Persistent logs (in volume)
docker exec pondmobile-payment tail -f /app/logs/access.log
docker exec pondmobile-payment tail -f /app/logs/security.log
```

---

## Rollback

To rollback to previous version:

```bash
# List available versions
docker images pondmobile-payment

# Run specific version
docker stop pondmobile-payment
docker rm pondmobile-payment
docker run -d \
  --name pondmobile-payment \
  --restart unless-stopped \
  -p 5001:5001 \
  -v pondmobile-logs:/app/logs \
  --env-file deploy/.env \
  pondmobile-payment:abc1234
```
