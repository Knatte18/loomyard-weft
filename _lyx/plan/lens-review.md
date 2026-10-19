# Lens review

fork0 was lost when its session was stopped mid-call, so the writer completed the reviewed files itself and ran this review once.
Every lens fork returned a report.

- row-reachability + branch-coverage, BLOCKING: card 1 rows that assert a failure leaves no workspace file fail once card 2 makes `CloneHub` write it — fixed as proposed: those rows remove the file first and assert it stays absent.
- row-reachability + forced-edits, BLOCKING: card 4's launcher-symlink refusal vector never refuses, because `containmentPath` leaves the leaf link unresolved and an empty outside dir makes every script removal a no-op — fixed as proposed: the launcher dir with its scripts moves outside the hub, and the script removal's `checkPathRequest` refuses with `CheckContainment`.
- row-reachability, BLOCKING: card 4's `addCase` rollback now removes the launcher-script blocker, an unpermitted change for an expectation with no permitted roots — fixed as proposed: the default expectation takes `slugPermittedRoots("verb-add-owner")`.
- criteria-coverage + row-reachability, MEDIUM: card 1 row 11's concurrent regeneration cannot fail without the lock, since both list the same pairs — fixed as proposed: two goroutines each add a different pair, then regenerate; the final file must list both.
- criteria-coverage + branch-coverage, MEDIUM: no spawn row reaches the lock-timeout or failed-write error — fixed differently: added `checkPrimeWriteFailure` (escaping symlink at the workspace path, spawn errors, no launch). A spawn lock-timeout row is not planned because it shares the write failure's return path in `Spawn` and would wait out the 5-second production wait (`PATTERN-test-speed`).
- criteria-coverage, MEDIUM: the Warn on a retired workspace file alone has no row — fixed as proposed: card 5 row 6 adds it.
- criteria-coverage + branch-coverage, MEDIUM: the sweep's skip of the retired file under an unresolvable prime has no row — fixed differently: Reconcile cannot reach it, because its `git worktree list` fails first, so a unit test, `TestSweepRetiredHubArtifacts_UnresolvablePrime`, calls the sweep on a no-git `locationkit` hub.
- criteria-coverage + branch-coverage, MEDIUM: Checkout's rollback and pre-flight exits have no row — fixed as proposed: card 2 row 4 adds a junction-failure rollback (arranged as `TestCheckout_JunctionFailureRollsBackBothSides`) that recreates a removed file, and a dirty-weft refusal that leaves it absent.
- branch-coverage + row-reachability, MEDIUM: Remove's pre-flight exit is untested, and row 7 cannot detect a wrong regeneration — fixed as proposed: row 7 removes the file first and asserts a refused Add and a refused Remove leave it absent.
- branch-coverage + criteria-coverage, MEDIUM: the lock file's `KindFileWritten` record and the `_board/.lyx` creation have no row — fixed as proposed: card 1 row 8.
- branch-coverage, MEDIUM: row 5 conflates the prunable skip with the missing-anchor-dir skip — fixed as proposed: row 5 reaches each at its own anchor.
- branch-coverage, MEDIUM: the sweep's foreign entry outside the anchor chain has no row — fixed as proposed: card 5 row 2 plants `_portals/x` on the nested hub.
- branch-coverage, MEDIUM: Reconcile's own regeneration being Warn-free is unasserted — fixed as proposed: card 5 row 6 checks the sweeping Reconcile's log.
- test-economy, MEDIUM: `checkPrimeWorkspaceSurvivesTopologyVerbs` duplicates card 2's rows and its doc becomes wrong — fixed as proposed: card 2 deletes it and its scenario step.
- test-economy, LOW: card 2 row 1 repeats card 1's folder-order row — fixed as proposed: row 1 keeps only the clone-writes-the-file assertion, on a `NewHub` hub.
- test-economy, LOW: card 1 row 7 repeats byte-level no-rewrite checks — fixed as proposed: row 7 asserts the mutation record only.
- test-economy + forced-edits, LOW: `//testtiming:keep` reasons go stale on `TestHubWorkspacePathAndFolders` and the portal/launcher accessor tests — fixed as proposed: card 1 and card 4 reword them.
- test-economy, LOW: every `TestRegenerateHubWorkspace` row takes its own hub — fixed as proposed: rows whose arrangements do not conflict may share one.
- test-economy, LOW: card 2 row 8's no-write assertion repeats card 1 — fixed as proposed: row 8 asserts the verb's outcome, the record and the Warn only.
- criteria-coverage, LOW: a settings-only change's mutation record is untested — fixed as proposed: card 1 row 7.
- criteria-coverage, LOW: the truncated-file rewrite is untested through the regeneration — fixed as proposed: card 1 row 7.
- criteria-coverage, LOW: no sweep row runs Reconcile from a task pair — fixed as proposed: card 5 row 1 also runs from a pair's location.
- criteria-coverage, LOW: no row shows Remove, Prune or spawn creating nothing under `_portals` — fixed as proposed: card 4 makes `TestVerbsRegenerateHubWorkspace` assert no `_portals` entry after each step.
- branch-coverage, LOW: the portal-only remnant's new `ErrPairNotFound` exit has no row — fixed as proposed: card 4 adds a row to `TestRemove_ReturnsPairNotFoundWhenNothingRemains`.
- branch-coverage, LOW: `restoreLaunchers`' mark-read failure, write failure and kept `junction_repointed` exits are untested — fixed differently: rows added for the write failure (read-only `_launchers/<AnchorRel>`) and the kept action. The mark-read failure is declared a fail-loud wiring guard instead of tested: it fails only on an unreadable warp config, which fails Reconcile's `git worktree list` first (`internal/gitrepo/branchmark.go:44-55`).
- branch-coverage, LOW: the Warn's behavior with an escaping `_portals` is unspecified — fixed as proposed: card 5 reads it through an `os.Root` at the hub as no artifact, and row 6 asserts no Warn.
- forced-edits, LOW: edited tests keep names that the change makes wrong — fixed as proposed: card 1 replaces `TestWriteHubWorkspace_RefusesEscapingLaunchersSymlink`, and card 4 replaces four portal-named tests under launcher-only names.
- forced-edits, LOW: unlisted comments name the portal path — fixed as proposed for the `remove.go` header, `resolvePathOwnership`, `TestGate_ZeroValueDeclarationsAreRefusals`, the headers above `TestCorrespondenceIndex` and `TestRemove_DoesNotDeleteOutsideHubThroughLauncherSymlink`, and `fslink#RawTarget` (`internal/fslink` joins the verify). Two are disputed, premise false:
  - `internal/fabricengine/livestate_doc_test.go:130-140` is a record of a past cell run, true of that run.
  - `internal/lyxcwd/enforcement_test.go:232-235` stays true: `_portals` remains a token owned by `internal/fabricengine` (D6).
- forced-edits, LOW: README still says `_launchers` holds workspaces, and `docs/overview.md:645` names portal entries — fixed as proposed: card 1 updates the README tree, and card 4 updates the overview's batten note.
- forced-edits, LOW: `SANDBOX-FABRIC-SUITE.md:340`'s exact hub listing lacks the workspace file — fixed as proposed: card 4 adds `<RepoName>.code-workspace` to it.
- row-reachability, LOW: card 2 row 6 omits the serial `FABRIC_SKIP_PUSH` setting its arrangement needs — fixed as proposed.
- row-reachability, LOW: card 1 Uses omit `lock#AcquireWriteLock` and `Topology.AddLocal` — fixed differently: `lock#AcquireWriteLock` was added. `Topology.AddLocal` stays out, because card 4 edits its doc comment and `lyx loom validate-plan` refuses it as `uses-later-target`.
