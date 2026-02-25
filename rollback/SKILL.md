---
name: rollback
description: >
  Rollback a ZeroDeploy deployment to a previous version.
  Use when the user wants to undo a deployment or revert to an
  earlier version of their site.
disable-model-invocation: true
allowed-tools: Bash(zerodeploy *)
---

# Rollback ZeroDeploy Deployment

Rollback to a previous deployment version.

## Steps

### 1. Read config

Check if `zerodeploy.json` exists in the project root. Extract `org` and `site` values.
If not found, ask the user for the org and site slugs.

### 2. List recent deployments

```bash
zerodeploy deployments list --json
```

(Site is resolved automatically from `zerodeploy.json`. Use positional `<site>` to override.)

Show the user the recent deployments so they can choose which one to rollback to.

### 3. Rollback

If the user specified a deployment ID:

```bash
zerodeploy rollback --to <deployment-id>
```

Otherwise, rollback to the previous deployment:

```bash
zerodeploy rollback
```

### 4. Verify

After rollback, confirm the active deployment has changed by checking the site URL.

## Error Handling

| Exit Code | Meaning | Action |
|-----------|---------|--------|
| 1 | Auth error | Run `zerodeploy login` |
| 2 | Not found | Check org/site slugs |
| 3 | Validation error | No previous deployment exists |

## User request
$ARGUMENTS
