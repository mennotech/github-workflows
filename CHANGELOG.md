# Changelog

All notable changes to this repository will be documented in this file.

## [Unreleased]

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