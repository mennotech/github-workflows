# Changelog

All notable changes to this repository will be documented in this file.

## [Unreleased]

## [1.1.0] - 2026-03-25

### Added

- New reusable workflow `.github/workflows/dispatch-downstream-deployments.yml`
- Support for dispatching `repository_dispatch` events to multiple downstream repositories via matrix fan-out
- Optional downstream run polling with configurable timeout and poll interval inputs

### Notes

- Workflow requires GitHub App credentials (`dispatch_app_id` and `dispatch_private_key`) to mint per-org dispatch tokens
- Workflow output `dispatch_result` summarizes the overall matrix outcome for callers

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