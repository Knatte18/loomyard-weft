---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: PATTERN.md:1
    summary: The merged PATTERN.md is 16576 bytes, 192 over the 16384 cap, so TestOverview_RealFilePassesCheck in internal/pattern fails
---

### [BLOCKING] Merged PATTERN.md exceeds the overview size cap

**Location:** `PATTERN.md` (whole file); cap declared as `MaxOverviewBytes` at `internal/pattern/check.go:21`.

**Issue:** Publish's verify failed on `TestOverview_RealFilePassesCheck` in `internal/pattern` with `oversize-overview PATTERN.md (line 0): overview is 16576 bytes, over the 16384 cap`.
I reproduced it with `lyx gate test ./internal/pattern ./cmd/lyx`: `internal/pattern` fails the same way and `cmd/lyx` passes.
The cause is the combination, not either side alone.
`main` at the merge-in (`8003f2d88`) is 16380 bytes, 4 under the cap.
The branch's own `PATTERN.md` change (`git diff 8003f2d88..HEAD -- PATTERN.md`) adds the `## Daemons and waits` section with the `PATTERN-daemon-wakeups` entry and extends the `PATTERN-leaf-packages` entry with `fswatch`, 196 bytes in all.
Before the merge the branch file was 16381 bytes and passed only because `main` had not yet added its three new entries (`PATTERN-hub-store-gc`, `PATTERN-gogit-read-helper`, `PATTERN-gate-slot`), offset by its own trims.
The merge-in commit `523ec97ad` resolved `PATTERN.md` cleanly in text but left it 192 bytes over the cap.
Nothing else in the Publish log failed.

**Fix:** Cut at least 192 bytes from `PATTERN.md` and keep every other `internal/pattern` check green (the 320-character entry-line cap, the `[background]` link resolution and the orphan-background check).
The `PATTERN-daemon-wakeups` entry and the `fswatch` addition to `PATTERN-leaf-packages` are already minimal; dropping the section heading saves about 22 bytes and is not enough alone.
Tighten the wording of the longest existing entries without dropping their rule, with the cut spread over several entries so no entry loses its trigger or its requirement.
Candidates are the long `PATTERN-push-both-sides`, `PATTERN-shuttle-stop`, `PATTERN-parent-directive`, `PATTERN-review-round` and `PATTERN-batcher-registry` lines, and any parenthetical whose content already lives in the entry's background file.
Re-run `lyx gate test ./internal/pattern` to confirm the file is at or under 16384 bytes.

## Review of the rest of the range

Beyond F1 I read the branch's own non-test diff for the `fswatch` leaf, the reed watch loop and name-repair backoff, the discover signal, the orch watcher's hold logging and hook-delivered pointer, the shuttle readiness reading and the poll floor, plus the PATTERN, budget and docs edits.
I found no further defect against the plan's cards.
The `cmd/lyx` suite, which holds the new `daemonwakeups_test.go` guard and the test-budget check, passes.
