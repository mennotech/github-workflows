# Example Usage for Mennotech GitHub Workflows

This repository provides reusable workflows. Calling repositories should pass configuration, not reimplement signing or deployment logic.

## Correct Separation of Responsibilities

- Application repositories own triggers and deployment values.
- `mennotech/github-workflows` owns orchestration.
- `mennotech/github-actions` owns mechanics.

## Example: Application Repository

The application repository should keep its workflow thin.

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
```

## Migration from Inline PowerShell

If a repository currently contains a workflow like this:

- import certificate inline or via a local script
- sign files inline or via a local script
- deploy files inline or via a local script

it can be replaced with a single reusable workflow job.

### Before

The caller repository owns orchestration and helper scripts.

### After

The caller repository only supplies:

- explicit code-signing secrets
- `runner_group`
- `destination_path`
- optional additional exclusion overrides

`.github` is excluded by default by the reusable workflow. Set `include_github_directory: true` only when `.github` content is part of the deployable payload.
`.git` is enforced as an exclusion by the underlying actions and does not need to be passed from the caller repository.
`exclude_dirs` and `exclude_files` are additive inputs. Their main purpose is to preserve directories and files that the deployed script generates and still needs after an upgrade. Use them for repository-specific content such as `logs`, generated outputs, certificates, or environment-specific config that should not be overwritten.

## PowerShell Script Deployment Example

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
      destination_path: C:\\Scripts\\powershell-script-deploy-directory
      exclude_dirs: logs
      exclude_files: *.crt,Config.json
```
