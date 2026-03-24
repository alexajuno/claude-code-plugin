# DigitalOcean Apps — <project-name>

## App Registry

| Alias | App Name | App ID | URL |
|-------|----------|--------|-----|
| `tenant` | my-tenant-app | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` | example.com |
| `app` | my-app | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` | my-app.ondigitalocean.app |
| `homepage` | my-homepage | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` | my-homepage.ondigitalocean.app |

## Component Map

### my-tenant-app (main multi-component app)

| Type | Component Name | Notes |
|------|----------------|-------|
| service | `my-api` | API service, Docker image from `user/my-api` |
| static_site | `my-app` | Frontend app |
| static_site | `my-admin` | Admin panel |
| worker | `my-worker` | Queue worker |
| worker | `my-cronjob` | Scheduled tasks |

### my-app (standalone)

| Type | Component Name |
|------|----------------|
| static_site | `my-app` |

## Deployment Phases

| Phase | Meaning |
|-------|---------|
| `ACTIVE` | Currently running deployment |
| `BUILDING` | Building components |
| `DEPLOYING` | Deploying built components |
| `SUPERSEDED` | Replaced by a newer deployment |
| `CANCELED` | Deployment was canceled |
| `ERROR` | Deployment failed |

## Deployment Triggers

Deployments are triggered by:
- **Docker image push**: `user/my-api:latest` → triggers tenant app
- **Git push to master**: source repos → triggers tenant app
- **GitHub release**: Creates a tag, which triggers the image build + push pipeline
