# Harden fabric's one-repository layout: archive tags and teardown

This change fixes the gaps a hardener campaign found on one-repository hubs, where one origin holds code under `refs/heads/` and `refs/tags/` and the records and board under `refs/lym/`.
The campaign drove every mutating `lym fabric` verb inside sandbox hubs made with `lym harden sandbox new --single-repo`.

## What changes

- **Archive tags no longer land under plain `refs/tags/`.**
  In a one-repository hub, the code store's archive tags go to `refs/lym/code-tags/` on origin.
  The records store's stay under `refs/lym/tags/`.
  No fetch refspec maps `code-tags/`, so neither store lists or fetches the other's tags or commits.
  `archiveTagNamespaceAt` decides the prefix from the records sibling of the code store's prime worktree, whatever the origin URL looks like.
  Pushing, listing (`remoteTagsUnder`) and owner-archive reads all go through it.
  Any other repository keeps `refs/tags/`.
- **A remote teardown that did not finish is now visible and finishable.**
  The command writes a pending-teardown record (`pendingteardown.go`) before it hands the job to the detached child.
  The child removes the record when every remote step succeeded, and keeps it with the step's way forward when one failed.
  A child killed outright leaves the record in place.
  `lym fabric prune` now reports `pending_teardowns`, and the docs say `lym fabric status` lists them too; each entry names what finishes it.
  That is the hand push of an archive tag only this machine holds, where one is owed, then the re-run of the verb.
- **A re-run teardown no longer deletes a branch whose tip only a local tag holds.**
  Every remote teardown job first pushes the `archive/<slug>/*` tags an earlier teardown made locally and origin lacks (`pushLocalArchiveTags`).
  A failed push there stops the job before any delete on origin.
- **A records store that lost its fetch refspecs fails loudly.**
  `Bolt.pullFirst`, the weft pull arm and classified pushes read the origin layout first.
  The error names the refspecs to restore, where before the fetch exited clean and the upstream then failed to resolve with no way forward.
- **A stopped seed is named by plain `lym fabric clone`.**
  Without `--single-repo`, an origin holding records branches and no board is refused, and the refusal names the `--single-repo` seed as the way forward.
- **A clone into a directory that does not exist is refused by name.**
  Git otherwise fails as if git itself were missing.
- **Detached pushes passing the sandbox guard are narrower.**
  A task session's detached push with `--weft-path` or `--warp-path` passed on the flag alone.
  It now passes only when every path is the role's own worktree, that worktree's records sibling, or a path under its own sandbox root.
  The refusal text now names the sandbox route and the stop-and-report route.
- **Unpushed-commit detection** for a branch never given an upstream now reads `origin/<branch>`.
- **`lym gate test --no-carry`** runs every tier even where a carried green record covers the unchanged tree.
  The hardener's live gate call passes it, since a carried record starts no test binary and proves nothing.

Tests cover each behaviour: unit and integration tests in `internal/fabricengine` and `internal/fabriccli`, plus the one-repository scenario in `tools/sandbox/SANDBOX-FABRIC-SUITE.md`.

## Design decisions

- The code store's archive tags get their own prefix rather than sharing `refs/lym/tags/`, so the records store's fetch refspec cannot pull a code commit into the records repository.
- The pending-teardown record sits beside the slug's teardown lock.
  `PendingTeardowns` probes that lock shared, so a teardown still running is not listed and the probe holds back no push of the pair.
- `prune` reports pending teardowns but never runs them.

## To check by hand

- A killed clone still leaves a hub with no named way forward (`killed-clone-leaves-hub-without-way-forward`, MEDIUM).
  The last campaign round did not close it and no fresh review has seen the result; check what `lym fabric status` and `lym board sync` say in a hub whose `lym fabric clone` was killed mid-way.
- The `refs/lym/code-tags/` prefix is new.
  Archive tags an earlier build pushed under plain `refs/tags/` are not moved; look at an existing one-repository origin for `archive/*` tags there.
- No regression check ran for this task.

## Merge danger

**Door:** one-way
**Blast radius:** fabricengine
