---
name: doctl
description: DigitalOcean CLI (doctl) command reference for App Platform operations. Use when user asks to check deployment status, view app logs, list apps/deployments, trigger redeployments, or any doctl/DigitalOcean-related task. Triggers on mentions of "doctl", "deployment", "deploy status", "app logs", "redeploy", "DigitalOcean", or checking if a release is live.
---

# doctl — DigitalOcean App Platform

Quick reference for `doctl apps` commands. For app IDs and component names, read [references/apps.md](references/apps.md).

## Check Deployment Status

```bash
# Latest deployment for an app
doctl apps list-deployments <app-id> --format ID,Cause,Progress,Phase,Created,Updated

# Specific deployment
doctl apps get-deployment <app-id> <deployment-id>
```

Deployment is complete when **Phase = ACTIVE** and **Progress = 18/18** (or N/N).

## View Logs

```bash
# Run logs (default) — live app output
doctl apps logs <app-id> <component> -f

# Build logs
doctl apps logs <app-id> <component> --type build

# Deploy logs
doctl apps logs <app-id> <component> --type deploy

# Last N lines
doctl apps logs <app-id> <component> --tail 100
```

Omit `<component>` to see all components. Use `-f` to follow (tail).

Log types: `run` (default), `build`, `deploy`, `run_restarted`.

## Trigger Redeployment

```bash
# Redeploy with latest source/image
doctl apps create-deployment <app-id>

# Force rebuild (skip cache)
doctl apps create-deployment <app-id> --force-rebuild

# Wait for completion (blocks terminal)
doctl apps create-deployment <app-id> --wait
```

## List Apps

```bash
doctl apps list
```

## Get App Spec

```bash
doctl apps spec get <app-id>                  # YAML
doctl apps spec get <app-id> --format json    # JSON
```
