```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests.** Per behaviour, the test that pins it or none.

- **Run-agent session refused.** Pinned by `TestUpsertTaskWithRecord_Refusals/RunAgentSession` and CLI `stepUpsertRecordFile` "a run task session".
- **Refusals for an absent slug, no stored record (flag unset, file missing), done, run-held, worktree and live weft, each with its way forward and nothing written.** Pinned by `TestUpsertTaskWithRecord_Refusals`. Each row checks the "way forward" text, the record file and the entry unchanged.
- **Origin probe failure refused.** Pinned by `Refusals/WeftProbeFails` and by the CLI row "an origin that cannot be probed", which also covers the probe wiring.
- **Record-file format checks.** Pinned by the CLI rows for a missing section, an unnamed decision, a missing file, and `TestCheckRecordSections`. The `## PATTERN changes` finding is pinned there too.
- **Replace in one write with the payload, and stamp the new body.** Pinned by `TestUpsertTaskWithRecord_ReplacesRecordWithPayload` and the CLI valid-record subtest.
- **Refused payload keeps the former record.** Pinned by `RefusedPayloadKeepsFormerRecord`. Its second row reaches the restore path, since the refusal comes after the record write.
- **Stamp on `StoreDecisionRecord`.** Pinned by `TestStoreDecisionRecord` rows `StampsTheDigestOfTheBody` and `RepeatWithSameBytesAfterABodyEditRewritesTheStamp`. The run-held DeepEqual row shows the stamp is not scope-locked.
- **Stamp on Raddle-File's filing (entries, project, parts).** Pinned by `TestFileRaddle`.
- **Payload naming `record_body_sha256` refused.** Pinned by `TestUpsertFieldAllowlist/upsert_record_body_sha256_key_errors`.
- **Promote warns and still promotes, silent for an unchanged body, silent for an unstamped entry.** Pinned by `TestPromoteToRecordWarning` and the CLI `stepPromoteWarnsOnChangedSketchBody`. The CLI step checks the envelope's `warning` key, present and absent.
- **`RecordsBranchLive` for a local branch, an origin-only branch, an archive-covered branch, no branch, `SkipPush`, and an unreadable origin.** Pinned by `TestRecordsBranchLive`. Add's refactor onto the shared probe has no new test; the existing Add tests are its coverage.
- **`Validate` unchanged after delegating to `CheckRecordSections`.** Pinned by the existing `TestValidate_*` tests.
- **Family-project exemption.** Only the exempt case is pinned, by `TestUpsertTaskWithRecord_FamilyProjectSkipsWorktreeAndWeftRefusals`. See F1.
- **Family-project exemption still applies the run lock.** None. See F1.
- **Promote warning limited to a Sketch (`IsSketch` gate).** None. See F3.
- **`recordsBranchLive` refusing under `--board-path` (`location == nil`).** None. See F4.
- **`RecordsBranchLive` with a local branch and an unreadable origin.** None. See F2.

## Findings

**F1 — MEDIUM, class `scope`.** `internal/boardengine/recordupsert_test.go:540` and `internal/boardengine/recordupsert.go:531-533`.
- The exemption condition is `task.Kind == KindProject && held && filed.Project == slug`.
- Only the all-true case is tested.
- No row refuses a project entry the raddle store does not record as filed, with a worktree standing and the probe live. Making any project exempt would pass every current test.
- No row has a run-held family project, which D4 says keeps the run lock.
- Fix: add refusal rows for a kind-`project` entry without a store filing (worktree and live-weft). Add a run-held family project row. Alternatively add a row for a filed part, where `Holds` is true but `Project != slug`.

**F2 — LOW, class `scope`.** `internal/fabricengine/add_remoteleftover_integration_test.go:828`.
- The "local weft branch is live" row has a reachable origin.
- It would also pass if `RecordsBranchLive` probed origin despite a local branch.
- Fix: add a row with a local weft branch and `origin` set to a missing URL, expecting `true` and no error, to pin "local alone decides".

**F3 — LOW, class `scope`.** `internal/boardengine/board.go:224` and `internal/boardengine/sketch_test.go:617`.
- The warning is gated on `IsSketch`, but no row shows a stamped non-Sketch entry whose body changed gets no warning.
- Fix: add a `TestPromoteToRecordWarning` row where the entry is a stamped task or note without the sketch label and the body was edited, expecting an empty `RecordWarning`.

**F4 — LOW, class `scope`.** `internal/boardcli/cli.go:70-75`.
- The `location == nil` refusal ("resolved only from a hub worktree, not under --board-path") has no test.
- The refusal is not covered by any `TestCLI` step.
- Fix: add a `--board-path` upsert `--record-file` row expecting that message and nothing written.

**F5 — LOW, class `consistency`, test economy.** `internal/boardengine/recordupsert_test.go:540`.
- The redundancy report lists `TestUpsertTaskWithRecord_FamilyProjectSkipsWorktreeAndWeftRefusals` as contained, covered by `TestFileRaddle`, `TestUpsertTaskWithRecord_Refusals` and `TestUpsertTaskWithRecord_ReplacesRecordWithPayload`.
- None of those three covers its subject. `TestFileRaddle` never calls `UpsertTaskWithRecord`, and the other two use non-project entries with the probe called or the worktree absent.
- It pins the exemption, with the probe never called and the worktree standing, and it has no `//testtiming:keep` marker.
- Fix: keep it and add `//testtiming:keep` directly above its `func Test…` line, with the reason "pins the family-project exemption from the worktree and live-weft refusals, which no covering test reaches".

Tool calls:
1. `git diff` of the nine test files. The parent context held only the non-test diff.
2. `Read` of the persisted full output of that diff, which the first call truncated.
