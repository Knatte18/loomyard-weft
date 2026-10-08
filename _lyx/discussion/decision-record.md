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
- A Merriam fixed-context section in `tools/tokencount`'s calibration report, and the calibration's reconstructed Merriam base brought to the texts `MerriamBase` counts.
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
A hex string of 4 to 39 characters is resolved to the one commit it abbreviates: the helper lists every object the prefix names with `git rev-parse --disambiguate=<value>` and keeps those that are commits.
Exactly one commit resolves, and its full SHA replaces the value for everything downstream: the merge-only walk in `reconcileHead`, the digest's `HeadSHA`, `CardSHAs`, the done-check and drift delta end, and `reset --to report-head`'s target.
A prefix that names no commit, or more than one, is refused with a new refusal that names the value and gives the way forward: have the fork rewrite `head_sha` as the output of `git rev-parse HEAD`, then re-run the verb.
A value that is not hex is left as given, so it meets the existing refusal at its site: the mismatch refusal in record-batch and recover-batch, the missing-commit refusal in `reset --to report-head`.
The helper is called at the three consume sites: record-batch, recover-batch (its digest's `HeadSHA`) and `reset --to report-head`.
In record-batch it resolves the parsed report's `head_sha` once, before the value's first use, so the correctness-failure path's failed digest records the resolved SHA too.
A correctness finding wins over an unresolved value: the batch fails on its finding, and the failed digest records an empty head, as it does today for a report that does not parse, never the raw value and never D1's refusal.

**Rationale:** the report named the right commit and the refusal only failed an exact string match.
Resolving once at the consume point keeps every later comparison and record on full SHAs, so the stored run record never holds an abbreviation.
The bound: only object names are matched, never ref names, so `HEAD`, a branch or a hex-named ref is never accepted as a head;
the prefix must name exactly one commit;
and the resolved commit then meets the same checks a full SHA meets at its site, `reconcileHead`'s merge-only walk or reset's descends-from-start check.
Letting the correctness failure win keeps a bad `head_sha` from holding off a batch failure a fork's own transcript already decided.
The 4-character floor is git's minimum abbreviation.

### D2 — a batch's own already-attributed transcripts count toward its attribution and are re-audited

**Decision:** each fork batch's state records, in a field of its own, the transcripts attributed during its current bracket.
The list is cleared whenever begin-batch or recover-batch writes that batch's record, a re-begin of a non-terminal batch included:
every begin-batch launches a new fork, so it opens a new bracket for attribution even where it keeps the batch's `StartSHA`.
record-batch settles as today: `SettleRetry` waits out its window for a transcript new since the run's boundary, judged against the full `SeenForkTranscripts`.
Its fetch re-reads the bracket's own transcripts too: the seen set handed to `AuditForksIncremental` excludes them, so each is audited again in full together with the new ones.
A new transcript counts toward attribution as today;
a bracket's own transcript counts only when its re-read holds a fork write (Write, Edit or NotebookEdit) to the batch's own report path.
Attribution counts the distinct union of the counted transcripts: zero is `ErrNoForkTranscripts`, one attributes, more than one is the existing multi-transcript warning.
`ErrNoForkTranscripts`'s trigger becomes: no counted fork transcript in the batch's current bracket, neither one new since the run's boundary nor one of the bracket's own that wrote the report.
`SeenForkTranscripts` still advances on every call, the no-report call included, so no other batch's record-batch or `auditTerminalFork` can claim a transcript this bracket holds.
A finding already dispositioned is dropped by its identity as today, so re-reading a transcript reports each finding once, and a resumed fork's later writes are audited.

**Rationale:** a stopped fork that is resumed appends to the transcript an earlier no-report record-batch already attributed, so the next call sees zero new transcripts and archives a good report.
Counting the bracket's own transcripts makes record-batch idempotent across a no-report call, and re-auditing them closes the hole where the resumed fork's later writes went unaudited.
Settling against the full seen set keeps the flush-timing window for a fork re-launched inside the open bracket, so its transcript is attributed to this batch and never leaks to the next batch's attribution.
The bound: a carried-over transcript lets through only a report its own fork wrote with a tool the audit sees.
A report Master wrote, by any tool, or one no bracket transcript shows a write to, still meets `ErrNoForkTranscripts`, and so does a bracket whose only transcripts belong to an earlier bracket of the same batch number or to another batch.
The cost is at most one settle window per call on a bracket that already attributed a transcript.

### D3 — a state without an overview frame hash derives the recorded frame from the stored plan baseline

**Decision:** one helper returns the run's recorded overview frame hash: `State.PlanOverviewFrameHash` when set, otherwise the frame hash (`planparser.OverviewWithoutCardIndex`, as `overviewFrameHashOf` hashes it) of the plan-baseline copy that `State.PlanFileHashes["00-overview.md"]` names under `plan-baseline/`.
`overviewIndexOnly` and `overviewRefusal` in `rebaseline.go` and the plan-drift way-forward advice in `beginbatch.go` all use it.
When neither source yields a frame (no hash recorded, the baseline copy absent, or the copy has no parseable Card Index), the run has no recorded frame and the current refusal and advice stand.
A successful rebaseline records `PlanOverviewFrameHash` through `restampBaseline` as today, so the derivation runs only until the first restamp.

**Rationale:** the plan-baseline store holds the recorded overview bytes for every record that hashes its plan, so the recorded frame is computed from them, not trusted from the current file.
This accepts follow-up cards added with their Card Index lines without weakening the guard: a change outside the Card Index still differs from the recorded frame and is refused.
A record with no `PlanFileHashes` already skips the changed-file check, so it is untouched.
The bound: the fallback can only accept an overview whose frame equals the frame of the stored copy the run recorded; it never accepts a frame it cannot compute.

### D4 — tools/tokencount's calibration reports a measured Merriam fixed context

**Decision:** the calibration report gains a Merriam fixed-context section.
Per run, it measures the run's first Merriam session by start time.
A later Merriam session of the run (a resume, a refresh, a re-launch) starts with prior conversation or a compaction summary in context, so it is listed among the section's skips with that reason.
The measurement point is the first counted message whose context holds the tool result of Merriam's Read of its prompt file, the file its launch line names.
lyx types the role skills' load turn before the launch line, so that context holds the system prompt, tools, CLAUDE.md files, the loaded skill bodies, the launch line and the rendered Master stencil with its PATTERN directive, and not yet the plan's `00-overview.md`, which the stencil has Merriam read afterwards.
The run's derived fixed context is the start context there (input, cache write and cache read, read as a fork's start context is) minus the line count of exactly the texts `websterengine.MerriamBase` counts but `00-overview.md`, times the profile's `context_per_line`.
Those texts are reconstructed at the run's base commit: `CLAUDE.md`, the Master template, and the orchestrator PATTERN directive rendered from that commit's `PATTERN.md` and its `contracts/stencils/pattern/pattern-directive-orchestrator.md`, filled as `pattern.Directive` fills it, never the raw `PATTERN.md`.
`readRunStartBase` changes to reconstruct the same directive in place of the raw `PATTERN.md`, keeping `00-overview.md`, so the start fit prices the base `MerriamBase` computes and both sections reconstruct one set of texts.
`CLAUDE.local.md`, which `MerriamBase` also counts, is untracked and cannot be reconstructed at a base commit: the derivation assumes the run's worktree had none, as a lyx-created worktree has none.
It lists each run's measured start, line count and derived fixed context, their median and spread, and the current `merriamFixedContext` beside them.
The constant stays a Go constant; the operator updates it, and its comment, from the report.
The comment on `merriamFixedContext` names the calibration section as its source and says the figure covers the system prompt, tools and the skills loaded at launch.
The current value is read through `websterengine.MerriamBaseOf`'s returned `Fixed`, so the tool needs no new export.

