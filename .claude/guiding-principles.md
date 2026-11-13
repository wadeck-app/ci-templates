# Guiding Principles — ci-templates

## Single source of truth
- This repo is the canonical location for all shared CI scripts and templates.
- Never edit copies in individual repos — update here and propagate manually.
- Individual repos hold files under `ci/`; those are copies, not sources.

## Execution context
- Scripts target GitHub Actions exclusively; they depend on `$GITHUB_OUTPUT`, `$GITHUB_EVENT_NAME`, `$GITHUB_SHA`, and related env vars.
- `generate-platform-packages.sh` is CI-only — it has no local execution path.

## Platform support
- Selectable among `win32-x64`, `darwin-arm64`, `darwin-x64`, `linux-x64` via `PLATFORMS`. Linux is x64 only (no `linux-arm64` yet).
- Consumers set `PLATFORMS` explicitly; the shim refuses every host outside the list. The owner runs Apple silicon only: new CLIs list `win32-x64,darwin-arm64,linux-x64` (Linux for the Linux machine).

## Versioning
- Edge builds (push): `YYYY.MM.DD-NNN-<sha8>` format.
- Stable builds (workflow_dispatch): semver.
- BUILD number zero-padded to 3 digits (`001` not `1`) — required for lexicographic sort correctness.

## Configuration
- npm scope defaults to `wadeck-app`, overridable via `PKG_SCOPE` env var.
- Registry defaults to `https://npm.pkg.github.com/`, overridable via `REGISTRY` env var.
- Template variables in `bin-launcher.js.tmpl` use `{{CLI_NAME}}`, `{{PKG_PREFIX}}` and `{{PLATFORMS}}` (comma list) substituted by `sed` at generation time.
- A shim failure is never silent: every error prints what failed, the path and the fix.
- Artifact transfers drop file modes: run `restore-exec-bits.sh` after `download-artifact`, and `smoke-packed.sh` on macOS before publishing.

## Distribution model
- Three-layer structure: platform packages (binary, OS+CPU restricted) → dist package (JS launcher + `optionalDependencies`) → extra bundled CJS files.
- `generate_dist` inlines extra file names into `files[]` in the dist `package.json`.
- On every platform, commands run `node <bundle>` directly except `start`, which requires the Go launcher to daemonize (Windows launcher is a SUBSYSTEM:WINDOWS GUI binary that swallows stdio).
