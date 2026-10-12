# Card 10 — pr-align-producer

**Create:**
- `internal/landingshed/pralign_test.go`
- `internal/landingshed#AlignmentDeps`
- `internal/landingshed#AlignmentConfig`
- `internal/landingshed#AlignmentSession`
- `internal/landingshed#AlignmentNote`
- `internal/landingshed#WriteAlignmentNote`
- `internal/landingshed#ReadAlignmentNote`
- `internal/landingshed#prAlignName`
- `internal/landingshed#PRAlign`
- `internal/landingshed#NewPRAlign`
- `internal/landingshed#PRAlign.Call`
- `internal/landingshed#fakeAlignmentSession`
- `internal/landingshed#TestPRAlign`
- `internal/loomcli#landingAlignmentDeps`
- `internal/loomcli#TestLandingAlignmentDeps_Config`

**Edit:**
- `internal/landingshed#Deps`
- `internal/loomcli#landingDeps`
- `internal/testkit/envkit#LandingDeps`
- `internal/loomcli#intentionallyNil`
- `internal/battencli#intentionallyNil`
- `internal/landingshed#PRGate.Call`
- `internal/landingshed#PRGate.awaiting` -> `func (g *PRGate) awaiting(ctx context.Context, reason, notice string) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `internal/landingshed#TestPRGate_Verdicts`
- `internal/landingshed#landingshedAllowedImports`
- `internal/landingshed/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/landingshed#RecordApproval`
- `internal/landingshed#RecordRejection`
- `internal/alignment#Session.Begin`
- `internal/loomcli#alignmentSettings`
- `internal/loomcli#alignmentSnapshot`
- `internal/loomcli#hubBoardTasks`
- `internal/shedrun#AlignmentDir`

**Intent:**
Add D5's `PR-Align` producer, which spawns the alignment agent at PR-Gate and applies its verdict, and make `PR-Gate` name an escalation in its awaiting notice.
It lives in `landingshed` beside `PR-Gate`, since it reads the pull request through `FindPullRequest` and `NewGitHubClient` (`internal/githubclient`, `PATTERN-github-auth`) and writes the records `PR-Gate` reads.
`landingshedAllowedImports` adds `internal/alignment`, which resolves no geometry.

