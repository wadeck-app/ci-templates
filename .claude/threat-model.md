# Threat Model — ci-templates

## Supply chain

| Threat | Status | Notes |
|--------|--------|-------|
| `bin-launcher.js.tmpl` runs `npm install -g <platform-pkg>` at first launch if the platform package is missing | Active risk | No integrity check (no lockfile, no hash verification) on the auto-installed package. Mitigated only by the npm registry's own security. |
| npm WRITE token leaked | Requires CI discipline | Token must live in CI secrets only; never committed to repo. Not enforced by this repo — responsibility of each consumer. |

## Template correctness

| Threat | Status | Notes |
|--------|--------|-------|
| `@wadeck-app/` scope hardcoded in launcher template | Bug risk | `{{PKG_PREFIX}}` is substituted but the `@wadeck-app/` org prefix in the auto-install path is not substituted. If `PKG_SCOPE` overrides the scope in `generate-platform-packages.sh`, the generated launcher still references `@wadeck-app/` — install silently targets the wrong package. |
| Exec bit lost by `upload-artifact` / `download-artifact` | Mitigated | Binaries were published 0644 and every command failed silently on macOS. Now: `restore-exec-bits.sh`, `smoke-packed.sh` fails on a tarball entry without the exec bit, and the shim reports exec errors. Each consumer must wire the two scripts in. |

## Propagation
- Files edited in consuming repos (not here) are silently diverged copies — no diff or drift detection exists.
- Consuming repos must manually re-copy after changes; stale copies are undetectable without manual audit.

<!-- TODO: no formal threat model file existed — inferred from README and script review -->