**Rationale:** the constant's own derivation is "measured session start minus the computed texts at that commit".
The measurement point is the first one whose context holds every computed text but the overview, so the subtraction removes only lines present there, and the skill bodies every launch pays land in the fixed figure.
Subtracting the rendered directive rather than the raw `PATTERN.md` keeps the directive wrapper's lines out of the fixed figure, so `MerriamBase` never counts them twice.
The bound on `CLAUDE.local.md`: a run whose worktree had one overstates its own derived figure by that file's lines times `context_per_line`, and the median over runs absorbs a minority of such runs.
The report reproduces that derivation over every recorded run instead of one.

### D5 — refusal rows and docs land with the code

**Decision:** `contracts/specs/refusal-spec.md` gains a row for D1's unresolvable abbreviation.
The `report unattributable` row is reworded to D2's `ErrNoForkTranscripts` trigger, and the `ErrNoForkTranscripts` text in `audit.go` is reworded to the same trigger.
The `rebaseline overview changed` row is reworded to D3's: an overview change outside the Card Index, judged against the recorded or derived frame.
`internal/websterengine/doc.go` states that `head_sha` may be an abbreviation resolved to the full SHA, that a bracket's own transcripts that wrote the report count toward attribution, and that the recorded overview frame falls back to the stored baseline copy.
`tools/tokencount`'s package comment in `main.go` names the new section.

