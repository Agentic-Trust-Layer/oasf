# Instructions for Updating Skills from OASF Repository

## Overview

The agent-explorer indexer can sync skills from your OASF fork. This includes:
- Base skills from `schema/skills/`
- Extension skills from `schema/extensions/<ext>/skills/`
- Skill categories from `schema/skill_categories.json` and extension `main_skills.json`

## Prerequisites

1. **Commit and push your changes** to your OASF fork on GitHub
2. **Get a GitHub Personal Access Token** (optional but recommended):
   - Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
   - Create a token with `public_repo` scope (or `repo` for private repos)
   - Copy the token

## Sync Command

Navigate to the indexer directory and run:

```bash
cd /home/barb/erc8004/agent-explorer/apps/indexer
```

### Basic Sync (using default agntcy/oasf)

```bash
pnpm -s skills:sync
```

### Sync from Your Fork

```bash
GITHUB_TOKEN="your_github_token" \
OASF_REPO="your_github_username/oasf" \
OASF_REF="main" \
pnpm -s skills:sync
```

### Sync from a Specific Branch/Tag

```bash
GITHUB_TOKEN="your_github_token" \
OASF_REPO="your_github_username/oasf" \
OASF_REF="heads/feature/trust-ext" \
pnpm -s skills:sync
```

Or for a tag:
```bash
OASF_REF="tags/v0.2.0" \
pnpm -s skills:sync
```

## Environment Variables

You can also set these in your `.env` file:

```bash
# GitHub repository (default: agntcy/oasf)
OASF_REPO=your_github_username/oasf

# Git reference - branch, tag, or commit (default: main)
OASF_REF=main

# GitHub token for higher rate limits (optional but recommended)
GITHUB_TOKEN=ghp_your_token_here

# Enable/disable OASF sync (default: enabled)
OASF_SYNC=1
```

## What Gets Synced

The sync process will:

1. **Fetch all skill files** from:
   - `schema/skills/**/*.json` (base skills)
   - `schema/extensions/*/skills/**/*.json` (extension skills)

2. **Process skill categories** from:
   - `schema/skill_categories.json` (base categories)
   - `schema/extensions/*/main_skills.json` (extension categories)

3. **Store in database**:
   - Skills are stored in `oasf_skills` table
   - Categories are stored in `oasf_skill_categories` table
   - Each skill includes: name, uid, caption, description, category, extends, schema JSON

## For Your OrgTrust Extension

After adding your extension files:

1. **Commit and push** to your OASF fork:
   ```bash
   cd /home/barb/erc8004/oasf
   git add schema/extensions/orgtrust/
   git commit -m "Add OrgTrust extension with trust validation skills"
   git push origin main
   ```

2. **Run the sync**:
   ```bash
   cd /home/barb/erc8004/agent-explorer/apps/indexer
   GITHUB_TOKEN="your_token" \
   OASF_REPO="your_github_username/oasf" \
   OASF_REF="main" \
   pnpm -s skills:sync
   ```

3. **Verify** the skills were synced by checking the database or GraphQL API

## Troubleshooting

### Rate Limit Errors

If you see rate limit errors, set `GITHUB_TOKEN`:
```bash
GITHUB_TOKEN="your_token" pnpm -s skills:sync
```

### Repository Not Found

- Verify `OASF_REPO` is correct (format: `username/repo`)
- Check the repository is public or your token has access
- Verify `OASF_REF` exists (branch/tag name)

### Skills Not Appearing

- Check the file paths match expected structure:
  - Extension skills: `schema/extensions/<ext>/skills/<category>/<skill>.json`
  - Extension categories: `schema/extensions/<ext>/main_skills.json`
- Verify JSON files are valid
- Check sync logs for errors
- The sync uses SHA checksums - if the tree hasn't changed, it skips (this is normal)

### Force Re-sync

To force a re-sync even if nothing changed, you can clear the checkpoint:
```sql
DELETE FROM checkpoints WHERE key = 'oasf_skills_tree_sha';
```

Then run the sync again.

## Expected Output

Successful sync will show:
```
[oasf-sync] Starting OASF synchronization... { repo: 'your_username/oasf', ref: 'main' }
[oasf-sync] Fetching OASF skills from GitHub...
[oasf-sync] Found X skill files
[oasf-sync] Skills sync complete: X new, Y updated, 0 errors
```

## Notes

- The sync is incremental - it only fetches changed files
- Skills are identified by their path, so moving files will create duplicates
- Extension skills use the path format: `extensions/<ext>/skills/<category>/<skill>`
- Categories from `main_skills.json` are automatically linked to extension skills
