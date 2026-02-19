---
name: zerodeploy-status
description: >
  Check ZeroDeploy deployment status, analytics, and site info.
  Use when the user asks about deployment status, site traffic,
  or wants to see recent deployments.
allowed-tools: Bash(zerodeploy *)
---

# Check ZeroDeploy Status

Show deployment status, recent deployments, and analytics for a ZeroDeploy site.

## Steps

### 1. Read config

Check if `zerodeploy.json` exists in the project root. Extract `org` and `site` values.
If not found, ask the user for the org and site names.

### 2. Current deployment

```bash
zerodeploy deployments list <site> --org <org> --json
```

The first entry is the current (active) deployment. Report:
- Deployment ID
- Status (ready, pending, failed)
- URL
- When it was deployed (created_at)
- File count and size

### 3. Site analytics (optional)

If the user asks about traffic:

```bash
zerodeploy site stats <site> --org <org> --period 7d
```

### 4. Site details

For more info about the site:

```bash
zerodeploy site list <org> --json
```

## Summarize

Report to the user:
- Current deployment URL and status
- When it was last deployed
- Traffic stats (if requested)
- Any issues (failed deployments, etc.)

## Error Handling

| Exit Code | Meaning | Action |
|-----------|---------|--------|
| 1 | Auth error | Run `zerodeploy login` |
| 2 | Not found | Check org/site names in zerodeploy.json |

## Specific request
$ARGUMENTS
