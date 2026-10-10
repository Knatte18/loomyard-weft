---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: contracts/specs/refusal-spec.md:260
    summary: The merge_in_progress refusal that lyx loom start reaches through refuseOverUnfinishedMerge has no refusal-spec row, and the only row for that message is scoped to resume
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/loomcli/start.go:196
    summary: Several refusals reachable from start and resume return a bare err.Error() with no way forward and no refusal-spec row, among them the merge-state probe failure and the status read on the live-driver branch
---

### [BLOCKING] Start's merge_in_progress refusal has no spec row

**Location:** `contracts/specs/refusal-spec.md:260`, `internal/loomcli/start.go:215` and `internal/loomcli/start.go:217`

**Issue:** `refuseOverUnfinishedMerge` is shared by `lyx loom start` and `lyx loom resume`; both emit the `merge_in_progress` kind with the message at `start.go:215` or `start.go:217`.
The loom section of `contracts/specs/refusal-spec.md` has exactly one row for it, `resume: merge in progress`, whose trigger reads "`lyx loom resume` would wake or replace a driver ...".
No row covers `lyx loom start` reaching the same refusal, though this branch changed that refusal's behavior for `start` as well (the Publish/Finalize own-leftover exception, documented in `startCmd`'s help text).
`PATTERN-refusal-way-forward` requires every reachable refusal of a bound module to have a row in its section; the neighbouring rows `start/resume: hub config reconcile failed` and `start/resume: parent branch unreadable` show the convention for a refusal both verbs share.
This round's focus names `refuseOverUnfinishedMerge` and asks whether each message has a row; for the merge refusal reached from `start` the answer is no.

**Fix:** Widen the `resume: merge in progress` row into a `start/resume: merge in progress` row whose trigger names both verbs and whose way forward is "resolve the merge as the message names, then re-run the verb".
A test already reaches the `start` side (`TestRunDriverSpawnAndWait_MidMerge_SpawnRefusals`) and the `resume` side (`TestResumeVerb`), so no new test is needed.

### [BLOCKING] Bare-error refusals in the start path name no way forward

**Location:** `internal/loomcli/start.go:196` (merge-state probe failure), `internal/loomcli/start.go:350` (status read on the live-driver, no-marker branch via `runHaltedAtHandBack`), and the sibling sites at `start.go:283` (run-lock probe), `start.go:298` (strand table read), `start.go:306` (retiring-strand removal) and `start.go:336` (park-marker removal)

**Issue:** Each of these refuses with `output.Err(out, err.Error())`: the underlying error alone, with no retry or fix instruction, and no row in the loom section of `contracts/specs/refusal-spec.md`.
`PATTERN-refusal-way-forward` requires every refusal reachable through a bound module's verbs to name its way forward and have a row.
This branch's own fix F3 applied that requirement to `currentProducer`'s refusal and to the parent-branch refusal, which settles that the start path is in scope; these sites are the same defect left unfixed, so it is filed once as a method gap rather than per site.
Two of them bear directly on the focus:

- `refuseOverUnfinishedMerge`'s probe-error branch (`start.go:193-197`) is one of the refusals the focus names.
  Its test row, `probe error names the failure` in `TestRunDriverSpawnAndWait_MidMerge_ProbeErrorRefuses`, pins only the canned error text (`wantIn: boom`), so the missing way forward is not caught.
- `runHaltedAtHandBack` (`start.go:115-128`) reads the same status file `currentProducer` reads, with the same strict decode.
  This branch made `currentProducer` treat a decode failure as "defer to the driver's own read gate" and log it.
  `runHaltedAtHandBack`, reached when a live driver strand has no park marker, still refuses on that decode failure with the raw `state: decode failed: ...` text.
  The two reads of one file disagree about whether an undecodable status file blocks `start`, and the refusal gives the operator nothing to act on.

**Fix:** Give each site a message that names the retry or the file to fix, as `currentProducer`'s refusal now does, and cover them with one table-driven `start/resume: raw I/O` (or per-cause) transient row in the loom section; extend `TestRunDriverSpawnAndWait_MidMerge_ProbeErrorRefuses` so the probe-error row asserts the way forward.
For `runHaltedAtHandBack`, decide whether an undecodable file is a refusal there at all: if it is, name the file to fix; if the deferral in `currentProducer` is the intended rule, apply it here too and add a row to the live-strand test.

## Publish failure check

The two failing tests, `TestSmokeBootstrapLifecycle/died_driver_proceeds_to_the_success_envelope_and_logs_why` and `.../malformed_status_proceeds_to_the_success_envelope_and_logs_why`, failed at the merge-in commit because `currentProducer` (new in the merge-in's range) read the status file strictly and refused `lyx loom start` with `state: decode failed ...` before the driver spawn; the log at `.lyx/verify/publish-failure.log` lines 81-93 shows exactly that refusal.
At HEAD `currentProducer` returns `""` and logs a warning when `errors.Is(err, state.ErrDecode)`; both the unknown-field and the truncated-JSON shape reach that branch, since `state.ReadJSONStrict` wraps every decode failure in `ErrDecode` (`internal/state/state.go:99`, `:165`).
`TestRunDriverSpawnAndWait_MidMerge_CleanPairProceeds` and the `own leftover under an undecodable status file` row of the spawn-refusal test pin the unit-level behavior, and a run of `TestRunDriverSpawnAndWait_MidMerge` and `TestResumeVerb` in `internal/loomcli` passed.
`bootstrapUnlessSeeded` makes the two rigged steps independent of the earlier steps when selected alone.
I did not raise the two tests as findings: by reading they pass at HEAD.
I could not confirm that by running them, because the smoke suite carries the `tmux` build tag and tmux-tier runs are refused in this session, so the real binary path is confirmed only by the operator's next `publish_verify`.

## Checks with no finding

- Per-card mechanical check: the diff's `Rename` and `Delete` groups were not in this round's focus; no gap seen in `start.go`, `resume.go` or their tests.
- Test economy: the added rows (`own leftover under an undecodable status file`, `clean pair with a status file carrying an unknown field`, `unreadable status file names the failure and the retry`, the parent-branch way-forward row) each pin a behavior no other row reaches.
- Comment conventions: the changed doc comments in `start.go` are self-contained and follow the surrounding style.
