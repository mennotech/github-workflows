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

### `deploy-private-github-script-windows`

Reusable workflow path:

```yaml
uses: mennotech/github-workflows/.github/workflows/deploy-private-github-script-windows.yml@v1
```

Purpose:

- Support **upstream-triggered auto-deployment** of scripts sourced from a private GitHub repository (e.g. `mennotech/script`) to Windows self-hosted runners.
- Caller repositories listen for `repository_dispatch` or other triggers and invoke this workflow via `workflow_call`.
- Scripts are fetched using a read-only SSH deploy key — no fork of the script repository is required.
- Supports two deployment rings controlled by the `script_ref` input:
  - **Testing ring**: pass `script_ref: testing`
  - **Broad ring**: pass `script_ref: main` (or a pinned commit SHA)

Steps performed:

1. Check out the calling client repository
2. Check out the private script repository (using `SCRIPT_DEPLOY_KEY`) into a `script/` subdirectory
3. Import the code-signing certificate
4. Sign PowerShell files in `script/`
5. Verify signatures
6. Deploy signed files to `destination_path`
7. Verify deployment state

## Required Secrets

### `sign-and-deploy-windows.yml`

- `CODESIGN_PFX_BASE64`
- `CODESIGN_PFX_PASSWORD`

### `deploy-private-github-script-windows.yml`

- `SCRIPT_DEPLOY_KEY` — Read-only SSH deploy key for the private script repository
- `CODESIGN_PFX_BASE64`
- `CODESIGN_PFX_PASSWORD`

Pass these secrets explicitly from the caller workflow:

```yaml
secrets:
  SCRIPT_DEPLOY_KEY: ${{ secrets.SCRIPT_DEPLOY_KEY }}
  CODESIGN_PFX_BASE64: ${{ secrets.CODESIGN_PFX_BASE64 }}
  CODESIGN_PFX_PASSWORD: ${{ secrets.CODESIGN_PFX_PASSWORD }}
```

## Workflow Inputs

### `sign-and-deploy-windows.yml`

- `destination_path`: Windows destination path to deploy to. Required.
- `runner_group`: Self-hosted runner group name. Required.
- `source_path`: Source path inside the checked-out repository. Default: `.`.
- `exclude_dirs`: Comma-separated additional directory exclusions for application content such as `logs` or other directories the deployed script generates and must preserve during upgrades. Default: empty.
- `include_github_directory`: Set to `true` only when `.github` content should be signed and deployed. Default: `false`.
- `exclude_files`: Comma-separated additional file exclusions for generated or environment-specific files that must not be overwritten during upgrades. Default: empty.
- `file_match`: Comma-separated PowerShell file glob list for signing. Default: `*.ps1,*.psm1,*.psd1`.
- `timestamp_server`: Authenticode timestamp server URL. Default: `http://timestamp.digicert.com`.
- `robocopy_options`: Comma-separated robocopy options. Default: `/R:2,/W:2,/NDL,/NFL,/NP`.
- `recurse`: Whether signing should recurse into subdirectories. Default: `true`.

### `deploy-private-github-script-windows.yml`

- `script_repo`: Private script repository (e.g. `mennotech/script`). Required.
- `script_ref`: Git ref to check out from the script repo — branch (`main`, `testing`) or commit SHA. Required. Controls the deployment ring.
- `runner_group`: Self-hosted runner group name. Required.
- `destination_path`: Windows destination path to deploy to. Required.
- `exclude_dirs`: Comma-separated additional directory exclusions. Default: empty.
- `include_github_directory`: Include `.github` in signing/deployment. Default: `false`.
- `exclude_files`: Comma-separated additional file exclusions. Default: empty.
- `file_match`: Comma-separated PowerShell file glob list for signing. Default: `*.ps1,*.psm1,*.psd1`.
- `timestamp_server`: Authenticode timestamp server URL. Default: `http://timestamp.digicert.com`.
- `robocopy_options`: Comma-separated robocopy options. Default: `/R:2,/W:2,/NDL,/NFL,/NP`.
- `recurse`: Whether signing should recurse into subdirectories. Default: `true`.

## Example Consumer Workflow

### Standard sign-and-deploy

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
      destination_path: C:\\Scripts\\MyApp
      exclude_dirs: logs
      exclude_files: *.crt,Config.json
      timestamp_server: http://timestamp.digicert.com
```

### Upstream-triggered auto-deploy (two rings)

Client repositories handle `repository_dispatch` and invoke the private-script workflow:

```yaml
name: Auto-Deploy from Upstream Script Repo

on:
  repository_dispatch:
    types: [deploy-testing, deploy-broad]

permissions:
  contents: read

jobs:
  deploy:
    uses: mennotech/github-workflows/.github/workflows/deploy-private-github-script-windows.yml@v1
    permissions:
      contents: read
    secrets:
      SCRIPT_DEPLOY_KEY: ${{ secrets.SCRIPT_DEPLOY_KEY }}
      CODESIGN_PFX_BASE64: ${{ secrets.CODESIGN_PFX_BASE64 }}
      CODESIGN_PFX_PASSWORD: ${{ secrets.CODESIGN_PFX_PASSWORD }}
    with:
      script_repo: mennotech/script
      # testing ring uses the testing branch; broad ring uses main
      script_ref: ${{ github.event.action == 'deploy-testing' && 'testing' || 'main' }}
      runner_group: Domain Controllers
      destination_path: C:\\Scripts\\MyApp
      exclude_dirs: logs
      exclude_files: Config.json
```

## Design Principles

- Reusable workflows own orchestration and guardrails.
- Composite actions own implementation details.
- Application repositories own deployment-specific values.
- Certificate cleanup is explicitly enabled by this workflow to avoid leaving certificates on self-hosted runners.
- All workflows rely on `mennotech/github-actions@v1` for the low-level signing and deployment mechanics.
- `.github` is excluded by default at the workflow layer, but callers can opt in with `include_github_directory: true` when that content is part of the deployable payload.
- `.git` is enforced as an exclusion by the underlying `mennotech/github-actions` actions, so callers do not need to pass it in workflow inputs.
- `exclude_dirs` and `exclude_files` are additive caller overrides. Their main purpose is to avoid clobbering generated directories and files that a deployed script needs to keep across upgrades. This workflow supplies the `.github` default at the workflow layer rather than relying on broad defaults in the underlying actions.
- `deploy-private-github-script-windows.yml` accesses the private script repository with a read-only SSH deploy key and never requires a fork. Logs remain in the client repository. Ring selection (`testing` vs `broad`) is controlled by the caller via `script_ref`.

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