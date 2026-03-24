# DigitalOcean Apps — xfeedback

## App Registry

| Alias | App Name | App ID | URL |
|-------|----------|--------|-----|
| `tenant` | xfeedback-tenant | `408c788a-80ca-45c2-a5ad-11ec136fc5ea` | bridz.io (wildcard) |
| `app` | xfeedback-app | `74cfa0cf-ed4f-4611-ab0c-cb0db7481948` | xfeedback-app-eyap6.ondigitalocean.app |
| `homepage` | xfeedback-homepage | `72d759df-1de2-4e93-9b84-4b04bf08eb09` | xfeedback-homepage-zggct.ondigitalocean.app |

## Component Map

### xfeedback-tenant (main multi-component app)

| Type | Component Name | Notes |
|------|----------------|-------|
| service | `xfeedback-api` | Laravel API, Docker image from `xownego/xfeedback-api` |
| static_site | `xfeedback-app` | Vue storefront |
| static_site | `xcanny-admin` | Vue admin panel |
| static_site | `xfeedback-sdk` | JS SDK |
| static_site | `xfeedback-homepage` | Marketing site |
| static_site | `xfeedback-docs` | Documentation site |
| worker | `xfeedback-worker` | Queue worker |
| worker | `xfeedback-cronjob` | Scheduled tasks |

### xfeedback-app (standalone)

| Type | Component Name |
|------|----------------|
| static_site | `xfeedback-app` |

### xfeedback-homepage (standalone)

| Type | Component Name |
|------|----------------|
| static_site | `xfeedback-homepage` |
| static_site | `xfeedback-admin` |

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
- **Docker image push**: `xownego/xfeedback-api:latest` → triggers tenant app
- **Git push to master**: `qikify/xcanny-admin`, `qikify/xcanny-app` → triggers tenant app
- **GitHub release**: Creates a tag, which triggers the image build + push pipeline
