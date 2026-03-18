# mennotech/github-workflows

Reusable GitHub Actions workflows for Mennotech deployment pipelines.

This repository holds reusable workflows that orchestrate the low-level actions published in `mennotech/github-actions`. The first workflow in this repository provides an end-to-end Windows pipeline for code signing, signature verification, deployment, and deployment verification.

## Repository Architecture

This repository sits between application repositories and `mennotech/github-actions`:

- Application repositories define what is deployed and where.
- `mennotech/github-workflows` defines how deployment happens.
- `mennotech/github-actions` implements the low-level mechanics.

## Available Workflows

### `sign-and-deploy-windows`

Reusable workflow path:

```yaml
uses: mennotech/github-workflows/.github/workflows/sign-and-deploy-windows.yml@v1
```

Purpose:

- Check out the caller repository
- Import the code-signing certificate
- Sign PowerShell files
- Verify signatures after signing
- Deploy files to a Windows target directory
- Run a dry-run deployment check after deployment

## Required Secrets

The reusable workflow expects the caller repository to provide these secrets:

- `CODESIGN_PFX_BASE64`
- `CODESIGN_PFX_PASSWORD`

Pass these secrets explicitly from the caller workflow:

```yaml
secrets:
  CODESIGN_PFX_BASE64: ${{ secrets.CODESIGN_PFX_BASE64 }}
  CODESIGN_PFX_PASSWORD: ${{ secrets.CODESIGN_PFX_PASSWORD }}
```

## Workflow Inputs

`sign-and-deploy-windows.yml` supports the following caller inputs:

- `destination_path`: Windows destination path to deploy to.
- `runner_group`: Self-hosted runner group name.
- `source_path`: Source path inside the checked-out repository. Default `.`.
- `exclude_dirs`: Comma-separated directory exclusions for signing and deployment.
- `signing_exclude_dirs`: Comma-separated directory exclusions used only during signing and signature verification.
- `exclude_files`: Comma-separated file exclusions for deployment.
- `file_match`: Comma-separated PowerShell file glob list for signing.
- `timestamp_server`: Authenticode timestamp server URL.
- `robocopy_options`: Comma-separated robocopy options.
- `recurse`: Whether signing should recurse into subdirectories.

## Example Consumer Workflow

Application repositories should keep only application-specific configuration:

```yaml
name: Sign and Deploy

on:
  workflow_dispatch:
  push:
    branches: ["main"]

permissions:
  contents: read

jobs:
  deploy:
    uses: mennotech/github-workflows/.github/workflows/sign-and-deploy-windows.yml@v1
    permissions:
      contents: read
    secrets:
      CODESIGN_PFX_BASE64: ${{ secrets.CODESIGN_PFX_BASE64 }}
      CODESIGN_PFX_PASSWORD: ${{ secrets.CODESIGN_PFX_PASSWORD }}
    with:
      runner_group: Domain Controllers
      destination_path: C:\\Scripts\\my-github-hosted-script
      exclude_dirs: .git,.github,_work,logs
      exclude_files: *.crt,Config.json
      timestamp_server: http://timestamp.digicert.com
```

## Design Principles

- Reusable workflows own orchestration and guardrails.
- Composite actions own implementation details.
- Application repositories own deployment-specific values.
- Certificate cleanup stays enabled by default to avoid leaving certificates on self-hosted runners.
- Signing exclusions intentionally omit `_work` because GitHub runner workspace paths contain `_work`, and excluding it would skip the real repository files.

## Notes

The upstream `mennotech/github-actions` documentation currently includes an example using action-style syntax for `github-workflows`. This repository uses the correct reusable workflow form:

```yaml
uses: mennotech/github-workflows/.github/workflows/sign-and-deploy-windows.yml@v1
```

## Development

- Maintainer instructions: [`.github/INSTRUCTIONS.md`](.github/INSTRUCTIONS.md)
- Example consumer usage: [`EXAMPLE_USAGE.md`](./EXAMPLE_USAGE.md)
- Release checklist: [`RELEASE.md`](./RELEASE.md)
- Changelog: [`CHANGELOG.md`](./CHANGELOG.md)

## License

MIT