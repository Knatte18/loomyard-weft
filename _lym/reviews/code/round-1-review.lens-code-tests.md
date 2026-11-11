```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test row that pins it, or none.

## Behaviour to test map

**adoptcli, `--here`** (`TestRunAdopt` rows in `caller_test.go`)
- **Owned prime → `owned_here`, no worker, no record, one pull:** pinned by `HereInAPrimeThisMachineOwnsEndsOwnedHereWithNoWorkerOrRecord`.
- **Owned-here beside another subject's unfinished record:** pinned by `HereOwnedHereBesideAnotherSubjectsUnfinishedRecordEndsOwnedHere`. That row asserts the pull count and the outcome, but not `wantNoRecord`.
- **Bolt pull skipped → local Bolt and a `warnings` line:** pinned by `HereReadsTheLocalBoltWhenThePullCannotReachOrigin`.
- **Foreign prime adopts `root`; foreign beam adopts the beam:** pinned by `HereInAForeignOwnedPrimeAdoptsRoot` and `HereInAForeignOwnedBeamAdoptsTheBeam`, which check the spawn args.
- **Worker killed at deploy → resumes the record, no pull:** pinned by `HereAfterAWorkerKilledAtDeployResumesTheRecordWithoutAPull`.
- **Beam with no owner of its own follows a running `root` adopt:** pinned by `HereInABeamWithNoOwnerOfItsOwnFollowsTheRunningRootAdopt`.
- **Record past `check` with a held-back project takes up no worktree:** pinned by `HereOnASubjectPastCheckBesideAHeldBackProjectEndsOwnedHere`.
- **Task worktree → `no_subject`, no record:** pinned by `HereInATaskWorktreeDoesNothing`.
- **`--here` with a subject is refused:** pinned by `HereWithASubjectIsRefused`.
- **`--here` with `root` or `--all` is refused:** none. The `call.Root || call.All` branch has no row.
- **`--here` with `--discard` is refused:** none. The `flags.Here` term in the discard condition at `caller.go:253` has no row.
- **Record's request merged with the call's flags (`--no-orch` from `lym s`):** none. The resume rows run with an empty call request and only check the spawn args for the subject. A `mergeRequest` that dropped the call's flags would still pass.
- **`TakesUp` returns false for a record that is not unfinished (done):** none. Only the `check`-done case has a row.
- **Beam with a foreign owner reached through `Owners.Effective` and `ParentOf` when it has no owner of its own, no record:** none. The `gamma` row goes through the record path, and the `alpha` rows use a beam with its own owner.
- **`PullBolt` returning an error is refused with a message:** none.
- **`cli.go` wiring (`resolveCall`, `deps.Worktree = location.WorktreeName`, the `--here` and `--archives` flag registration, `hubCallerDeps.PullBolt`, `hubArchiveDeps`):** none. Every adoptcli test builds `callerDeps` or `archiveDeps` by hand, and no test drives `Command()`. A wrong worktree name would make every `--here` end `no_subject`, and no test would fail.

**adoptcli, `--archives`** (`TestRunArchives` in `archives_test.go`)
- **Root and beam listing with per-slug grouping, subject mapping and `Look` command:** pinned by the first two rows.
- **A slug the board no longer holds moves with its parent's subject:** pinned by the `gone` slug in the first row.
- **Empty list, and refusal of no subject, an unknown slug or a task (naming the mover):** pinned.
- **Other flags and `--here` refused, and the read failure naming the rerun:** pinned. Only `KeepLocal` is exercised, though the condition also covers `All`, `DropLocal`, `NoOrch` and `Discard`.
- **Duplicate subjects collapsed in the `subjects` field:** none.
- **A `ParentOf` failure inside `keep`, joined into the read error (`placeErr`):** none.
- **Writes no record:** not testable here, since `archiveDeps` has no record seam.

**adoptcli, end envelope**
- **`archives` command names the subjects whose owner write stood, sorted, with `root` last:** pinned by `OwnerWritesThatStoodNameTheArchivesCall` in `cli_test.go`.
- **The field is absent when no owner write stood:** pinned by the existing rows, since `wantArchives` is empty there.

**fabricengine `ReadOwnerArchives`** (`TestReadOwnerArchives`, `ownerarchive_integration_test.go`)
- **Code tag whose commit the branch on origin lacks is listed and fetched as a local tag:** pinned.
- **Records tag listed with the records branch recorded in `Branch`:** pinned.
- **Code tag the branch on origin holds is not listed:** pinned.
- **`keep` refusing a slug excludes it:** pinned.
- **Records tag contained in the records branch on origin is not listed (weft-branch comparison):** none. `pushTip` only runs for the code side, so a records tag compared against the wrong branch would still pass. The records row (`archiveRecords: true`, no `pushTip`) is listed either way.
- **A `archive/` teardown or `--drop-local` tag is not listed:** none. This is an explicit acceptance criterion and D12 sentence, and no row creates an `archive/<slug>/…` tag on origin.
- **Non-prime slug resolved through `PairTaskBranchOrOrigin`:** none. Only the prime slug is exercised.
- **Branch absent on origin (`branchTip == ""`) lists the tag:** none.
- **No origin remote returns nothing; an unreachable origin is an error:** none.
- **`ownerArchiveSlug` shapes (nested slug, malformed names) and the sort by slug, side and tag:** none. Only one tag is ever returned.

**fabricengine `remoteTagsUnder` refactor:** `remoteArchiveTags` behaviour is unchanged, and the existing remote-leftover tests cover it indirectly.

**Shortcut `s`:** the `s` row's help text is asserted in `shortcut_test.go` and `cmd/lym/shortcuts_test.go`. The stop-at-first-non-zero behaviour is the shortcut engine's own, which this diff does not change. I did not check that an existing shortcut test pins it, so criterion "a stopped adopt runs no `beam start`" has no row in this diff.

## Findings

1. **MEDIUM, design:** no test pins that teardown and `--drop-local` `archive/` tags are excluded, which D12 and the sixth acceptance criterion require. `internal/fabricengine/ownerarchive_integration_test.go:158` (`TestReadOwnerArchives`) has no row. Add a row that pushes an `archive/<slug>/<sha>` tag and asserts it is not listed.
2. **MEDIUM, design:** the weft-branch comparison for records tags is not pinned. `internal/fabricengine/ownerarchive_integration_test.go:168` only sets `pushTip` for code tags, so judging a records tag against the code branch passes. Add a `archiveRecords` + `pushTip` row that expects nothing listed.
3. **MEDIUM, design:** the `--here` wiring in `cli.go` is untested, namely `resolveCall`/`deps.Worktree`, the `--here`/`--archives` flags, `hubCallerDeps.PullBolt` and `hubArchiveDeps`. Evidence: `internal/adoptcli/cli.go` `RunE`; `caller_test.go` and `archives_test.go` build deps by hand. A hub-level test of `Command()` (the repo has `hubforge`) would catch a wrong worktree name.
4. **LOW, design:** the merge of the call's flags into a resumed record's request is unpinned. `internal/adoptcli/here.go:53`. Add a resume row with `req.NoOrch` and assert the merged worker args or request.
5. **LOW, design:** `--here` with `root` or `--all`, and `--discard --here`, have no refusal rows. `here.go:54` and `caller.go:253`. Add them as rows beside `HereWithASubjectIsRefused` and the discard refusal row.
6. **LOW, design:** `PullBolt` returning an error is untested. `here.go:582`. Add a row where `PullBolt` fails and assert the refusal text.
7. **LOW, design:** the `Effective`-through-parent owner path for a beam with no own owner and no record is untested. `here.go:593`. Add a no-record row for the `gamma` beam under a foreign-owned prime.
8. **LOW, design:** `ReadOwnerArchives` rows are missing for a non-prime slug, a branch absent on origin, no origin remote, an unreachable origin, and ordering across several tags. `ownerarchive.go:742` onward.
9. **LOW, design:** `runArchives` rows are missing for duplicate subjects collapsing in `subjects`, and a `ParentOf` failure inside `keep` (`placeErr`). `archives.go:107` and `archives.go:117`.
10. **LOW, design:** `TakesUp` returning false for a not-unfinished (done) record has no row. `internal/adoptrecord/watch.go:656`.

## Tool calls
None; all evidence was already in the inherited context.