Seams on `Deps`, a new field `Alignment AlignmentDeps` (`deps.go`, its doc in the field's comment):
- `Config func() (AlignmentConfig, error)` is read on each call. `AlignmentConfig` holds `Off` (`skip_alignment` or no board dir), `PRApprove` (`alignment_pr_approve`) and `Settings` (`alignment.Settings`).
- `Session AlignmentSession` is the interface `*alignment.Session` satisfies: `Begin`, `Poll`, `Close`, `MarkApplied` and `Attempts`, with card 6's signatures.
- `Snapshot func() (string, error)` reads the decision snapshot.
- `Prompt alignment.PromptInput` holds the run's static prompt fields.
- `NotePath string` is the alignment note record. Empty means no note, which is how `envkit` and tests that wire no alignment are built; a darn run gets a path but never a note, since its recipe has no `PR-Align` row.
- `Tick time.Duration` is the poll interval, where zero takes one second.

`AlignmentNote` (`alignmentnote.go`) is the record `PR-Align` leaves when it applies no approval or rejection: `PRNumber`, `HeadSHA`, `Report` and `Reason`.
`WriteAlignmentNote` writes it atomically through `writeRecordAtomic`. `ReadAlignmentNote` returns found false when it is absent and an error for an undecodable file.

`NewPRAlign` refuses an empty `ApprovalPath` or `RejectionPath`, a nil `TaskHead`, a nil `Alignment.Config`, `Alignment.Session` or `Alignment.Snapshot`, and an empty `Alignment.NotePath`. These are fail-loud wiring guards, as `NewPRGate`'s are, and `shedrecipe`'s conformance table reaches them through card 11.

`PRAlign.Call`, top to bottom. It ends `Done` on every path except a cancellation or an error a producer returns today, and never `Stuck`:
1. `entryErr`.
2. The parent branch is not in `Config.RequirePRToBase`: `Done`, with no GitHub call and nothing spawned.
3. `Alignment.Config()`: an error is returned. `Off` is `Done` with nothing spawned.
4. The pull request is read as `PR-Gate` reads it. A transient GitHub failure is returned through `transientFailure`. Any other failure, no pull request, or a closed or merged one is `Done`, with a log line, leaving `PR-Gate` to report it.
5. An approval or rejection record that already matches the pull request number and head is `Done`.
6. An attempt from `Attempts()` that is closed with the key `pr-<number>-<head>` is `Done`: that head's verdict was already applied. An open one is reused through `Begin`, which returns it after a restart (D5).
7. The snapshot is read, and `Begin` is called with that key, the prompt filled with `alignment.PointPRGate`, the parent branch, number and head, and the settings, snapshot, number and head.
   A `Snapshot()` error is handled as a `Begin` error is, with nothing spawned.
   A `Begin` error writes a note with the error as its reason and is `Done`, so a spawn failure leaves the review to the run's parent through `PR-Gate`, as every other failure does.
8. The wait polls `Poll` once per `Tick`, at most once a second (`PATTERN-daemon-wakeups`).
   - A `Poll` error closes the attempt, writes a note naming the error and is `Done`.
   - A `Close` error, on this or any other path, is logged, and the note is still written.
   - A cancelled `ctx`, which is how an interrupt or a teardown reaches the row, closes the attempt and returns `cancelErr`.
   - A `Lock.PauseRequested` answer of true closes the attempt, writes a note with the reason "the run was paused while the alignment agent ran", and is `Done`. A goto reaches a running row only this way, since `shedengine.Goto` refuses a running run and names pause as the way forward.
   - A closed attempt's later verdict polls as `CauseClosed` and is never applied.
   - A run moved back to `PR-Align` at the same head after such a pause finds the attempt closed (step 6) and ends `Done`, so that head's review stays with the run's parent through the pause note; a new head after rework or a reroute aligns again.
9. When `Poll` is done, the pull request and `TaskHead` are read again, and the outcome is applied.
   Every `AlignmentNote` records the pull request number and head read at apply time, the re-read values and not the spawn's, so `PR-Gate` finds the note against the pull request it then reads.
   A note written with no re-read, for a `Snapshot`, `Begin` or `Poll` error or a pause, records the values read at step 4.
   - A cause, invalid, timeout, died or missing, writes a note with the detail and the report when the file exists.
   - A pull request number or head that moved since the spawn, or a task head that differs, writes a note saying so, and nothing is applied.
   - `escalate` writes a note with the report and the question.
   - `approve` against a snapshot that changed since the spawn writes a note with the reason "a decision was recorded for this task while the alignment agent ran".
   - `approve` with `PRApprove` false writes a note with the report and the reason "the alignment agent approves; alignment_pr_approve is false, so the approval is the run's parent's to give".
   - `approve` otherwise is applied through `RecordApproval(ApprovalPath, RejectionPath, Approval{number, head, now})`. When the verdict names `superseded` slugs and the attempt is not yet `Notified`, it queues one `Deps.Notify` line naming the slug, the pull request and the board entries to amend. `MarkApplied` then records `approve` and the notify.
   - `reject`, when an earlier attempt's `Applied` is already `reject`, writes a note with the reason "the alignment agent rejected this run once already; a second agent reject goes to the run's parent". An orch rejection is never counted, since it leaves no attempt.
   - `reject` otherwise first has `MarkApplied` record `reject`, and is then applied through `RecordRejection` with the report's content as `Findings`.
     The count comes first so a restart between the two never leaves a written rejection uncounted; a failed `RecordRejection` makes a later agent reject only escalate, the safe direction.
   - Every path then closes the attempt.

`PRAlign` takes no parent lock (`PATTERN-parent-lock`).
Bound, in the doc comment: an agent approve writes the approval record, so `PR-Gate` lands the run and batten's Land row deploys with no orch turn. It is bounded by `alignment_pr_approve`, the snapshot and head checks, and one agent reject per run.

`PR-Gate`:
- `PRGate.awaiting` takes a notice, which it sets as `OutputPointer.ParentNotice`. Every existing call in `PRGate.Call` passes an empty one.
- At the stale-record and no-record exits, when `Alignment.NotePath` is set and `ReadAlignmentNote` finds a note matching the pull request number and head, it starts the reason with a clause, report path first, and passes the same clause as the notice: "alignment report <path> did not settle this pull request - <reason>; " followed by the existing reason. The driver's relay therefore carries the report path to the run's parent, and batten's fallback awaiting notice, which renders the status error cut at `noticeErrorMax` runes (`internal/battenshed/notice.go`), names it too, since the path leads the text the cut keeps.
- An unreadable note is logged and adds nothing.

Tests:
- `TestPRGate_Verdicts`' row struct gains `noticeHas`. New rows: a note matching the pull request and head starts the reason with the clause, report path first, and sets it as the parent notice; a note for another head adds nothing; an empty `NotePath` adds nothing. Existing rows assert an empty notice.
- `TestPRAlign` is one table test over `newPRGateFixture`'s GitHub server and `fakeAlignmentSession`, which records begins, polls, closes and applied marks and returns scripted outcomes; it can also change the server's pull request head from inside `Poll`. Rows:
  - A parent outside `require_pr_to_base` spawns nothing and makes no GitHub call.
  - `Off` spawns nothing.
  - A `Config` closure that errors returns the error and spawns nothing.
  - A 5xx answer from the server returns the transient error and spawns nothing.
  - A 404 or 422 answer from the server, a non-transient failure, ends `Done` and spawns nothing.
  - No pull request ends `Done` and spawns nothing.
  - A closed pull request ends `Done` and spawns nothing.
  - A `Snapshot` closure that errors spawns nothing, writes the note naming the error and ends `Done`.
  - A `Begin` that fails writes no record, writes the note naming the error and ends `Done`.
  - A matching approval spawns nothing.
  - A matching rejection spawns nothing.
  - An open attempt with the key `pr-<number>-<head>` is passed to `Begin`, and no new spawn is counted, which pins the criterion that a driver restart during `PR-Align` reuses the live spawn.
  - A `Poll` that errors writes the note naming the error and ends `Done`.
  - An approve writes an approval with the number and head, removes a pending rejection and leaves no note.
  - An approve with `PRApprove` false writes no record and writes the note.
  - An approve naming superseded slugs queues one notify naming them, and the attempt is marked `Notified`.
  - An approve with none queues no notify.
  - An approve with a changed snapshot writes no record and writes the note.
  - A first reject writes a rejection whose findings are the report, and the fake session records that no rejection file existed when `MarkApplied(reject)` ran.
  - A reject after an applied-reject attempt writes no record and writes the note.
  - An escalate writes the note naming the report.
  - A timeout cause writes the note naming the cause.
  - A head that moved during the wait writes no record, and its note carries the new head, which `TestPRGate_Verdicts`' matching-note row then covers.
  - A `TaskHead` that differs from the pull request head at apply time writes no record and writes the note.
  - A closed attempt for the same key spawns nothing.
  - A cancelled context closes the attempt and returns the cancellation error.
  - A pause request closes the attempt, writes the pause note and ends `Done`.

The field is filled where `Deps` is built, in the same card, so `TestLandingDeps_EveryFieldPopulated` and `envkit`'s nil-seam walk stay green:
- `landingAlignmentDeps`, a new function in `internal/loomcli/landingdeps.go`, builds the struct. `Config` loads loom's config through `loomengine.LoadConfig(fabricengine.ConfigDirs(l), "loom")` on each call. Its `Off` is `alignmentOff` and `Settings` is `alignmentSettings` from card 8. `Session` is `alignment.NewSession` over `shedrun.AlignmentDir(l, shedrun.SelfRunID, "pr-gate")`, the stencils dir and `alignment.RunnerShuttle(runner)`, the run id the status checkpoint closure here already uses. `Snapshot` is `alignmentSnapshot(loomengine.DiscussionDecisionRecord(l), seedSlug(l.WorktreeName), hubBoardTasks(l))`. `Prompt` holds slug, worktree, decision record, support log and `planparser.PlanDir(l.AnchorPath())`. `NotePath` is `note.json` in that pr-gate directory, which `ListAttempts` skips as a file.
- `landingDeps` sets `Alignment: landingAlignmentDeps(l, geom.StencilsDir, registry, runner)`. Its other callers, the darn wiring included, take it unchanged: a darn recipe has no `PR-Align` row, and no note is ever written there, so its `PR-Gate` reads none.
- `envkit.LandingDeps` fills `Alignment` with an off `Config`, a no-op session, a fixed snapshot and a note path under its dir.
- The loom and batten wiring seam tests list `Landing.Alignment.Config`, `Landing.Alignment.Session` and `Landing.Alignment.Snapshot` in their `intentionallyNil` maps, with the reasons their sibling `Landing.*` entries give: loom assembles `Env.Landing` in run.go, and batten lands no branch.

`TestLandingAlignmentDeps_Config` drives the real `Config` closure over a temp location with `loom.yaml` seeded:
- `skip_alignment: true` reads `Off`.
- No board dir reads `Off`.
- With a board dir and the template, it reads not `Off`, with `PRApprove` true and the resolved `opus[high]` settings.
- `alignment_pr_approve: false` reads `PRApprove` false.

This pins the PR-side half of the criterion that `skip_alignment` or no board dir leaves both points as before.

`doc.go` gains `PR-Align` in its producer list and a paragraph on it. `test-budget.yaml` raises `internal/landingshed` by one and `internal/loomcli` by one.

**ImpactSummary:** `Deps` gains one struct field, which `landingDeps` and `envkit` fill, and `PR-Gate` an optional notice, with every existing awaiting call passing an empty one; no recipe runs `PR-Align` until card 11.
