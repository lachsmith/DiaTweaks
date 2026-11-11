# Upstream sync log

Syncing the fork (origin, github.com/lachsmith/DiaTweaks) with upstream (github.com/JaceThings/DiaTweaks) while keeping the fork's security changes (f09c7bf, ec01611, 0f3f9bc: no pre-built DiaTweaks updates, only Dia's own signature-checked updates, quarantine cleared after signing).

## Branch naming

The fork has no `main` branch. Its default and only branch is `my-version`. On the user's instruction (2026-11-11), `my-version` stands in wherever the plan says `main`:
it was pulled, tagged `pre-sync-2026-11-11`, and `sync-upstream` was branched from it. The PR goes into `my-version`, and abandoning the sync means switching back to `my-version`.
No branch is created or renamed to `main`, and nothing is committed to `my-version` directly.

## Checklist

- [x] 1. Prepare repo, tag, branch, create this log
- [ ] 2. Merge upstream/main and resolve conflicts
- [ ] 3. Compare quarantine handling
- [ ] 4. Write scripts/check-fork and FORK.md
- [ ] 5. Confirm defaultOn is NO for Passkeys, ICloudPasswords, OnePassword
- [ ] 6. make, make test
- [ ] 7. Summary, then install on approval, diapatch status
- [ ] 8. Push and open PR into my-version on approval

## Current state / next action

Step 1 done. Next: step 2, `git merge upstream/main` on `sync-upstream`.

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
