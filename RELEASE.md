# Release Guide

## Initial Release Checklist

1. Validate reusable workflow syntax on the branch.
2. Test the workflow from a calling repository on a Windows self-hosted runner.
3. Update `CHANGELOG.md` for the release.
4. Merge the branch with a squash commit.
5. Create and push tag `v1` after the first stable release.

## Compatibility

- `v1` should remain backward compatible for existing callers.
- Breaking workflow input changes should ship in a new major tag.

## Consumer Tagging

Consumers should reference a major tag:

```yaml
uses: mennotech/github-workflows/.github/workflows/sign-and-deploy-windows.yml@v1
```