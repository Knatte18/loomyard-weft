---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "Code, doc comment, doc.go and test rows changed as proposed. The third named edit, the sentence in `_lym/landing/summary.md`, was not made: this round never writes `_lym` beyond its report files, so the summary still says every ignored path counts as work."
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
---

# Round 1 fixer report

## Changes

- F1: `ignoredWork` skips the paths lym writes. In the prime these are the anchored `warp_exclude` paths from the hub's fabric config and `.vscode/settings.json` and `.vscode/tasks.json`; other files under `.vscode/` are still named. In the Bolt checkout they are `board.lock`, `board.push.lock` and the render manifest, whose name moved to `fabriclayout.BoardRenderManifestFile`. A hub whose anchor or config cannot be read skips fewer paths, so more is named.
- F1, the facts the fix left unverified: no current code writes `.gitrepo-push.lock`, `*.swaplock`, `*.json.lock` or `settle-record.patch.lock` at a store root. Current locks go under `.lym/locks` (`internal/lock/tracked.go`), so none of them is skipped.
- F1, consequence for the parent: this applies to `board.lock` and `board.push.lock` too, which are leftovers of the earlier layout and are skipped as the review asked. An older hub that holds `.gitrepo-push.lock` or `board.json.swaplock` in `_bolt`, as this machine's live hub does, still has a plain `--reset` refused over those empty files.
- F2: commits only the Bolt checkout's HEAD holds are named.
- F3: a hub whose work cannot be read is refused with the move aside.
- F4: the grouped commit carries the `Root-Cause:` trailer.
- F5: the S6 step of the core suite names `--reset --discard-work`.
- F6: a row in `TestCommand_RefusesBeforeAnyHubWork`.
- F7: a row with seven ignored files.
- F8: `RefuseHubWithoutBolt` names the discarding clone and the work it loses, with a new reconcile test row.
- F9: only a missing `.git` reads as no repository.
- Gate findings: the wrapped comment lines the gate named in `internal/fabriccli/doc.go`, `internal/fabricengine/clone.go` and `internal/fabricengine/doc.go` now break at sentence or clause ends, in a commit of their own. No code changed.

## Self-check

- fix-consistency 1, MEDIUM (doc states lym writes the root lock files): fixed. `doc.go` now says they are earlier-layout leftovers and that other leftovers are still named.
- fix-consistency 2, LOW (`fabriccli/doc.go` not updated for F8): fixed.
- fix-consistency 3, LOW (summary still states the old rule): deferred, `_lym/landing/summary.md` is outside this round's write surface.
- fix-code-correctness 1, LOW (sandbox sentence too broad): fixed.
- fix-proposal 1, MEDIUM (skipped files can hold operator content): deferred, a tradeoff for the parent. `doc.go` now states the limit. See below.
- fix-proposal 2, MEDIUM (a hub in use is still named a clone that is refused): fixed. The refusal names what uses the hub and the ways past it. No test row covers this branch.
- fix-proposal 3, MEDIUM (the discarding clone was the only way forward): fixed. The refusal also names pushing, committing or moving the hub aside.
- fix-proposal 4, MEDIUM (same as fix-consistency 1): fixed with it.
- fix-affected-set 1, LOW (`--reset` flag help promises an idempotent re-clone): fixed.
- fix-affected-set 2, LOW (only some leftovers are skipped): fixed with fix-consistency 1 and stated under Changes.

## Verification

`lym gate test` passed with 0 failures at tier 0 and tier 1. Tier 0 was run again on `internal/fabricengine` and `internal/fabriccli` after the comment repair; tier 1 was not, since the repair changes comments only. `internal/boardengine` and `internal/vscode` were run before the self-check, which did not touch them.

## Disputed

None.

## Deferred

- The `_lym/landing/summary.md` sentence "Every ignored path counts as work" (F1, fix-consistency 3). Whoever owns the summary should narrow it to: every ignored path counts but links and what lym writes itself.
- fix-proposal 1: should a plain `--reset` remove `.vscode/settings.json`, `.claude/settings.local.json` and `warp_exclude` entries unnamed? lym merges its keys into the first two and keeps the operator's, and the operator can add any path to `warp_exclude`. The review asked for the skip, and naming these files whenever present would refuse a plain `--reset` of nearly every used hub. Telling operator content from lym's inside a file needs a design the review did not ask for.

## Rulings

None.
