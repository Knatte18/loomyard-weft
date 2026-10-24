Task: lens fix-affected-set. I reran the set searches for F2, F4, F5 and F6; F1 and F3 have no set premise to rerun.

Unchecked: F1 and F3 (their premise is one function and one test table, with no set to search).

Findings:

- **LOW**: `contracts/stencils/webster/webster-body-implementer.md:39`. This is not a miss by the fix. Line 39 holds two sentences ("...full grammar. Make exactly the changes...") and no diff hunk touches it. F6 covered only the line the diff edited, so it's out of scope; I mention it only because it's the same defect class.
- No finding for F5. The offender switch at `runlevel.go:1017` was the only site producing "begun, not recorded terminal" outside tests. A waiting batch reaches `beginbatch.go:553` ("begun and not terminal") only through an existing report, and `recordWaiting` archives that report, so the path is not reachable for it.
- No finding for F2. The other `RecoveryRetry` caller, `beginbatch.go:556`, still folds a git fault into false, but there it only picks the remedy text and refuses either way, so no dead end is misreported.
- No finding for F4. The doc comment sits directly above its function again, and `TestRecordBatch_NeedsParentWaits` has its own.

Tool calls:
1. Grepped `internal` and `contracts` for "begun, not recorded"/"not recorded terminal" (F5), to find sibling messages for a waiting batch; my context lacked the repo-wide hits.
2. Grepped the implementer stencil for sentence boundaries and ran `git diff --stat` (F6), to see whether other multi-sentence lines were in the diff; my context lacked the file's other lines and the diff list.
3. Read `beginbatch.go:545-570` (F2), to check the other `RecoveryRetry` caller; my context lacked that file.
4. Grepped `!bs.Terminal` and `Terminal {` across `internal/websterengine` (F5), to find other non-terminal reporters; my context lacked the repo-wide hits.