**Rationale:** `PATTERN-refusal-way-forward` requires a new refusal's row in the same commit and a test that reaches it; CLAUDE.md requires the module doc in the same commit.

## Constraints

- `PATTERN-refusal-way-forward`: D1's new refusal names its way forward, has its spec row in the same commit, and a test reaches it.
- `PATTERN-no-denied-recovery`: no new or reworded way forward names `git reset --hard`, `git push --force` or `rm -rf`; D1's way forward names `git rev-parse HEAD`, a read-only command, for the fork to run.
- `PATTERN-gitexec-checked-call`: D1's resolution runs `git rev-parse --disambiguate` through the checked `gitexec.Run`, inside `gitwrap.go`, the one webster file that runs git.
  It never goes through go-git's `ResolveRevision` (`gitrepo.Repo.ResolveSHA`), which returns the first match of an ambiguous prefix and also resolves ref names.
- `PATTERN-told-geometry`: no new helper derives a path; the baseline store path comes from the told `WebsterDir`.
- `PATTERN-shuttle-provider-seam`: D2 changes only websterengine's seen set and counting; `claudeengine`'s transcript layout knowledge is untouched.
- `PATTERN-durable-vs-ephemeral-state`: the per-bracket transcript list is a field of the existing `state.json`, under `_lyx`.
- `PATTERN-test-economy` and `PATTERN-test-speed`: each scenario extends an existing table or test where one covers the path, at the lowest tier its substrate needs; git-backed cases follow the existing `*_git_test.go` files.
- `PATTERN-comment-line-breaks`: new Go comments use one sentence per line.

## Auto-mode assumptions

- No operator was available; every design pick was made by the Discussion producer and is logged as an auto-pick in the support log.
- #457's root cause is a fork stopped and later resumed in the same transcript file across a no-report record-batch; no other cause of zero transcripts (a different session id, a cross-machine resume) is designed for here beyond the existing refusal.
- A fork writes its report with the Write or Edit tool; a resumed fork that writes it any other way still meets today's archive and re-drive.
- The plan-baseline store holds the overview copy for every state with `PlanFileHashes`, since both landed in the same commit; D3 falls back to today's refusal when it does not.
- Merriam reads its prompt file in full in its first Read, before it reads the plan overview.

## Open risks

