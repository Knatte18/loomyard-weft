# Fixer-report, darn round 1

## Changes

- F2: `internal/boardengine/doc.go` Intake section now says intake reads the hub's own GitHub repository and lists open issues of that repository; the comments at `internal/boardengine/intake.go` (header, `IssueComment`) and `internal/boardengine/task.go` (`Issues` field) say "GitHub issue" instead of "inbox issue".
  The `InboxIssue` identifier is unchanged.
- F3: `internal/boardcli/cli_test.go`, `stepBoardPathResolution` gained two subtests: `intake list` under `--board-path` is refused naming a worktree of the hub, and `intake list` from the fixture hub (no binding, no origin) is refused naming the binding file, which pins the `HubRepoURL` wiring.
  The second subtest asserts the text "no usable origin remote" rather than the binding file's identifier, because the fabric-vocabulary scan (`TestEnforcement_FabricVocabulary`, which failed the verify gate) refuses a bare warp token in `internal/boardcli/cli_test.go`.
  The offline guarantee for other verbs is covered by the existing `list --board-path` subtest, which succeeds only if `list` never calls `hubRepo`.
  Fixed with an own change: the review proposed extending the intake tests, but those call the handlers directly and never reach the closure, so the check goes through `Command()` instead of a new export.
- F4: `internal/boardcli/intake_test.go`, rows `RepositoryUnresolvedImport` and `RepositoryUnresolvedClose` in the intake table, which runs through `intakeCommand`; the unresolved-repository rows now also assert no GitHub mutation and an unchanged `board.json`.
  Fixed with an own change: `TestIntakeImport_RefusedBeforeAnyWrite` and `TestIntakeClose` call the handlers directly and skip the `hubRepo()` branch.
- F5: `internal/fabricengine/hubrepourl_integration_test.go` asserts the neither-source error names `fabricengine.WarpBindingFileName` and `origin`.
- F6: `internal/orchcli/doc.go` step 5 says friction findings are filed as LoomYard issues, which reach the board through intake when the operator asks for it.
- Self-check follow-up: `internal/boardengine/intake_test.go` header reworded to match `intake.go`.

Gate: `lyx gate test --tags integration` over `boardcli`, `fabricengine`, `boardengine`, `orchcli` passed; the verify gate then failed `TestEnforcement_FabricVocabulary` on the F3 subtest, and after the repair `boardcli` and `lyxcwd` pass.

## Disputed

None.

## Deferred

- F1: the only sound change is recording or refusing the parent correction with `lyx loom decision add`, which is the parent's and lies outside the fixer's write surface; the hub-wide orch rules live on Bolt and are written only through `lyx orch rules`.
  No code or stencil change made.

## Self-check

- fix-affected-set: no findings.
- fix-consistency: LOW, `internal/boardengine/intake_test.go` header still said "inbox issues"; fixed as proposed.
- fix-proposal: LOW, F4 rows sit in a test whose name says list; left as is, since the name appears in the measurement report `docs/benchmarks/test-redundancy.md` and the rename would stale it.
- fix-proposal: LOW, the non-intake offline guarantee is covered only indirectly; left as is, since the existing `list --board-path` subtest fails if `list` resolves the repository.
- fix-code-correctness: no findings.
