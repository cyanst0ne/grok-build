# Local patches for cyanst0ne/grok-build

These patches are re-applied on top of `xai-org/grok-build` `main` by the
`auto-release` workflow whenever the official CLI stable version advances.

| Patch | Purpose |
|-------|---------|
| `0001-ssrf-allow-198.18-fake-ip.patch` | Do not SSRF-block `198.18.0.0/15` (Clash/mihomo fake-ip) |
| `0002-windows-proto-build-skip-dev-paths.patch` | Skip Unix `/dev/*` protoc dependency tracking on Windows |

## Regenerate after resolving conflicts

From a working tree that already has the desired source changes relative to
upstream:

```bash
git fetch upstream main
git diff upstream/main -- \
  crates/codegen/xai-grok-tools/src/implementations/grok_build/web_fetch/ssrf.rs \
  > patches/0001-ssrf-allow-198.18-fake-ip.patch

git diff upstream/main -- \
  crates/build/xai-proto-build/src/lib.rs \
  > patches/0002-windows-proto-build-skip-dev-paths.patch
```

Then commit the updated `patches/` and re-run **auto-release**
(`workflow_dispatch`).

## Apply locally

```bash
git fetch upstream main
git checkout -B main upstream/main
git checkout origin/main -- patches .github
git apply --3way patches/*.patch
```
