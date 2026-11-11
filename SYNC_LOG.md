# Upstream sync log

Syncing the fork (origin, github.com/lachsmith/DiaTweaks) with upstream (github.com/JaceThings/DiaTweaks) while keeping the fork's security changes (f09c7bf, ec01611, 0f3f9bc: no pre-built DiaTweaks updates, only Dia's own signature-checked updates, quarantine cleared after signing).

## Branch naming

The fork has no `main` branch. Its default and only branch is `my-version`. On the user's instruction (2026-11-11), `my-version` stands in wherever the plan says `main`:
it was pulled, tagged `pre-sync-2026-11-11`, and `sync-upstream` was branched from it. The PR goes into `my-version`, and abandoning the sync means switching back to `my-version`.
No branch is created or renamed to `main`, and nothing is committed to `my-version` directly.

## Checklist

- [x] 1. Prepare repo, tag, branch, create this log
- [x] 2. Merge upstream/main and resolve conflicts
- [x] 3. Compare quarantine handling
- [x] 4. Write scripts/check-fork and FORK.md (2 host findings await the user)
- [x] 5. Confirm defaultOn is NO for Passkeys, ICloudPasswords, OnePassword
- [ ] 6. make, make test
- [ ] 7. Summary, then install on approval, diapatch status
- [ ] 8. Push and open PR into my-version on approval

## Current state / next action

Step 5 done. Next: step 6, `make` then `make test` on `sync-upstream`. Still open from step 4: the user's decision on the two check-fork host findings (install.sh stops until then).

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

### Step 3: done (2026-11-11)

Ran: read-only comparison (`git show 96f7b45`, `git log -S xattr upstream/main`). No code changed in this step.

Upstream's quarantine handling after the merge:
- `clone()` runs `xattr -cr` on the fresh copy, before patching (d7f9ab4). This cleans the input because `codesign --strict` rejects extended attributes. It doesn't cover files created later in `build()`: the rewritten Info.plist, the icon files from `DPPrepareIcons`, and `_CodeSignature` from signing.
- `update()` runs `xattr -cr` on the unzipped Dia download before verifying it. This covers the pristine copy, not the patched build.
- #67 (96f7b45) cleared quarantine on DiaTweaks release downloads (`unquarantine`, removed in step 2) and added `DPLiftQuarantine` in Updates.m, which clears quarantine on the installed `bin/diapatch`, `bin/diatweaks`, dylib and patches.json. It's kept as upstream code: it only clears a flag and downloads nothing.
- `PasswordsLaunch.m` clears quarantine on its own copy of Dia (passwords tweaks only).

Decision: keep the fork's `xattr -cr work` after `sign(work)` and before `moveBundle` in `build()`, and keep upstream's handling too. The fork's call is the only one that runs on the finished patched bundle. When Dia starts diapatch (Software Update, install on quit), every file `build()` creates after the clone is quarantined. Upstream's calls clean the inputs and are still needed for `codesign --strict`. The two complement each other, so neither replaces the other.

For the user to look at: as f09c7bf said, the post-signing clear still hasn't been proven against a real Dia update started from inside Dia.

### Step 4: done, with findings for the user (2026-11-11)

