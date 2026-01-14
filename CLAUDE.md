# CLAUDE.md

This file provides guidance for Claude Code (claude.ai/claude-code) and other AI assistants working with this repository.

## Repository Purpose

This is a **supply chain security fork** of [billchurch/webssh2](https://github.com/billchurch/webssh2), maintained by F5 Government Solutions. The primary purpose is disaster recovery and supply chain security - ensuring we can continue operations if the upstream repository ever becomes unavailable.

**Key principle:** This fork is a passive mirror. We sync everything from upstream and do not intentionally diverge unless absolutely necessary for security patches.

## Upstream Relationship

- **Upstream repository:** `billchurch/webssh2`
- **Sync frequency:** Daily (automated via GitHub Actions)
- **Sync scope:** All branches, all tags, and optionally release artifacts
- **Divergence policy:** Avoid divergence; if patches are required, use the `patches/` directory pattern

## Automated Sync Workflow

The `.github/workflows/sync-upstream.yml` workflow:

1. **Runs daily** at 6:00 AM UTC (and can be triggered manually)
2. **Syncs all tags** from upstream automatically
3. **Creates a PR** when the main branch has new commits from upstream
4. **Optionally syncs release artifacts** when manually triggered with `sync_releases: true`

When a sync PR is created:
- Review the changes for any security concerns
- Verify CI passes
- Merge when ready (no auto-merge to allow review)

## Release Process

This fork uses the same release tooling as upstream (release-please). The release flow:

1. **Sync PR merged** - When upstream changes are merged via sync PR
2. **Release-please creates release PR** - Automatically based on conventional commits
3. **Merge release PR** - Creates tag `webssh2-server-v<version>`
4. **Automated release** - Builds and publishes:
   - npm package
   - Docker images (Docker Hub + GHCR)
   - Release artifacts (`webssh2-<version>.tar.gz` + `.sha256`)

### Downstream Consumer

The `F5GovSolutions/nginx-webssh2` repository consumes releases from this fork via `scripts/fetch-webssh2-release.sh`. It expects:

- Tags named: `webssh2-server-v<version>`
- Release assets: `webssh2-<version>.tar.gz` and `webssh2-<version>.tar.gz.sha256`

## Patching Policy

If security patches or org-specific modifications are ever required:

1. **Create a `patches/` directory** at the repository root
2. **Document each patch** with a README explaining:
   - Why the patch is needed
   - What upstream issue/PR it relates to (if any)
   - When it can be removed (e.g., "remove after upstream v3.2.0")
3. **Prefer upstreaming** - Submit patches upstream when possible
4. **Track patch status** - Mark patches as temporary with removal criteria

Example patch structure:
```
patches/
├── README.md           # Overview of all patches
├── 001-security-fix/
│   ├── README.md       # Patch-specific documentation
│   └── patch.diff      # The actual patch
```

## Development Commands

```bash
# Install dependencies
npm ci

# Build the project
npm run build

# Run tests
npm test

# Run E2E tests
npm run test:e2e

# Start development server
npm run dev

# Lint code
npm run lint
```

## Key Files

- `.github/workflows/sync-upstream.yml` - Upstream sync automation
- `.github/workflows/release-please.yml` - Release automation
- `.github/workflows/docker-publish.yml` - Docker image builds
- `release-please-config.json` - Release configuration
- `CHANGELOG.md` - Auto-generated changelog

## Repository Secrets (Required for Full Functionality)

For the sync and release workflows to function fully:

- `GITHUB_TOKEN` - Automatically provided, used for PRs and releases
- `NPM_TOKEN` - For npm publishing
- `DOCKER_USERNAME` / `DOCKER_TOKEN` - For Docker Hub publishing
- `DOWNSTREAM_TOKEN` / `DOWNSTREAM_REPO` - For notifying downstream consumers
- `UPSTREAM_SYNC_TOKEN` - (Optional) Token with `repo` scope for syncing release artifacts from upstream

## Important Notes for AI Assistants

1. **Do not create divergent changes** unless explicitly requested for security patches
2. **Preserve upstream compatibility** - any changes should not break sync
3. **Check sync status** before making changes - ensure we're current with upstream
4. **Use conventional commits** - required for release-please to work correctly
5. **Test changes locally** before committing - run `npm test` and `npm run build`
