```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. Below, per behaviour, the test that pins it (or none), then findings. I used 4 tool calls, all of them greps or diffs of test files.

## Per behaviour: pinning test

- **Beam-Await D9 order and failure paths:** `TestBeamAwait` rows `AcceptedRequestThatPassesLandsAndClearsEveryFile`, `FailingRecheckLiftsTheHoldRestartsTheDobbyAndPostsTheBlockers`, `FailingRestartKeepsTheMarkerAndNamesBeamStart`, `FailingParentAlignWriteKeepsTheRequestForTheNextWake`, `FailingBeamAwaitWriteAfterARefusalKeepsRequestAndMarker...`, `FailingDobbyStopIsAFailedRecheck...`, `FailingSnapshotWriteIsAFailedRecheck`, `RecheckErrorIsAFailedRecheck...`. Pinned (`internal/pawlengine/beamawait_test.go:31-250`).
- **Beam-Await other behaviours:** each has a row in the same test.
  - The other-run request is removed.
  - A re-entry at Parent-Align posts the interrupted notice.
  - A re-entry with a live dobby posts nothing.
  - A re-entry with the dobby down posts the dobby-down notice (the Added-after-Discussion item).
  - The first entry writes the await marker.
  - A beam owned elsewhere yields.
  - A failed entry write is retried.
  - A failing notify leaves the row waiting.
  - The timer back-off, a watch event and a cancelled context have `TestNextAwaitWait`, `TestBeamAwait_WatchEvent` and `TestBeamAwait_CancelledContext`.
  - Pinned.
- **Beam-Up and Beam-Down:** `TestBeamUp`, `TestBeamUp_RerunIsIdempotent`, `TestBeamDown` (a failing stop is still Done). Pinned.
- **Land request file:** `TestLandRequest`. Pinned.
- **`runproc.EndRole`:** `TestEnd` row "ending one role leaves the other roles..." (`internal/runproc/end_test.go:245`). It drives the unexported `e.end(..., endsOnlyRole(role))`, not the exported `EndRole`; the one-line wrapper itself is unpinned but trivial. No finding.
- **`Owners.Effective`:** `TestOwnersEffective` (`internal/boardengine/owners_test.go:12`). Pinned.
- **Leftover task `owner_machine` dropped on load:** the owner-parse fixture (`owners_test.go:22`) and `task_test.go:159` (stores none on note, task or grouping project). The drop at `migrate.go` on load and write-out has no test that I found. See T1.
- **Owner setter writes only root and beam owners, sets beam flag and `recipe: pawl`:** `TestSetOwners` (`ownerset_integration_test.go:153`). Pinned.
- **Run lock and landing hold:** `TestRunLock` has beam rows at Parent-Align and at Beam-Await (`runlock_test.go:135-152`), and `TestRunHolds`. Pinned.
- **`beamForkPrecondition` and the member's Worktree-Create hold refusal:** `beamfork_test.go:68,82` (hold vs Beam-Await) and `TestMemberHoldRefusal` (`beamfork_test.go:134`). Pinned.
- **Finalize re-check ways forward:** `TestFinalize_LandCheck` rows cover pair, in-flight elsewhere, no status, forked pair, every blocker, failed read and empty list. Pinned.
- **Finalize owner halt:** `finalize_owner_test.go`. Pinned.
- **Adopt refusals and ordering:** `TestAdopt` rows (`adopt_test.go:217+`) cover task, unknown slug, note, done task, moving records, no slug and a task naming its beam. A refusal for a grouping project, for a non-beam naming `lyx beam create`, and for a nested beam left alone is not visible in my excerpt. See T2.
- **Adopt resumes the pawl run when this machine owns both beam and parent, in either order, and reports owned-elsewhere:** the fixtures at `adopt_test.go:~230-240` (`ownerHalted`, nested beam with `EntryOwnedElsewhere`) show the setup. I did not verify the row names. Probable.
- **`lyx s`:** `shortcut_test.go:137-152`. Pinned.
- **Removed verbs:** `TestRemovedVerbs_NamedNowhere`. Pinned.
- **`beam start --all` and a failing middle target:** `TestStartBeams` (`internal/beamcli/start_test.go:43-65`). Pinned.
- **Beam start refusals (D3) and the D2 fallback:** `TestStart_Refusals` covers an owned-elsewhere beam, the root, an anchor move and a landing hold. `TestStart_ResumesTheRecordedSession` and `TestStart_StarterErrorFailsAndSavesNoState` cover the resume. A row asserting the envelope names the abandoned session is plausible at `start_test.go:1240` ("a resume and the fresh launch after it both failing"); I did not verify it.
- **Orch watcher stops on another machine's owner:** `TestWatcher_StopsTheOrchOfABeamOwnedElsewhere`. Pinned. The failed-read-once-per-episode logging is not visible. See T3.
- **Batten start guard for a beam project:** `TestStartGuard_Refuse` and `TestArmSeed_CancelMarkerRefusesTheWrite`. Pinned.
- **Batten cancel of a beam:** `TestCancelRun` (`cancel_test.go:241`) plus `cancel_test.go` changes. Plausibly pinned; I did not open the rows.
- **Can-land notice:** `TestLandNotice` (`internal/dobbyengine/landnotice_test.go:35`). Presumably covers the episode and re-arm, with its state persistence. Not row-verified.
- **Parent dobby `max_runs` slot for a pawl run:** `dobbyengine` has `starts_test.go` and `passes_test.go` changes (+26 and +4 lines). Not row-verified.
- **`beam land` refusals:** `TestLand` (`beamcli/land_test.go:81`). Pinned.
- **`beam create`:** `TestCreate` and `TestCreate_StepFailureNamesTheRerun`. The `--anchor`-across-resume path through Worktree-Create is not verified from the names. See T4.
- **Battenshed InnerRun owner and quiet notice:** `TestInnerRun_OwnerCheck`. The "no quiet notice at Beam-Await" row, its driver-dead and respawn-cap behaviour, and the Beam-Await interrupted-landing notice are not visible in the test names I saw. See T5.

## Findings

**T1 — MEDIUM, scope (test gap): the load-time drop of a task's `owner_machine` has no test of the written-out board.**
- Location: `internal/boardengine/owners_test.go:22`, `internal/boardengine/migrate.go`.
- The criterion "gone after the next board write" is covered only where `ParseOwners` ignores it and where an upsert stores none (`task_test.go:159`). Neither covers the load-and-write round trip that removes a stale key from `board.json`.
- Evidence: the `grep owner_machine` results in `owners_test.go` and `task_test.go` only.
- I did not open `migrate_test.go`; if it covers the drop, reject this.

**T2 — LOW, scope (unconfirmed gap): adopt's grouping-project refusal and non-beam refusal may have no row.**
- Location: `internal/adoptcli/adopt_test.go:217-288`.
- The excerpt shows a task row, "RefusesATaskNamingItsBeam", but no visible rows for a grouping project naming its beam or `root`, or for a non-beam project naming `lyx beam create <slug>`. `PATTERN-refusal-way-forward` requires a test reaching each way forward.
- The grep was cut at 40 lines, so check the rest of `TestAdopt` before raising this.

**T3 — LOW, scope (unconfirmed gap): the watcher's "failed owner read is logged once per episode and does not stop the orch" has no visible test.**
- Location: `internal/orchengine/watcher_owner_test.go:13`.
- The only test function is `TestWatcher_StopsTheOrchOfABeamOwnedElsewhere`.
- The D7 negative case, where a failed read must not stop the orch, would be a row of that test. I did not open the file, so this is unconfirmed.

**T4 — LOW, scope (unconfirmed gap): `--anchor` surviving a Worktree-Create resume into `Topology.Add`.**
- Location: `internal/beamcli/create_test.go` and the battencli or loomcli Worktree-Create tests.
- This is the criterion "also across a resume of Worktree-Create". I saw no test name for it in the files grepped, and the resume path lives in `pawlwiring_test.go` or the battencli run record.
- Unconfirmed.

**T5 — MEDIUM, scope (unconfirmed gap): batten InnerRun's "no quiet notice at Beam-Await", the driver-dead notice at Beam-Await and the interrupted-landing exclusion have no visible test.**
- Location: `internal/battenshed/innerrun_owner_test.go`, `internal/battenshed/notice.go`.
- The diff touches `innerrun.go` (6 lines) and `innerrun_memory.go`. The new test in this area is `innerrun_owner_test.go`, which is the owner check.
- The criterion "a Beam-Await driver found dead sends one driver-dead notice, also when it dies after re-entering the row and writing its status" needs its own test. I did not search `notice_test.go`.
- If a quiet-notice test exists in `notice_test.go` or `innerrun_test.go`, reject this.

**T6 — NIT, test economy: possible overlap in `TestStart_*` in `orchcli/start_test.go`.**
- Location: `internal/orchcli/start_test.go:814-916`, `:1271`.
- `TestStart_Refusals` has rows for an owned beam, the root, an anchor move and a landing hold. `internal/beamcli/start_test.go` also owns the D3 refusals, since the refusal moved to `beam start`. If both assert the same refusal text, one of them duplicates the other.
- I did not open the bodies, so this is a suspicion only and I am not raising it as a finding.

No new test was found that is a per-helper test duplicating a public-surface test, or a review-fix test added for a non-gap. I did not check the "no assertion on a stencil's static prose" rule beyond `removedverbs_test.go`, which asserts the removed names appear nowhere.

## Tool calls

1. `grep` of `func Test` and subtest names across the new test files, because the manifest holds no test files.
2. Row names of `beamawait_test.go`, `adopt_test.go`, `end_test.go`, `start_test.go` and `runlock_test.go`, because those rows were not in my context.
3. A grep for `EndRole`, `owner_machine`, `max_runs`, the Finalize rows, Worktree-Create hold tests, shortcut tests and beamcli start rows, to locate pinning tests.
4. `git diff` of `runproc/end.go` and `end_test.go`, to confirm how `EndRole` is tested, since the grep found no `EndRole` call in a test.
