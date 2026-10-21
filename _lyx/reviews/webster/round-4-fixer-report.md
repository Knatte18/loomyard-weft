# Round 4 fixer-report

## Changes

- F1 (BLOCKING), fixed as proposed, commit `03f5b28a`: `reindexIfPacksChanged` in `internal/gitrepo/gogit.go` now calls `storer.HasEncodedObject(plumbing.ZeroHash)` right after `storer.Reindex()` under the exclusive lock, so go-git's unsynchronized index rebuild finishes before any shared-lock reader enters.
  The function's comment says why, one sentence per line; the gate's comment lint flagged a first draft that wrapped a sentence across two lines, and commit `b19d9fa8` breaks it at the sentence boundary.
  `TestGoGit_StandaloneRepo` is unchanged, and `lyx gate test ./internal/gitrepo` passed.
- F1 follow-up, commit `9fce9642`: the reactive-reindex paragraph in `internal/gitrepo/doc.go` names the rebuild among the exclusive-lock steps (self-check finding).
- F3 (LOW), fixed as proposed, commit `32a9a8fc`: `timing.IdleCycles` in `internal/reedcli/watchdog_integration_test.go` is `int(2 * watchdogTestWait / watchdogTestCycle)`, and the comment says the window spans the Up call plus the tests' wait budget.

## Disputed

None.

## Deferred

- F2 (MEDIUM, decision): not fixed.
  Restoring main's five overview lines takes the overview from 15736 to 16003 bytes, against the 15743-byte base.
  The two declared entries, `PATTERN-push-failure-visible` (117 bytes) and `PATTERN-origin-reconcile` (143 bytes), total exactly 260 bytes with their newlines.
  Removing both entirely would reach only 15743, so no shortening of them fits.
  The finding's own fallback is to ask `ly:orch` whether to raise the declared-change list; my instructions bar any message about the review, so the question goes to the parent through this report.
  `PATTERN.md` and `pattern/PATTERN-recipe-setting-lookup.md` are untouched.

## Self-check

- fix-consistency, LOW: `internal/gitrepo/doc.go` still said only the fingerprint check and reindex take the exclusive lock — fixed as proposed.
- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.
