# Release Guide

## Release Targets

- Immutable release tag: `v1.1.0`
- Moving major tag for consumers: `v1`

Consumers should reference the major tag:

```yaml
uses: mennotech/github-workflows/.github/workflows/sign-and-deploy-windows.yml@v1
```

## Release Checklist

1. Ensure GitHub Actions is enabled for the repository.
2. Validate YAML locally with `yamllint .`.
3. Push the release branch and confirm `Validate Workflows` passes on GitHub.
4. Test the reusable workflow from a calling repository on a Windows self-hosted runner.
5. Verify required caller secrets exist: `CODESIGN_PFX_BASE64` and `CODESIGN_PFX_PASSWORD`.
6. Update `CHANGELOG.md` for the release date and shipped scope.
7. Merge the release branch with a squash commit.
8. Create and push annotated tag `v1.1.0` on the release commit.
9. Create or update tag `v1` to point to the same release commit.
10. Publish a GitHub Release using the `v1.1.0` notes.

## Suggested Release Commands

Run these after the release commit is on `main`:

```powershell
git checkout main
git pull --ff-only origin main
git tag -a v1.1.0 -m "github-workflows v1.1.0"
git tag -f -a v1 -m "github-workflows v1"
git push origin v1.1.0
git push origin v1 --force
```

If you prefer not to force-push tags, delete and recreate `v1` only when intentionally advancing the major tag.

## Compatibility

- `v1` should remain backward compatible for existing callers.
- Breaking workflow input changes should ship in a new major tag.
- New optional inputs and internal implementation changes can ship in minor or patch releases.

## Release Notes Scope

For `v1.1.0`, the release notes should call out:

- New reusable `dispatch-downstream-deployments` workflow
- Matrix-based downstream `repository_dispatch` fan-out
- Optional wait/poll behavior for downstream workflow completion
- Use of GitHub App credentials to mint scoped dispatch tokens

## Consumer Tagging

Consumers should reference a major tag rather than a full patch tag unless they need strict pinning:

```yaml
uses: mennotech/github-workflows/.github/workflows/sign-and-deploy-windows.yml@v1
```