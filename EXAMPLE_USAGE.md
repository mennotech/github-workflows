# Example Usage for Mennotech GitHub Workflows

This repository provides reusable workflows. Calling repositories should pass configuration, not reimplement signing or deployment logic.

## Correct Separation of Responsibilities

- Application repositories own triggers and deployment values.
- `mennotech/github-workflows` owns orchestration.
- `mennotech/github-actions` owns mechanics.

## Example: Application Repository

The application repository should keep its workflow thin:

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
      runner_group: SCS Domain Controllers
      destination_path: C:\\Scripts\\exchange-apply-address-book-policy
      exclude_dirs: .git,.github,_work,logs
      exclude_files: *.crt,Config.json
      timestamp_server: http://timestamp.digicert.com
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

- `runner_group`
- `destination_path`
- optional exclusion overrides
- inherited code-signing secrets

By default, signing exclusions are narrower than deployment exclusions. This is intentional because GitHub runner workspace paths include `_work`, and excluding `_work` during signing would filter out the actual checked-out repository files.

## Exchange Apply Address Book Policy Example

Based on the current deployment in `exchange-apply-address-book-policy`, the equivalent caller workflow becomes:

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
      runner_group: SCS Domain Controllers
      destination_path: C:\\Scripts\\exchange-apply-address-book-policy
      exclude_dirs: .git,.github,_work,logs
      exclude_files: *.crt,Config.json
      timestamp_server: http://timestamp.digicert.com
```