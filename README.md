# Axero App Workflows

Reusable GitHub Actions workflows for building and releasing Axero Apps.

## Quick Setup

### 1. Add the workflow to your app repo

Create `.github/workflows/release.yml`:

```yaml
name: Release

on:
  workflow_dispatch:
    inputs:
      version_bump:
        description: 'Version bump (ignored if manifest changed)'
        required: true
        default: 'patch'
        type: choice
        options:
          - patch
          - minor
          - major

  push:
    branches: [main, master]
    paths-ignore:
      - '**.md'

jobs:
  release:
    uses: joanlm/axeroapp-workflows/.github/workflows/build-axero-app.yml@main
    with:
      version_bump: ${{ inputs.version_bump || 'patch' }}
    secrets:
      cli_token: ${{ secrets.AXEROAPP_CLI_TOKEN }}
```

### 2. Add the required secret

Add `AXEROAPP_CLI_TOKEN` to your repo's secrets:

1. Go to your repo → Settings → Secrets → Actions
2. Add new secret: `AXEROAPP_CLI_TOKEN`
3. Value: A GitHub PAT with `repo` scope and access to `joanlm/axeroapp-cli`

## How It Works

### Triggers

| Trigger | When | Version |
|---------|------|---------|
| **Manual** | Actions → Release → Run workflow | Choose bump type |
| **Auto** | Push/merge to main | Patch bump |

### Version Logic

1. Reads `version` from `manifest.json`
2. Compares to latest release tag
3. **If different** → Uses manifest version (you bumped it manually)
4. **If same** → Auto-bumps (patch by default)

### Workflow Steps

1. **Checkout** your app repo
2. **Determine version** from manifest vs latest release
3. **Download CLI** from private axeroapp-cli releases
4. **Build** pages (TypeScript/CSS → HTML)
5. **Package** into ZIP
6. **Create release** with ZIP attached

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `version_bump` | `patch` | Bump type if manifest unchanged |
| `app_path` | `.` | Path to app if not at repo root |
| `cli_version` | `latest` | Specific CLI version to use |
| `skip_build` | `false` | Skip build step for simple apps |

## Outputs

| Output | Description |
|--------|-------------|
| `version` | The released version |
| `release_url` | URL to the GitHub release |

## Example: Access outputs

```yaml
jobs:
  release:
    uses: joanlm/axeroapp-workflows/.github/workflows/build-axero-app.yml@main
    secrets:
      cli_token: ${{ secrets.AXEROAPP_CLI_TOKEN }}

  notify:
    needs: release
    runs-on: ubuntu-latest
    steps:
      - run: echo "Released v${{ needs.release.outputs.version }}"
```

## Troubleshooting

### "workflow was not found"
- Ensure you're referencing `@main` (not `@master`)
- Check that `AXEROAPP_CLI_TOKEN` has access to this repo

### "Resource not accessible"
- Your `AXEROAPP_CLI_TOKEN` doesn't have access to `joanlm/axeroapp-cli`
- Create a PAT with `repo` scope

### Build fails
- Test locally: `axeroapp build .`
- Use `skip_build: true` for apps without TypeScript/CSS

## License

MIT
