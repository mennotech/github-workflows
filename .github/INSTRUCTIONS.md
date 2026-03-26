# Development Instructions

## Repository Purpose

This repository contains reusable GitHub workflows that compose `mennotech/github-actions` into end-to-end deployment pipelines.

## Conventions

- Keep orchestration in reusable workflows.
- Do not duplicate low-level signing or deployment logic from `github-actions`.
- Favor workflow inputs for caller-specific values.
- Preserve secure defaults, especially certificate cleanup on self-hosted runners.
- Use pwsh instead of bash as the shell to run code (except validate-workflows.yml)

## Current Scope

- Windows self-hosted runner workflows
- PowerShell signing and file deployment pipelines

## Validation Expectations

- Validate workflow syntax before release.
- Test from a real calling repository before tagging a new major version.
- Treat workflow input changes as public API changes.