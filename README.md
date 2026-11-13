# ci-templates

Shared CI scripts and templates for all wadeck-app repos. Copy or reference these files; do not edit per-repo copies — update here and propagate.

## Scripts

| Script | Purpose |
|--------|---------|
| `scripts/compute-version.sh` | Canonical version string computation. Outputs `version`, `dist_tag`, `is_prerelease`, `tag` to `$GITHUB_OUTPUT`. |
| `scripts/generate-platform-packages.sh` | Generates one npm package directory per platform in `PLATFORMS` (`<cli>-cli-<platform>`) and a dist package with the JS shim. Run in CI before copying binaries. |
| `scripts/restore-exec-bits.sh` | Restores the exec bit that `download-artifact` drops. Run before `npm pack` / `npm publish`. |
| `scripts/smoke-packed.sh` | Packs and installs the CLI into a throwaway prefix on a macOS or Linux runner; fails if a tarball entry lacks the exec bit or `--version` / `--help` fail. |

## Templates

| File | Purpose |
|------|---------|
| `templates/bin-launcher.js.tmpl` | JS shim of published CLI packages. Placeholders: `{{CLI_NAME}}`, `{{PKG_PREFIX}}`, `{{PLATFORMS}}`. Refuses any host outside `PLATFORMS`; runs `node <bundle>` directly for every command except `start`; only `start` uses the Go launcher (auto-installed, exec bit restored if missing). Every exec failure is printed with its code, path and fix. Logic is `run(deps)` (injectable side effects): consumers unit-test it, see `task/packages/task-cli/src/binShim.unit.test.ts`. |

## Usage

### compute-version.sh

```bash
# In a GitHub Actions workflow step:
bash ci/scripts/compute-version.sh
# Reads: GITHUB_EVENT_NAME, VERSION_INPUT, BREAKING_INPUT, GITHUB_SHA
# Writes to $GITHUB_OUTPUT: version, dist_tag, is_prerelease, tag
```

Version format: `YYYY.MM.DD-NNN-<sha8>` (edge) or `<semver>` (stable, via `workflow_dispatch`).

### generate-platform-packages.sh

```bash
# Single CLI:
bash ci/scripts/generate-platform-packages.sh flow

# CLI with extra bundled files:
bash ci/scripts/generate-platform-packages.sh flow:worker.cjs task

# Environment overrides:
TEMPLATES_DIR=path/to/templates REGISTRY=https://npm.pkg.github.com/ PKG_SCOPE=wadeck-app \
PLATFORMS=win32-x64,darwin-arm64 \
  bash ci/scripts/generate-platform-packages.sh flow
```

Generates under `packages/`: `<cli>-cli-<platform>/` for each platform in `PLATFORMS`, and `<cli>-cli-dist/`. An unknown platform key fails the script.

`PLATFORMS` defaults to all three platforms so existing consumers keep building; set it explicitly in every consumer (the same list for `restore-exec-bits.sh` and `smoke-packed.sh`).

### restore-exec-bits.sh and smoke-packed.sh

```yaml
- uses: actions/download-artifact@v4
  with: { name: <cli>-cli-packages, path: packages }
- run: bash ci/scripts/restore-exec-bits.sh packages <cli> win32-x64,darwin-arm64
- run: bash ci/scripts/smoke-packed.sh packages <cli> "$VERSION" win32-x64,darwin-arm64   # macOS or Linux runner only
```

`macos-14` (Apple silicon) runs the smoke job. Make it `continue-on-error` until the runner is proven reliable.

## Supported platforms

`PLATFORMS` selects among:

| Key | OS | Arch |
|-----|----|------|
| `win32-x64` | Windows | x64 |
| `darwin-arm64` | macOS | arm64 |
| `darwin-x64` | macOS | x64 |
| `linux-x64` | Linux | x64 |
