# mennotech/github-workflows

Reusable GitHub Actions workflows for Mennotech deployment pipelines.

This repository holds reusable workflows that orchestrate the low-level actions published in `mennotech/github-actions`. The workflows currently provide:

- A Windows end-to-end pipeline for code signing, signature verification, deployment, and deployment verification.
- A downstream deployment dispatcher that emits `repository_dispatch` events to one or more repositories and can wait for their workflow completion.

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

Required secrets:

- `CODESIGN_PFX_BASE64`
- `CODESIGN_PFX_PASSWORD`

Secret mapping example:

```yaml
secrets:
  CODESIGN_PFX_BASE64: ${{ secrets.CODESIGN_PFX_BASE64 }}
  CODESIGN_PFX_PASSWORD: ${{ secrets.CODESIGN_PFX_PASSWORD }}
```

### `dispatch-downstream-deployments`

Reusable workflow path:

```yaml
uses: mennotech/github-workflows/.github/workflows/dispatch-downstream-deployments.yml@v1
```

Purpose:

- Build a per-repository matrix from a newline-separated downstream repository list
- Mint scoped GitHub App tokens per downstream owner
- Dispatch `repository_dispatch` events with correlation metadata
- Optionally poll and fail fast when a downstream run times out or concludes unsuccessfully

Required secrets:

- `dispatch_app_id`
- `dispatch_private_key`

Secret mapping example:

```yaml
secrets:
  dispatch_app_id: ${{ secrets.GH_APP_ID }}
  dispatch_private_key: ${{ secrets.GH_APP_PRIVATE_KEY }}
```

## Workflow Inputs

`sign-and-deploy-windows.yml` supports the following caller inputs:

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

`dispatch-downstream-deployments.yml` supports the following caller inputs:

- `downstream_repos`: Newline-separated `org/repo` list. Required.
- `event_type`: Downstream `repository_dispatch` event type string. Required.
- `wait_for_downstream`: Wait for downstream completion. Default: `true`.
- `wait_timeout_seconds`: Max wait per downstream repo. Default: `1200`.
- `poll_interval_seconds`: Poll interval while waiting. Default: `20`.

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
      destination_path: C:\\Scripts\\MyApp
      exclude_dirs: logs
      exclude_files: *.crt,Config.json
      timestamp_server: http://timestamp.digicert.com
```

### Dispatch Downstream Example

```yaml
name: Dispatch Downstream Deployments

on:
  workflow_dispatch:

permissions:
  contents: read

jobs:
  dispatch:
    uses: mennotech/github-workflows/.github/workflows/dispatch-downstream-deployments.yml@v1
    permissions:
      contents: read
    secrets:
      dispatch_app_id: ${{ secrets.GH_APP_ID }}
      dispatch_private_key: ${{ secrets.GH_APP_PRIVATE_KEY }}
    with:
      downstream_repos: ${{ vars.DOWNSTREAM_REPOS }}
      event_type: script-deploy-testing
      wait_for_downstream: true
      wait_timeout_seconds: 1200
      poll_interval_seconds: 20
```

## Design Principles

- Reusable workflows own orchestration and guardrails.
- Composite actions own implementation details.
- Application repositories own deployment-specific values.
- Dispatch workflows emit traceable payloads and surface aggregate outcomes for callers.
- Certificate cleanup is explicitly enabled by this workflow to avoid leaving certificates on self-hosted runners.
- This workflow relies on `mennotech/github-actions@v1` for the low-level signing and deployment mechanics.
- `.github` is excluded by default at the workflow layer, but callers can opt in with `include_github_directory: true` when that content is part of the deployable payload.
- `.git` is enforced as an exclusion by the underlying `mennotech/github-actions` actions, so callers do not need to pass it in workflow inputs.
- `exclude_dirs` and `exclude_files` are additive caller overrides. Their main purpose is to avoid clobbering generated directories and files that a deployed script needs to keep across upgrades. This workflow supplies the `.github` default at the workflow layer rather than relying on broad defaults in the underlying actions.

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