Added:
- `scripts/check-fork`: scans the working tree (tracked and untracked, via `git grep --untracked`). It fails on the release patterns in the plan, on `.github/workflows/release.yml` existing, on `diapatch self-update` doing anything but `fail(`, and on anything in src/ or cli/ calling `self-update`. It lists every http(s) host in src/ and cli/ with a verdict, and fails on hosts outside the allowlist. It skips itself, FORK.md and SYNC_LOG.md, which name the patterns to explain them.
- Ignored by design, all listed in the output: test fixtures (`tests/`, `fixtures/`), example hosts (`example.com/org/net`, `.example`, `.test`, `.invalid`, `localhost`, `127.*`), Sparkle's XML namespace name (`http://www.andymatuschak.org/xml-namespaces/sparkle`, an identifier in the feed parser, never fetched), and Cargo.lock's `github.com/rust-lang/crates.io-index` (Cargo's registry ID). `www.diabrowser.com` counts as `diabrowser.com`.
- `install.sh` runs `scripts/check-fork` before `make -s`, and `set -e` stops the install if it fails.
- `FORK.md`: the rules, how to update, how to sync, and a History section.

Ran `scripts/check-fork`. Negative test: planting a file with `api.github.com` + `stageTweaks` + a `self-update` call, plus a `.github/workflows/release.yml`, made all four checks fail (planted files removed afterwards).

Current output:

```
check-fork: http(s) hosts in src/ and cli/
    FAIL                                             1  github.com/JaceThings/DiaTweaks  in src/dylib/Settings/Home.m
    FAIL                                             1  www.google.com/search  in src/dylib/Patches/Peek/Pages.m
    ignored (Cargo registry ID)                    105  github.com
    ignored (XML namespace name, never fetched)      1  www.andymatuschak.org
    ok                                               1  github.com
    ok                                               1  raw.githubusercontent.com
    ok                                               1  releases.diabrowser.com
    ok                                               6  www.diabrowser.com
check-fork: hosts outside diabrowser.com, releases.diabrowser.com, github.com/JaceThings/DiaTweaks/wiki and raw.githubusercontent.com/wiki/JaceThings/DiaTweaks (see FAIL above)
```

**For the user to decide.** Both are upstream code, and neither is release code:
1. `src/dylib/Settings/Home.m`: the "GitHub" and "Report a Bug" rows open `https://github.com/JaceThings/DiaTweaks` and `/issues/new/choose` in the browser (the "Wiki" row is allowed). Options: allow these two links in check-fork, or point them at the fork, or drop the two rows.
2. `src/dylib/Patches/Peek/Pages.m:74`: the Peek tweak falls back to `https://www.google.com/search?q=…` when Dia's default search engine has no usable template. This is upstream tweak code, so per the plan it's not edited without asking. Options: allow `www.google.com/search` in check-fork, or change Peek's fallback.

### Step 5: done (2026-11-11)

Ran: read-only review of `src/dylib/Patches/{Passkeys,ICloudPasswords,OnePassword}/*.m`, `src/dylib/StdPatch/Patch.m`, `src/diapatch/{Passkeys,PasswordsLaunch,PasswordsHost}.m` and `src/diapatch/main.m`. No code changed.

`defaultOn`: none of the three `DP_PATCH` initializers sets `.defaultOn`, so the field is zero (NO). `DPPatch.on` is the saved value, or else `defaultOn` (`Patch.m:102`). Keys: `passkeys` (order 100), `icloudPasswords` (101), `onePassword` (102), all in the Passwords section and marked Experimental. Behaviour unchanged.

Where they write when turned on:
- **Passkeys:** `~/Library/Application Support/DiaPatches/passkeys-extension/` (`manifest.json`, `page.js`, `bridge.js`, `worker.js`, extracted from the dylib) and `~/Library/Application Support/Google/Chrome/NativeMessagingHosts/com.diapatches.passkeys.json` (points at `DiaPatches/bin/diapatch`). While running, diapatch copies Dia's `global_key_value_store.db*` from `~/Library/Application Support/Dia/User Data` into the helper Dia's own profile.
- **iCloud Passwords:** Chrome `NativeMessagingHosts/com.apple.passwordmanager.json` and `io.github.fanckush.betterpasswords.bridge.json`, both pointing at diapatch. If another app's manifest is there and points at Apple's helper, it's moved to `DiaPatches/<name>.json` and put back when the tweak is turned off. diapatch clones the original Dia into `/Applications/.DiaTweaks/Dia.app` (Apple's helper only trusts browsers under /Applications) and uses `DiaPatches/passwords-launch/<pid>` (mode 0700).
- **1Password:** writes no manifest of its own (it needs 1Password's `com.1password.1password.json` there). It interposes `posix_spawn` so 1Password's helper starts through diapatch, which uses `passwords-launch/` and, if needed, the `/Applications/.DiaTweaks` clone.

When off: on launch, Passkeys deletes its extension folder and manifest, and iCloud Passwords removes only manifests that point at diapatch (restoring any it moved aside). It also kills any leftover original-Dia helper processes.

For the user to look at: `diapatch install` (run by install.sh) always calls `removeRelay()`. That runs `launchctl bootout gui/<uid>/com.diapatches.passwords` and deletes `~/Library/LaunchAgents/com.diapatches.passwords.plist`, `/Applications/.DiaTweaks` and the relay sockets **if they exist** (left by an older 1Password relay). These paths are outside the plan's allowed set, but only DiaTweaks' own leftovers get deleted.
