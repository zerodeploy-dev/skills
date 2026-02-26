---
name: zerodeploy-setup
description: >
  Set up ZeroDeploy for a project. Installs CLI, authenticates,
  and initializes zerodeploy.json config.
disable-model-invocation: true
allowed-tools: Bash(zerodeploy *) Bash(npm install *) Bash(npx *) Bash(which *)
---

# Set Up ZeroDeploy

Install the ZeroDeploy CLI, authenticate, and initialize the project config.

## Steps

### 1. Install CLI

```bash
which zerodeploy
```

If not found:

```bash
npm install -g @zerodeploy/cli
```

### 2. Authenticate

```bash
zerodeploy whoami --json
```

If exit code 1 (not authenticated):

```bash
zerodeploy login
```

This opens a browser for GitHub OAuth. Wait for the user to complete login.

### 3. Initialize project

If `zerodeploy.json` doesn't exist yet:

```bash
zerodeploy init
```

This will:
- Auto-detect the org (or use the personal org)
- Create a site using the folder name
- Detect the build output directory
- Save config to `zerodeploy.json`

If the user wants a specific site:

```bash
zerodeploy init --site <site>
```

Other init options:
- `--dir <directory>` — Build output directory
- `--build <command>` — Build command
- `--ignore <patterns...>` — Glob patterns to ignore during deploy
- `--force` — Overwrite existing config file

### 4. Verify setup

```bash
zerodeploy whoami --json
```

Report:
- Logged in as: username
- Organization: org slug
- Site: site slug
- Ready to deploy

## Config File

The generated `zerodeploy.json` looks like:

```json
{
  "org": "my-org",
  "site": "my-site",
  "dir": "dist"
}
```

Optional fields:
- `"build": "npm run build"` — Build command
- `"install": "npm install"` — Install command
- `"ignore": ["*.map", "drafts/"]` — Ignore patterns

## Next Steps

After setup, the user can deploy with:

```bash
zerodeploy deploy
```

Or with build:

```bash
zerodeploy deploy --build
```

## Additional context from user
$ARGUMENTS
