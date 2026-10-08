# Webster record-batch and rebaseline: no needless re-drive, a calibrated Merriam base

## Goal

Webster's record-batch, recover-batch, reset and rebaseline in `internal/websterengine` never force a needless re-drive or a dead end on four observed paths:
an abbreviated `head_sha` that names HEAD is accepted (#456),
a stopped-then-resumed fork's report is attributed to its batch (#457),
a Card-Index-only change to `00-overview.md` is accepted on a run record without an overview frame hash (#463, rebaseline half),
and the Merriam fixed context (`merriamFixedContext`) can be re-measured from real runs in `tools/tokencount`.

## Scope

In:

- Resolving a report's `head_sha` to a full commit SHA wherever webster consumes it: record-batch, recover-batch and `reset --to report-head`.
- Transcript attribution in record-batch for a batch whose current bracket already attributed a transcript on an earlier call.
- The recorded overview frame for a state without `PlanOverviewFrameHash`, used by `rebaseline` and by the plan-drift way-forward advice.
- A Merriam fixed-context section in `tools/tokencount`'s calibration report.
- The refusal-spec rows, `doc.go` text and tests these changes touch.

Out:

- The plan gate's done-card scoping, the first half of #463, which landed in e003ef6d9.
- A route from Plan-Bouncer back to Webster (`goto` reachability), named in #463 but not asked for in this task's goal.
- Moving `merriamFixedContext` into `batcher.yaml` or any config.
- Changing the fork's report-writing stencil (`webster-body-implementer.md`), which already says `git rev-parse HEAD`.
- Recovering a report that a pre-fix binary already archived as unattributable.

## Decisions

### D1 — head_sha is resolved to the full commit SHA once, where the report is consumed

**Decision:** one websterengine helper resolves a report's `head_sha` before any comparison.
A 40-hex value passes unchanged.
A hex string of 4 to 39 characters is resolved through git to the one commit it abbreviates; the resolved full SHA replaces it for everything downstream: the merge-only walk in `reconcileHead`, the digest's `HeadSHA`, `CardSHAs`, the done-check and drift delta end, and `reset --to report-head`'s target.
A value that is not hex is left as given, so it meets the existing mismatch refusal.
An abbreviation that resolves to no commit, or to more than one, is refused with a new refusal that names the value and gives the way forward: have the fork rewrite `head_sha` as the output of `git rev-parse HEAD`, then re-run the verb.
The helper is called at the three consume sites: record-batch, recover-batch (its digest's `HeadSHA`) and `reset --to report-head`.

**Rationale:** the report named the right commit and the refusal only failed an exact string match, then proposed a reset to the commit HEAD already was.
Resolving once at the consume point keeps every later comparison and record on full SHAs, so the stored run record never holds an abbreviation.
Hex-only resolution means a ref name such as `HEAD` or a branch is never accepted as a head; git's own ambiguity check bounds the abbreviation, and the 4-character floor is git's minimum.

### D2 — a batch's own already-attributed transcripts count toward its attribution and are re-audited

**Decision:** each fork batch's state records the transcripts attributed during its current bracket, cleared when begin-batch or recover-batch opens a new bracket for that batch number.
record-batch audits the bracket's own transcripts again together with the transcripts new since the run's boundary: the seen set handed to `AuditForksIncremental` and to `SettleRetry` excludes the current bracket's own transcripts.
Attribution counts the distinct union of the two: zero is `ErrNoForkTranscripts` as today, one attributes, more than one is the existing multi-transcript warning.
`SeenForkTranscripts` still advances on every call, the no-report call included, so no other batch's record-batch or `auditTerminalFork` can claim a transcript this bracket holds.
A finding already dispositioned is dropped by its identity as today, so re-reading a transcript reports each finding once, and a resumed fork's later writes are audited.

**Rationale:** root cause, from the review-loop-convergence Master transcript (session 83342516): the batch 01 fork was stopped by the user, Master called record-batch, which returned `no_report` and advanced attribution past the fork's transcript;
the fork was then resumed, which appends to the same transcript file, finished, and wrote its report;
the next record-batch saw zero new transcripts, archived a good report and named a full re-drive.
Counting the bracket's own transcripts makes record-batch idempotent across a no-report call, and re-auditing them closes the hole where a resumed fork's later writes were never audited.
The bound: only transcripts attributed to this batch's current bracket are re-read and counted; transcripts of earlier brackets of the same batch number, and of other batches, are not.

### D3 — a state without an overview frame hash derives the recorded frame from the stored plan baseline

**Decision:** one helper returns the run's recorded overview frame hash: `State.PlanOverviewFrameHash` when set, otherwise the frame hash (`planparser.OverviewWithoutCardIndex`, as `overviewFrameHashOf` hashes it) of the plan-baseline copy that `State.PlanFileHashes["00-overview.md"]` names under `plan-baseline/`.
`overviewIndexOnly` and `overviewRefusal` in `rebaseline.go` and the plan-drift way-forward advice in `beginbatch.go` all use it.
When neither source yields a frame (no hash recorded, the baseline copy absent, or the copy has no parseable Card Index), the run has no recorded frame and the current refusal and advice stand.
A successful rebaseline records `PlanOverviewFrameHash` through `restampBaseline` as today, so the derivation runs only until the first restamp.

**Rationale:** `PlanFileHashes` and the content-addressed plan baseline store landed in 9ae5f9405, before overview-frame tracking in 1a8a6d0e3, so every record that hashes its plan holds the exact recorded overview bytes; the recorded frame is computed from them, not trusted from the current file.
This accepts the shuttle-send case (cards 11 and 12 added with Card Index lines) without weakening the guard: a change outside the Card Index still differs from the recorded frame and is refused.
A record with no `PlanFileHashes` already skips the changed-file check, so it is untouched.
The bound: the fallback can only accept an overview whose frame equals the frame of the stored copy the run recorded; it never accepts a frame it cannot compute.

### D4 — tools/tokencount's calibration reports a measured Merriam fixed context

**Decision:** the calibration report gains a Merriam fixed-context section.
Per run with a Webster session, it takes the Merriam session's start context (the input, cache write and cache read of its first counted message, as a fork's start context is read) and subtracts the line count of the texts present at that message, reconstructed at the run's base commit as `readRunStartBase` does but without the plan's `00-overview.md`, times the profile's `context_per_line`.
It lists each run's measured start, line count and derived fixed context, their median and spread, and the current `merriamFixedContext` beside them.
The constant stays a Go constant; the operator updates it, and its comment, from the report.
The comment on `merriamFixedContext` names the calibration section as its source.
The current value is read through `websterengine.MerriamBaseOf`'s returned `Fixed`, so the tool needs no new export.

**Rationale:** the constant's own derivation is "measured session start minus the computed texts at that commit", and the overview is read by Merriam after the first message, so it is excluded the same way; the report reproduces that derivation over every recorded run instead of one.
Keeping the figure in Go avoids a `batcher.yaml` key, which webster-dead-ends just retired keys from and which would bring reconcile churn to every hub.

### D5 — refusal rows and docs land with the code

**Decision:** `contracts/specs/refusal-spec.md` gains a row for D1's unresolvable abbreviation, and the `report unattributable` row (zero fork transcripts) and the `rebaseline overview changed` row are reworded to their new triggers.
`internal/websterengine/doc.go` states that `head_sha` may be an abbreviation resolved to the full SHA, that a bracket's own transcripts count toward attribution, and that the recorded overview frame falls back to the stored baseline copy.
`tools/tokencount`'s package comment in `main.go` names the new section.

**Rationale:** `PATTERN-refusal-way-forward` requires a new refusal's row in the same commit and a test that reaches it; CLAUDE.md requires the module doc in the same commit.

## Constraints

- `PATTERN-refusal-way-forward`: D1's new refusal names its way forward, has its spec row in the same commit, and a test reaches it.
- `PATTERN-no-denied-recovery`: no new or reworded way forward names `git reset --hard`, `git push --force` or `rm -rf`; D1's way forward names `git rev-parse HEAD`, a read-only command, for the fork to run.
- `PATTERN-gitrepo-client-boundary` and `PATTERN-gitexec-checked-call`: D1's resolution goes through `gitrepo` (a local read) or the checked `gitexec.Run`, inside `gitwrap.go`, the one webster file that runs git.
- `PATTERN-told-geometry`: no new helper derives a path; the baseline store path comes from the told `WebsterDir`.
- `PATTERN-shuttle-provider-seam`: D2 changes only websterengine's seen set and counting; `claudeengine`'s transcript layout knowledge is untouched.
- `PATTERN-durable-vs-ephemeral-state`: the per-bracket transcript list is a field of the existing `state.json`, under `_lyx`.
- `PATTERN-test-economy` and `PATTERN-test-speed`: each scenario extends an existing table or test where one covers the path, at the lowest tier its substrate needs; git-backed cases follow the existing `*_git_test.go` files.
- `PATTERN-comment-line-breaks`: new Go comments use one sentence per line.

## Auto-mode assumptions

- No operator was available; every design pick was made by the Discussion producer and is logged as an auto-pick in the support log.
- #457's root cause is the stop-resume sequence read from the review-loop-convergence Master transcript; no other cause of zero transcripts (a different session id, a cross-machine resume) is designed for here beyond the existing refusal.
- The plan-baseline store holds the overview copy for every state with `PlanFileHashes`, since both landed in the same commit; D3 falls back to today's refusal when it does not.
- The Merriam session's first counted message holds the system prompt, tools, CLAUDE.md files and the rendered Master stencil with its PATTERN directive, and not the plan overview.

## Open risks

- A resumed fork's transcript is re-read in full on each record-batch of its bracket; finding identity (`findingIdentity`, the violation `Key`) must be stable across re-reads of a grown transcript, or a finding repeats.
- Skill-load turns typed at Merriam's launch may sit in or before its first counted message; if they do, D4's derived figure includes them, which matches how the original 20600 was measured only if that session loaded the same skills.

## Acceptance criteria

- record-batch with a report whose `head_sha` is a 9-character abbreviation of HEAD records the batch, and the digest's `HeadSHA` and `CardSHAs` hold the full 40-character SHA.
- An abbreviated `head_sha` a clean parent merge sits above is accepted with the moved-HEAD warning naming the full SHA.
- An abbreviation that names no commit is refused with D1's way forward, not a reset; a non-hex `head_sha` still meets the mismatch refusal.
- recover-batch and `reset --to report-head` accept an abbreviated `head_sha` the same way.
- record-batch called once with no report (consuming the fork's transcript), then again after the same transcript grew and the report landed, records the batch instead of archiving the report.
- On that second call, a policy finding from the transcript's later part is reported, and a finding the first call dispositioned is not reported again.
- A re-driven bracket (begin-batch after an archive) does not count the earlier bracket's transcript, so zero new transcripts there is still `ErrNoForkTranscripts`.
- A no-report call's transcript is not claimed by another batch's record-batch or by `auditTerminalFork`.
- rebaseline on a state with `PlanFileHashes` and a baseline copy but no `PlanOverviewFrameHash` accepts a Card-Index-only overview change with follow-up cards named by `--card`, and records `PlanOverviewFrameHash`.
- The same state refuses an overview change outside the Card Index with the follow-up-card way forward, not the reset-to-start one.
- The plan-drift advice for such a state offers `lyx webster rebaseline` for a Card-Index-only change.
- With the baseline copy absent, rebaseline refuses as today.
- tokencount's calibration over a fixture run with a Webster session writes the Merriam fixed-context section with the run's measured start, line count, derived fixed figure, the median and the current constant.
- `go test ./...` and `go test -tags integration ./...` pass.

## Notes for the plan writer

- `reconcileHead` in `internal/websterengine/gitwrap.go` holds the exact comparison; `ParseReport` in `report.go` only checks non-empty.
- Attribution: `RecordBatch` in `recordbatch.go` (seen set, `SettleRetry`, `ClassifyAttribution` in `audit.go`); begin-batch and recover-batch carry `prior.ForkTranscripts` into a new bracket, so the bracket's own list needs its own field.
- Overview frame: `overviewIndexOnly` and `overviewRefusal` in `rebaseline.go`, the drift advice near the top of `beginbatch.go`, the store in `planbaseline.go`.
- Calibration: `readRunStartBase` and `Calibrate` in `tools/tokencount/calibrate.go`; session start parsing lives beside the fork start context in `count.go`.
