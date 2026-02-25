---
name: deploy
description: >
  Deploy a static site or SPA to ZeroDeploy. Use when the user asks to
  deploy, publish, ship, or host a frontend project. Handles first-time
  setup (CLI install, auth) and subsequent deploys.
disable-model-invocation: true
allowed-tools: Bash(zerodeploy *) Bash(npm install *) Bash(npx *) Bash(which *)
---

# Deploy to ZeroDeploy

Deploy the current project as a static site to ZeroDeploy.

## Prerequisites

Check each prerequisite and fix if needed:

### 1. CLI installed?

```bash
which zerodeploy
```

If not found:

```bash
npm install -g @zerodeploy/cli
```

### 2. Logged in?

```bash
zerodeploy whoami --json
```

If exit code 1 (not authenticated):

```bash
zerodeploy login
```

This opens a browser for GitHub OAuth. Wait for the user to complete login.

### 3. Config exists?

Check if `zerodeploy.json` exists in the project root. If not, the CLI will auto-create
a site on first deploy using the folder name — no prior setup needed.

## Deploy

Run the deploy command with JSON output for structured results:

```bash
zerodeploy deploy --json
```

Or specify a site explicitly:

```bash
zerodeploy deploy my-site --json
```

Parse the JSON output:

**Success:**
```json
{
  "success": true,
  "deployment_id": "abc123",
  "url": "https://my-site.zerodeploy.app",
  "preview_url": "https://abc123-my-site.zerodeploy.app",
  "is_preview": false,
  "file_count": 42,
  "total_size_bytes": 1048576,
  "site": "my-site"
}
```

**Error:**
```json
{
  "success": false,
  "error": "not_authenticated",
  "message": "Not logged in. Run: zerodeploy login",
  "exit_code": 1
}
```

## Options

- `[site]` — Site slug (positional argument, optional if zerodeploy.json exists)
- `--org <org>` — Organization slug
- `--dir <path>` — Directory to deploy (default: auto-detect from framework)
- `--build` — Run build command before deploying
- `--no-build` — Skip build step
- `--build-command <command>` — Override build command
- `--install` — Run install command before building
- `--preview` — Create a preview deployment (not set as current)
- `--prod` — Force production deploy (override auto-preview for PRs)
- `--append` — Add files to existing deployment instead of replacing
- `--no-verify` — Skip deployment verification
- `--no-auto-rollback` — Disable automatic rollback on verification failure

## Error Handling

Handle errors by exit code:

| Exit Code | Meaning | Action |
|-----------|---------|--------|
| 0 | Success | Report the URL to the user |
| 1 | Auth error | Run `zerodeploy login` |
| 2 | Not found | Check org/site slugs |
| 3 | Validation error | Fix the input (check error message) |
| 4 | Rate limited | Wait a moment and retry |
| 5 | Server error | Retry, or report to user |
| 6 | Network error | Check internet connection |
| 7 | Billing error | User needs to add balance |

## First Deploy (No Config)

If there's no `zerodeploy.json` and the user hasn't set up a site yet:

1. The CLI will auto-create a site using the folder name as the slug
2. It saves the config to `zerodeploy.json` automatically
3. No manual setup needed — just run `zerodeploy deploy --json`

## Build and Deploy

If the project needs to be built first:

```bash
zerodeploy deploy --build --json
```

Or build and install dependencies:

```bash
zerodeploy deploy --build --install --json
```

## Additional context from user
$ARGUMENTS
