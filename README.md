# GitHub Actions POC - Go API

REST API in Go with CI/CD using GitHub Actions, Docker image builds and multi-environment deployments.

## Project Structure

```
github-actions-poc/
├── .github/
│   └── workflows/
│       ├── develop.yml
│       └── release.yml
├── cmd/
│   └── api/
│       └── main.go
├── internal/
│   └── handlers/
│       └── environment.go
├── .env.example
├── .gitignore
├── Dockerfile
├── go.mod
└── README.md
```

## API

### GET /environment

Returns current environment information.

**Response:**
```json
{
  "environment": "development",
  "version": "1.0.0"
}
```

## Local Development

### Prerequisites
- Go 1.21+
- Docker (optional)

### Run locally

```bash
cp .env.example .env
go run cmd/api/main.go
curl http://localhost:8080/environment
```

### Run with Docker

```bash
docker build -t api-environment:local .
docker run -p 8080:8080 -e ENVIRONMENT=development -e VERSION=1.0.0 api-environment:local
curl http://localhost:8080/environment
```

## CI/CD

### Workflow: develop.yml

**Trigger:** Push to `develop` branch

- Build Docker image
- Push to `ghcr.io/<owner>/api-environment:develop`

### Workflow: release.yml

**Trigger:** Tag `v*` creation on `main` branch

**Jobs:**

1. **build-and-push**: Build and push image with tag version
2. **deploy-development**: Automatic deploy to development
3. **deploy-qa**: Deploy to QA
4. **deploy-staging**: Deploy to staging (requires approval)
5. **deploy-production**: Deploy to production (requires approval, depends on staging)

## GitHub Configuration

### 1. Environments

In **Settings > Environments**, create:

| Environment | Protection |
|-------------|------------|
| development | - |
| qa | - |
| staging | Required reviewers |
| production | Required reviewers |

### 2. Actions Permissions

In **Settings > Actions > General**:

- Workflow permissions: **Read and write permissions**

### 3. GitHub Pages

In **Settings > Pages**:

- Source: **Deploy from a branch**
- Branch: **gh-pages**

## GitHub Pages

After deployments, pages will be available at:

- `https://<username>.github.io/<repo>/development/`
- `https://<username>.github.io/<repo>/qa/`
- `https://<username>.github.io/<repo>/staging/`
- `https://<username>.github.io/<repo>/production/`

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| ENVIRONMENT | Environment name | development |
| VERSION | Application version | 1.0.0 |
| PORT | Server port | 8080 |
