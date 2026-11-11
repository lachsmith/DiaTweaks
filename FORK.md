# About this fork

This is a personal fork of [JaceThings/DiaTweaks](https://github.com/JaceThings/DiaTweaks). It follows upstream, with one difference: **DiaTweaks only runs code built here from source.** It never checks for, downloads, stages or installs pre-built DiaTweaks releases. Dia itself still updates through Dia's own feed, and each download is checked against The Browser Company's signature before it's patched.

## Rules

`scripts/check-fork` enforces these. `install.sh` runs it before building and stops if it fails. It checks the working tree, not git history.

1. **No release code.** None of these may appear: `kReleases`, `latestTweaks`, `stageTweaks`, `DiaTweaks.zip`, `staged/DiaTweaks`, `release.plist`, `autoUpdateTweaks`, `api.github.com`, `/compare`, `.github/workflows/release.yml`. `diapatch self-update` may only refuse, and nothing may run it.
2. **Known hosts only.** Every http(s) host in `src/`, `cli/`, `install.sh`, the `Makefile` and `scripts/` is listed. Any host other than these fails:
   - `diabrowser.com` (with or without `www.`)
   - `releases.diabrowser.com`
   - `github.com/JaceThings/DiaTweaks/wiki`
   - `raw.githubusercontent.com/wiki/JaceThings/DiaTweaks`

   Exact URLs allowed as exceptions: in `install.sh`, `sh.rustup.rs` and `rustup.rs` (it offers to install Rust for the `diatweaks` command, only when run interactively and you answer yes). In `scripts/check-tweak-docs`, `github.com/JaceThings/DiaTweaks.wiki.git` (a contributor check that clones the wiki read-only). Also `github.com/JaceThings/DiaTweaks`, which Settings → Home's "GitHub" and "Report a Bug" rows open in the browser, and `www.google.com/search`, Peek's fallback search page.

   Test fixtures and example URLs (`example.com`, `.test`, `localhost`) are ignored, along with two names that are never fetched: Sparkle's XML namespace and Cargo.lock's registry ID.
3. **Quarantine is cleared after signing.** `diapatch build` runs `xattr -cr` on the patched Dia after signing it, because files created while Dia is running diapatch get quarantined.

## Updating

- **Dia:** Settings → Home → Software Update, or `diatweaks update-dia`.
- **DiaTweaks:** `git -C ~/DiaTweaks pull && ~/DiaTweaks/install.sh`.

## Syncing with upstream

Merge `upstream/main` into a branch off `my-version`. Upstream wins, except for release code. Run `scripts/check-fork`, `make` and `make test`, then open a PR into `my-version`.

## History

- f09c7bf, ec01611, 0f3f9bc: removed pre-built DiaTweaks updates and cleared quarantine after signing.
- 2026-11-11: synced with upstream (see the `sync-upstream` PR).
