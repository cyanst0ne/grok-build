# Local patches for cyanst0ne/grok-build

These patches are re-applied on top of `xai-org/grok-build` `main` by the
`auto-release` workflow whenever the official CLI stable version advances.

| Patch | Purpose |
|-------|---------|
| `0001-ssrf-allow-198.18-fake-ip.patch` | Do not SSRF-block `198.18.0.0/15` (Clash/mihomo fake-ip) |
| `0002-windows-proto-build-skip-dev-paths.patch` | Skip Unix `/dev/*` protoc dependency tracking on Windows |
| `0003-mygrok-update-from-fork-releases.patch` | `mygrok update` checks/installs from `cyanst0ne/grok-build` GitHub Releases into `~/.mygrok/bin/mygrok.exe` (does not touch official `~/.grok/bin/grok.exe`) |

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

git diff upstream/main -- \
  crates/codegen/xai-grok-update/src/version.rs \
  crates/codegen/xai-grok-update/src/auto_update.rs \
  crates/codegen/xai-grok-update/src/auto_update_tests.rs \
  crates/codegen/xai-grok-update/tests \
  > patches/0003-mygrok-update-from-fork-releases.patch
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

## `mygrok update` (patch 0003)

Default installer is GitHub Releases (`cyanst0ne/grok-build`), not `https://x.ai/cli`.

| What | Where |
|------|--------|
| Version check | `gh release list --repo cyanst0ne/grok-build` |
| Download | `mygrok-windows-x64.exe` / `mygrok-macos-aarch64` |
| Install | `%USERPROFILE%\.mygrok\bin\mygrok.exe` |
| Official grok | untouched (`%USERPROFILE%\.grok\bin\grok.exe`) |
| Auth / sessions | still `~/.grok` (`GROK_HOME`) |

Overrides: `GROK_GH_RELEASE_REPO=owner/repo`, `MYGROK_HOME=...`.
Requires `gh` on PATH. Shared `~/.grok/config.toml` `cli.installer` is ignored so official grok cannot flip mygrok back to the CDN.

`0003` is a diff against current `upstream/main`. Unit tests live in
`auto_update_tests.rs` (extracted from `auto_update.rs` in official 1.0.4);
regenerate that file in the same patch if upstream moves it again.

To ship this change without a new official version: push `main`, then run **build-release** (`workflow_dispatch`, `version=1.0.4`). Do not use **auto-release** for that — it rebases onto current upstream first.

## Binary version stamp

`auto-release` passes the official stable version (e.g. `1.0.0`) into
`build-release` as `version` → env `GROK_VERSION`. That overrides the open-source
crate `Cargo.toml` version (`0.2.x`) so `mygrok --version` matches the GitHub
Release tag / official install channel. Manual `build-release` runs can set the
same `version` input; leave it empty to keep the crate version.
