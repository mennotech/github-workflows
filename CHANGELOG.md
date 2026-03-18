# Changelog

All notable changes to this repository will be documented in this file.

## [Unreleased]

## [1.0.0] - 2026-03-18

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
- Signing exclusions now use a separate default list so GitHub runner `_work` paths do not suppress signing