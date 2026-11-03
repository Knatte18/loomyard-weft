# Round 1 fixer-report

## Changes

- **F1 (MEDIUM), fixed as proposed.**
  - Added `shedrun.SettleRecordsRel` beside `SeedRel` and `StatusRel` in `internal/shedrun/paths.go`, with a row in `paths_test.go`.
  - `statusCommitPathspec` (`internal/loomcli/wiring.go`) and `battenRunCommitPaths` (`internal/battencli/commitstatus.go`) append it when the settle log exists; rows added to `wiring_commitstatus_test.go` and `commitstatus_test.go`.
  - Reworded the `runStateExemptRels` comment and the `internal/preflight/doc.go` paragraph: the status commit carries what it stages, and the pair teardown's commit of uncommitted record changes carries the rest.
  - Stop markers stay out of the status commit; they are removed before each spawn and the teardown commit covers them.
- **F2 (LOW), fixed as proposed.** `SettleRecords` no longer claims a dirty-path check counts the file; `RunsRootRel` names Preflight's worktree-clean check beside fabric's `Add`.
- **Self-check fixes.** The `loomCommitStatusDeps` doc no longer enumerates the pathspec's contents; the `runStateExemptRels` comment no longer claims the status commit stages the attempt directories on the batten path.

Commits: `e7512f10` (F1, F2 together, since both touch `paths.go`) and a follow-up for the self-check fixes.
Gate: `lym gate test` over `shedrun`, `loomcli`, `battencli` and `preflight` passed at tier0.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-consistency, LOW, `loomCommitStatusDeps` doc stale: fixed as proposed.
- fix-consistency, LOW, preflight comment "where it can" overstates the batten path: fixed with own wording ("carries the files it stages").
- fix-affected-set, LOW, `loomCommitStatusDeps` doc stale (duplicate of the fix-consistency finding): fixed as proposed.
- fix-affected-set left `internal/battencli/cleanup.go:80` unchecked; it is a removal path, not a status or records commit.