- A resumed fork's transcript is re-read in full on each record-batch of its bracket; finding identity (`findingIdentity`, the violation `Key`) must be stable across re-reads of a grown transcript, or a finding repeats.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was addressed by the round's fixer, and no fresh reviewer has seen the fix.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - d1-head-sha-resolution-point-vs-correctness-path (class decision, severity MEDIUM, open in rounds 2)
  - d4-subtracted-texts-differ-from-merriambase (class design, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- record-batch with a report whose `head_sha` is a 9-character abbreviation of HEAD records the batch, and the digest's `HeadSHA` and `CardSHAs` hold the full 40-character SHA.
- An abbreviated `head_sha` a clean parent merge sits above is accepted with the moved-HEAD warning naming the full SHA.
- An abbreviation that names no commit, and one that names more than one commit, are each refused with D1's way forward, not a reset.
- record-batch on a batch with a correctness finding fails it with the full SHA in the failed digest when `head_sha` is an abbreviation of HEAD, and with an empty digest head, not D1's refusal, when `head_sha` resolves to no commit.
- A non-hex `head_sha` still meets the mismatch refusal in record-batch and the missing-commit refusal in `reset --to report-head`.
- recover-batch and `reset --to report-head` accept an abbreviated `head_sha` the same way.
- record-batch called once with no report (consuming the fork's transcript), then again after the same transcript grew with the fork's write to the report and the report landed, records the batch instead of archiving the report.
- On that second call, a policy finding from the transcript's later part is reported, and a finding the first call dispositioned is not reported again.
- After a no-report call, a report that no bracket transcript shows a write to is still archived as unattributable.
- After a no-report call, a fork re-launched in the open bracket whose transcript appears only after record-batch's first fetch is attributed to this batch within the settle window and is not counted by the next batch's record-batch.
- A re-begin (begin-batch on the batch's non-terminal record, after a no-report call or an archive) clears the bracket's own list, so zero new transcripts there is still `ErrNoForkTranscripts`.
- A no-report call's transcript is not claimed by another batch's record-batch or by `auditTerminalFork`.
- rebaseline on a state with `PlanFileHashes` and a baseline copy but no `PlanOverviewFrameHash` accepts a Card-Index-only overview change with follow-up cards named by `--card`, and records `PlanOverviewFrameHash`.
- The same state refuses an overview change outside the Card Index with the follow-up-card way forward, not the reset-to-start one.
- The plan-drift advice for such a state offers `lyx webster rebaseline` for a Card-Index-only change.
- With the baseline copy absent, rebaseline refuses as today.
- tokencount's calibration over a fixture run with a Webster session whose skill-load turn precedes the prompt Read writes the Merriam fixed-context section with the run's start measured at the message holding the prompt Read's result, its line count, derived fixed figure, the median and the current constant.
The fixture's line count equals the lines of its base commit's `CLAUDE.md`, Master template and orchestrator directive rendered from its `PATTERN.md`, so the directive wrapper's lines are subtracted.
- A second Merriam session of the same run is listed among the section's skips with its reason, not measured.
- `go test ./...` and `go test -tags integration ./...` pass.

## Notes for the plan writer

- `reconcileHead` in `internal/websterengine/gitwrap.go` holds the exact comparison; `ParseReport` in `report.go` only checks non-empty.
- Attribution: `RecordBatch` in `recordbatch.go` (seen set, `SettleRetry`, `ClassifyAttribution` in `audit.go`); begin-batch and recover-batch carry `prior.ForkTranscripts` into a new bracket, so the bracket's own list needs its own field.
  `ForkReport.WritePaths` holds a fork's Write, Edit and NotebookEdit paths; `CheckFork` already compares them against the batch's own report path.
- Overview frame: `overviewIndexOnly` and `overviewRefusal` in `rebaseline.go`, the drift advice near the top of `beginbatch.go`, the store in `planbaseline.go`.
- Calibration: `readRunStartBase` and `Calibrate` in `tools/tokencount/calibrate.go`; session start parsing lives beside the fork start context in `count.go`.
  The skill-load turn is typed by `loadSkillsThenPrompt` in `internal/shuttleengine/run.go`, before the launch line.
