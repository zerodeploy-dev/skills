---
name: zerodeploy-drop
description: >
  Deploy temporary sites to ZeroDeploy with one HTTP call. No auth, no CLI,
  no account required. Use for quick prototypes, demos, or one-off deploys.
  Sites expire in 72 hours unless claimed.
disable-model-invocation: true
allowed-tools: Bash(curl *) Bash(tar *)
---

# Deploy with Drop API (No Auth)

Deploy a static site to ZeroDeploy using the Drop API — zero authentication, one HTTP call. Perfect for quick demos, prototypes, or temporary sites.

## When to Use Drop API vs CLI

- **Drop API**: Quick one-off deploys, prototypes, demos, temporary sites. Zero setup. Sites expire in 72 hours.
- **CLI**: Ongoing projects, CI/CD, custom domains, analytics, rollbacks. Requires login. See `/zerodeploy-deploy` skill.

## Single File Deploy

For a single HTML file:

```bash
curl -X POST https://api.zerodeploy.dev/drop \
  -H "Content-Type: text/html" \
  --data-binary @index.html
```

For other file types (JS, CSS, PDF, images), use `X-Filename` header:

```bash
curl -X POST https://api.zerodeploy.dev/drop \
  -H "Content-Type: application/javascript" \
  -H "X-Filename: app.js" \
  --data-binary @app.js
```

## Multi-File Site Deploy

Package the build output as tar.gz and deploy:

```bash
# From build directory (dist/, out/, build/, etc.)
tar -czf site.tar.gz -C ./dist .

# Deploy the archive
curl -X POST https://api.zerodeploy.dev/drop \
  -H "Content-Type: application/gzip" \
  --data-binary @site.tar.gz
```

**Important**: Use `-C ./dist .` (note the trailing `.`) to ensure files are extracted to the root, not in a subdirectory.

## Response Format

The API returns a self-documenting JSON response:

```json
{
  "data": {
    "url": "https://shy-hill-9433.zerodeploy.app",
    "claim_token": "zdr_abc123...",
    "expires_at": "2026-03-10T12:00:00Z"
  },
  "next_steps": {
    "redeploy": {
      "description": "Update this site with new files",
      "method": "POST",
      "url": "https://api.zerodeploy.dev/drop",
      "headers": {
        "Content-Type": "application/gzip",
        "X-Claim-Token": "zdr_abc123..."
      }
    },
    "claim": {
      "description": "Make this site permanent (requires login)",
      "url": "https://dashboard.zerodeploy.dev/drop-claim?token=zdr_abc123..."
    }
  },
  "message": "Site live at https://shy-hill-9433.zerodeploy.app — expires in 72 hours"
}
```

**Show the user**:
- The live URL (`data.url`)
- Expiration notice (72 hours)
- Claim URL if they want to make it permanent (`next_steps.claim.url`)
- Store the claim token for potential redeploys

## Redeploy (Update Existing Site)

To update an existing drop site, include the `X-Claim-Token` header from the original deploy:

```bash
curl -X POST https://api.zerodeploy.dev/drop \
  -H "Content-Type: text/html" \
  -H "X-Claim-Token: zdr_abc123..." \
  --data-binary @index.html
```

This:
- Updates the site at the same URL
- Resets the 72-hour expiration timer
- Returns the same claim token

## Framework-Specific Examples

### React / Vite / Astro

```bash
npm run build
tar -czf site.tar.gz -C ./dist .
curl -X POST https://api.zerodeploy.dev/drop \
  -H "Content-Type: application/gzip" \
  --data-binary @site.tar.gz
```

### Next.js (Static Export)

```bash
npm run build
tar -czf site.tar.gz -C ./out .
curl -X POST https://api.zerodeploy.dev/drop \
  -H "Content-Type: application/gzip" \
  --data-binary @site.tar.gz
```

### Single HTML + Inline CSS/JS

```bash
# If everything is in one index.html file
curl -X POST https://api.zerodeploy.dev/drop \
  -H "Content-Type: text/html" \
  --data-binary @index.html
```

### PDF or Image File

```bash
curl -X POST https://api.zerodeploy.dev/drop \
  -H "Content-Type: application/pdf" \
  -H "X-Filename: document.pdf" \
  --data-binary @document.pdf
```

## Error Responses

### Rate Limited (429)

```json
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Drop rate limit exceeded",
    "limits": {
      "max_deploys_per_hour": 5,
      "retry_after_seconds": 1842
    }
  }
}
```

**Action**: Wait ~30 minutes and retry, or ask the user to claim a site and use the CLI for more deploys.

### Payload Too Large (413)

```json
{
  "error": {
    "code": "payload_too_large",
    "message": "Total deployment size exceeds 25 MB limit",
    "limits": {
      "max_file_bytes": 10485760,
      "max_files": 1000,
      "max_total_bytes": 26214400
    }
  }
}
```

**Action**: Ask the user to reduce file sizes, remove large assets, or use the CLI with a paid plan (250 MB limit).

### Invalid Archive (400)

```json
{
  "error": {
    "code": "invalid_archive",
    "message": "Could not extract tar.gz archive",
    "hint": "Ensure the body is a valid gzipped tar archive. Create with: tar -czf site.tar.gz -C ./dist ."
  }
}
```

**Action**: Verify tar command syntax. Ensure `-C` flag is used correctly with trailing `.`

## Limits

- **10 MB** max per file
- **1,000 files** max per deployment
- **25 MB** total per deployment
- **5 deploys per hour** per IP address
- **72 hours** expiration (resets on each redeploy)

## Claiming a Site

To make a drop site permanent:

1. Give the user the claim URL from `next_steps.claim.url`
2. User logs in at the claim page (GitHub OAuth, no credit card)
3. User can rename the site and keep it permanently
4. Claimed sites get: custom domains, analytics, forms, rollbacks, no expiration

## Workflow Example

```bash
# 1. Build the site
npm run build

# 2. Create tar.gz archive
tar -czf site.tar.gz -C ./dist .

# 3. Deploy to Drop API
RESPONSE=$(curl -s -X POST https://api.zerodeploy.dev/drop \
  -H "Content-Type: application/gzip" \
  --data-binary @site.tar.gz)

# 4. Parse response (use jq if available, or grep/sed)
URL=$(echo "$RESPONSE" | grep -o '"url":"[^"]*"' | cut -d'"' -f4)
CLAIM_TOKEN=$(echo "$RESPONSE" | grep -o '"claim_token":"[^"]*"' | cut -d'"' -f4)
CLAIM_URL=$(echo "$RESPONSE" | grep -o '"url":"https://dashboard[^"]*"' | cut -d'"' -f4)

# 5. Show user the results
echo "✓ Site deployed: $URL"
echo "⏰ Expires in 72 hours"
echo "🔗 Claim to make permanent: $CLAIM_URL"

# 6. Optional: Store claim token for redeploys
echo "$CLAIM_TOKEN" > .zerodeploy-drop-token
```

## Best Practices

- **Use Drop for quick demos** — One curl command, instant results, no account needed
- **Use CLI for production** — After claiming or for new projects with ongoing updates
- **Store claim tokens** — Save to `.zerodeploy-drop-token` for easy redeploys (add to .gitignore)
- **One build directory** — Ensure tar creates from the correct output directory (dist/, out/, build/)
- **No auth needed** — Drop API never requires API keys or login

## Additional context from user
$ARGUMENTS
