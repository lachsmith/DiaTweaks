# Upstream sync log

Syncing the fork (origin, github.com/lachsmith/DiaTweaks) with upstream (github.com/JaceThings/DiaTweaks) while keeping the fork's security changes (f09c7bf, ec01611, 0f3f9bc: no pre-built DiaTweaks updates, only Dia's own signature-checked updates, quarantine cleared after signing).

## Branch naming

The fork has no `main` branch. Its default and only branch is `my-version`. On the user's instruction (2026-11-11), `my-version` stands in wherever the plan says `main`:
it was pulled, tagged `pre-sync-2026-11-11`, and `sync-upstream` was branched from it. The PR goes into `my-version`, and abandoning the sync means switching back to `my-version`.
No branch is created or renamed to `main`, and nothing is committed to `my-version` directly.

## Checklist

- [x] 1. Prepare repo, tag, branch, create this log
- [x] 2. Merge upstream/main and resolve conflicts
- [ ] 3. Compare quarantine handling
- [ ] 4. Write scripts/check-fork and FORK.md
- [ ] 5. Confirm defaultOn is NO for Passkeys, ICloudPasswords, OnePassword
- [ ] 6. make, make test
- [ ] 7. Summary, then install on approval, diapatch status
- [ ] 8. Push and open PR into my-version on approval

## Current state / next action

Step 2 done (merge commit). Not built yet: that's step 6. Next: step 3, compare upstream's quarantine handling with the fork's `xattr -cr` in `build()` of src/diapatch/main.m (both are in place for now).

## Step log

### Step 1: done (2026-11-11)

Ran:
- `git switch my-version`, confirmed the working tree was clean, then `git pull --ff-only origin my-version` (already up to date, 0f3f9bc)
- `git remote add upstream https://github.com/JaceThings/DiaTweaks.git`, then `git fetch --no-tags upstream`
- `git remote set-url --push upstream DISABLED`, so a push to upstream fails by accident (PRs go to the fork only)
- `git tag pre-sync-2026-11-11 my-version` (local tag, not pushed)
- `git switch -c sync-upstream my-version`

Found: upstream/main is 167 commits ahead of `my-version`, and `my-version` is 3 ahead. Merge base is ade5a1a.

For the user to look at: nothing.

Rollback: `git switch my-version && ./install.sh --yes`, and leave `sync-upstream` for the user to delete.

### Step 2: done (2026-11-11 · see merge commit time)

Ran `git merge --no-ff upstream/main` (upstream/main at the time of fetch, 167 commits). Rule: upstream wins everywhere except code that checks for, downloads, stages or installs pre-built DiaTweaks releases, which stays removed. Dia's own update path (`update`, `apply --dia`, `check`, the Sparkle hooks) is kept.

Conflicted files:

| File | Resolution |
|---|---|
| `README.md` | Took upstream's tweak docs (merged cleanly). Kept the fork note and the build-from-source update steps. The Updating paragraph keeps the fork's Dia-only wording, with upstream's new page name: Settings → **Home** → Software Update (upstream renamed it from Patches). |
| `scripts/package` | Kept deleted (builds the release zip). |
| `scripts/release-notes` | Kept deleted (drafts release notes, uses `/compare`). |
| `src/diapatch/main.m` | Took upstream's imports minus CommonCrypto (only the release digest used it). Dropped `kReleases`, `kReleaseAsset` and the new `kCompare`. Dropped the release block: `installedRelease`, `stagedRelease`, `tweaksTargets`, `sha256`, `latestTweaks`, `sourceCommit`, `sourceNotOlder`, `tweaksDiffer`, `tweaksAvailable`, `unquarantine`, `stageTweaks`, `applyTweaks`. Kept upstream's `agentPlist`, `agentService` and `removeRelay` from the same hunk, since install and uninstall use them. `build()`: upstream's icon and LaunchServices steps, plus the fork's `xattr -cr` after signing (provisional, see step 3). Purge list: upstream's new entries minus `release.plist`. `status`: upstream's `printJSON`, no `release` key. Usage: upstream's `set-icon`, no `--tweaks`, no `DIAPATCH_RELEASES`/`DIAPATCH_COMMIT`/`DIAPATCH_COMPARE`. `main()`: upstream's native messaging host dispatch, no `gStagedTweaks`, `apply` has no `--tweaks`. The `self-update` stub still refuses and points at building from source. |
| `src/dylib/Settings/Overview+Updates.m` | Accepted upstream's deletion (a5f74a2 split Settings into Home and Patches). Its replacement, `Settings/Home.m`, is covered below. |
| `src/dylib/Settings/SoftwareUpdate.m` | Took upstream's failure reason in the shape string. Only Dia is listed for the auto-update levels. The diff against upstream is exactly the fork's DiaTweaks removals. |
| `src/dylib/Updates/Updates.h` | Took upstream (which deleted the header comment). Still declares only `DPDia`. |
| `src/dylib/Updates/Updates.m` | `download:` uses upstream's `DPRunCLIReporting`, with `update` only (no `self-update`). `DPInstallOnQuit` installs a staged Dia only, via upstream's `DPLaunchCLI`. Kept upstream's Sparkle hooks (save Dia's feed, route Check for Updates to DiaPatches, turn off Sparkle's own schedule) and `DPLiftQuarantine`. |

Release code that upstream brought back in files that merged cleanly, now removed:
- `src/dylib/Settings/Home.m` (new, replaces Overview+Updates.m): the update summary counts only Dia, and the version label says "Built from source" instead of a release tag.
- `cli/src/update.rs`: deleted. `diatweaks update` ran `self-update` and then `apply --tweaks`. Removed the `Update` subcommand and the "Update DiaTweaks" menu item from `cli/src/main.rs` and `cli/src/menu.rs`. `diatweaks update-dia` is unchanged.
- `tests/tweaks-update`: deleted (tested release staging and the GitHub compare check). Removed from `make check` in the Makefile.
- `install.sh`: removed the new `rm -f "$support/release.plist"` line, as ec01611 did for the earlier cleanup.

Checked and kept: `.github/workflows/build.yml` (build on push/PR) and `pr.yml` (format check). Neither publishes releases. There is no `release.yml`.

For the user to look at:
- `src/dylib/Settings/Home.m` links to `https://github.com/JaceThings/DiaTweaks`, `/issues/new/choose` and `/wiki`. Only `/wiki` is on the step 4 allowed-host list. They're links opened in the browser, not fetches. To be decided in step 4.
- README's install line still clones upstream (`github.com/JaceThings/DiaTweaks.git`). It's unchanged from before the sync.
- The Settings page name changed upstream: Software Update is now under **Settings → Home**, not Patches.
