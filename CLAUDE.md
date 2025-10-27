# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a GitHub Action (composite action) that triggers FluxCD reconciliation for HelmChart sources and HelmReleases in Kubernetes clusters. The action is published to the GitHub Marketplace as "FluxCD Reconcile".

## Architecture

### Composite Action Structure

The core functionality is in `action.yml`, which defines a composite action with bash shell steps:

1. **Setup kubectl** - Uses `azure/setup-kubectl@v4` to install kubectl
2. **Configure kubectl** - Sets up kubeconfig from the `kube-config` input
3. **Install FluxCD CLI** - Installs the flux CLI if not already present
4. **Reconcile HelmChart** - Triggers chart reconciliation using `flux reconcile source chart`
5. **Reconcile HelmRelease** - Triggers release reconciliation using `flux reconcile helmrelease`
6. **Push commits and tags** - Optional step to push git changes (controlled by `git-push` input)

### Dual Mode Operation

The action supports two modes:

**Simple Mode**: When `name` and `namespace` inputs are provided:
- Chart name is auto-generated as `{namespace}-{name}`
- Release name uses the `name` value
- Registry namespace defaults to `flux-registry`

**Advanced Mode**: When individual inputs are provided:
- `chart-name` specifies the HelmChart resource name
- `release-name` specifies the HelmRelease resource name
- `registry-namespace` and `release-namespace` can be customized

The mode selection logic is in the bash scripts within action.yml:71-79 (HelmChart) and action.yml:90-97 (HelmRelease).

## Development Commands

### Testing the Action Locally

Since this is a GitHub Action, it can't be run directly. Test by:
1. Creating a test workflow in `.github/workflows/test.yml`
2. Using `act` (https://github.com/nektos/act) to run workflows locally
3. Testing in a live repository with actual Kubernetes/Flux setup

### Validation

```bash
# Validate action.yml syntax
cat action.yml | yamllint -

# Check for common issues
grep -r "TODO\|FIXME\|XXX" .
```

### Release Process

The repository uses an automated release workflow (`.github/workflows/release.yml`):
- Triggers on push to `main` branch
- Uses custom bot for authentication (`starburst997/custom-bot-init@v1`)
- Auto-versions using `starburst997/auto-version@v1` with major/minor updates
- Generates changelog and creates release via `starburst997/commits-logs@v1`

To create a new release, simply push to main branch - the automation handles versioning and release creation.

### Documentation

Documentation is served via GitHub Pages from the `docs/` directory. The workflow at `.github/workflows/gh-pages.yml` automatically deploys changes when:
- Changes are pushed to `docs/**`
- Workflow is manually triggered

## Important Implementation Details

### Naming Convention

The "simple mode" assumes a naming pattern where HelmChart resources follow `{namespace}-{name}` format. This is a common pattern but may not fit all FluxCD setups. The chart name generation is at action.yml:74.

### Timeout Configuration

Two separate timeouts:
- `chart-timeout` (default: 2m) - for HelmChart reconciliation
- `release-timeout` (default: 5m) - for HelmRelease reconciliation

HelmRelease typically takes longer as it involves actual deployment to the cluster.

### Git Push Behavior

The `git-push` input (action.yml:105-117) has specific error handling:
- Always force-pushes tags
- Attempts to push commits but ignores failures from detached HEAD state (common in PR contexts)
- This is designed for automated workflows that need to update version tags

### Security Considerations

The action accepts `kube-config` which can be either:
- Base64 encoded kubeconfig
- Plain text kubeconfig

The config is written to `~/.kube/config` with 600 permissions (action.yml:52-56).

## Development Rules

### Documentation Requirements

**CRITICAL**: When adding, modifying, or removing action inputs or outputs, you MUST update documentation in THREE locations:

1. **action.yml** - The source of truth for input/output definitions
2. **README.md** - The Inputs table (around line 81-94) must be kept in sync
3. **docs/index.html** - The HTML documentation must reflect all changes

All three files must stay synchronized. Any change to `action.yml` inputs/outputs requires corresponding updates to the other two files. This ensures users have consistent documentation whether they're viewing the repository, the marketplace listing, or the GitHub Pages site.

## Repository Details

- **Author**: starburst997
- **License**: MIT
- **Marketplace**: https://github.com/marketplace/actions/fluxcd-reconcile
- **Repository**: https://github.com/starburst997/flux-reconcile
