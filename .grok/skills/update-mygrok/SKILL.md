---
name: update-mygrok
description: >
  Check the local mygrok binary against cyanst0ne/grok-build GitHub Releases
  and install immediately if a newer version exists. Use when the user runs
  /update-mygrok, or asks to 升级 mygrok, 更新 mygrok, or check mygrok updates.
  Does not update official grok (`~/.grok/bin/grok`) or this repo's source.
compatibility: Requires `mygrok` and `gh` on PATH
---

# Update mygrok

Upgrade the **installed** `mygrok` binary. Target is `{MYGROK_HOME:-~/.mygrok}/bin/mygrok` via `mygrok update` (GitHub Releases). Official `grok` and this git tree stay untouched.

Do not ask for confirmation. If an update exists, install it in the same turn.

## 1. Resolve binary

```bash
command -v mygrok || echo "$HOME/.mygrok/bin/mygrok"
```

Use that path as `$MYGROK`. If it is missing or not executable, stop and say so. Also require `gh` on PATH (`mygrok update` calls it).

## 2. Check

```bash
"$MYGROK" update --check --json
```

Parse stdout JSON (`camelCase`):

| Field | Meaning |
|-------|---------|
| `currentVersion` | Installed version |
| `latestVersion` | Latest release (may be null) |
| `updateAvailable` | Whether to install |
| `error` | Check failed if non-null |

If `error` is set, print it and stop.

If `updateAvailable` is false: report `currentVersion` is already latest and stop.

## 3. Install

```bash
"$MYGROK" update
```

Download can be ~170MB. Give the command at least 5 minutes. Do not pass `--force-reinstall` unless the user asked to reinstall the same version.

Done when stdout contains `installed successfully` (or equivalent) and the process exits 0. On failure, print the command output; do not fall back to `grok update` or `curl https://x.ai/cli/install.sh`.

## 4. Verify

```bash
"$MYGROK" --version
"$MYGROK" update --check --json
cat "${MYGROK_HOME:-$HOME/.mygrok}/installed-version"
```

Report `before → after`. If a mygrok session is still running, tell the user to restart it.
