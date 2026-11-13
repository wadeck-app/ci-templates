# Out of Scope — ci-templates

- Local execution — scripts are CI-only; no local developer workflow is provided or expected.
- Platforms other than `win32-x64`, `darwin-arm64`, `darwin-x64`, `linux-x64` (for example `linux-arm64`).
- Tests of the scripts and of the shim — the shim's unit test lives in consumers (`task`), here it is checked by hand.
- Self-versioning — no `package.json` or lockfile; this repo is not a published package.
- Linting or testing infrastructure — no test harness for the scripts themselves.
- Automated propagation to consumer repos — propagation is manual.
- Secret management — tokens and credentials are the responsibility of each consuming repo's CI configuration.
