# Changelog

All notable changes to this repository will be documented in this file.

## [Unreleased]

### Added

- `deploy-private-github-script-windows.yml`: New reusable workflow for upstream-triggered auto-deployment of scripts from a private GitHub repository to Windows self-hosted runners.
  - Supports two deployment rings (`testing` and `broad`) controlled by the caller via `script_ref`.
  - Fetches the private script repository using a read-only SSH deploy key (`SCRIPT_DEPLOY_KEY`). No fork required.
  - Performs the full sign-and-deploy pipeline (sign, verify, deploy, verify, execute) on the fetched script content.
  - Logs remain in the calling client repository only.
- Updated `README.md` and `EXAMPLE_USAGE.md` with documentation and example callers for the new workflow.

## [1.0.0] - 2026-03-19

### Added

- Initial reusable workflow repository scaffold
- Windows sign-and-deploy reusable workflow
- Workflow validation CI for repository YAML files
- `yamllint` configuration and lint enforcement in CI
- Consumer documentation and maintainer release guidance

### Notes

- Initial public release for reusable Mennotech GitHub workflows
- Primary supported entry point is `.github/workflows/sign-and-deploy-windows.yml`
- Intended for Windows self-hosted runners and PowerShell-based deployments
- `.github` is excluded by default by the workflow and can be included with `include_github_directory: true`
- `.git` is enforced as an exclusion by the underlying `mennotech/github-actions` actions