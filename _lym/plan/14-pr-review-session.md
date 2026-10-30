# Card 14 — pr-review-session

**Create:**
- `plan:internal/landingshed#prReviewerRole` -> `const prReviewerRole = "pr-reviewer"`
- `plan:internal/landingshed#prSessions` -> `type prSessions interface`
- `plan:internal/landingshed#PRReview.ensureSession` -> `func (p *PRReview) ensureSession(rec pullrequest.Record) (pullrequest.Record, error)`
- `plan:internal/landingshed#fakePRSessions` -> `type fakePRSessions struct`
- `plan:internal/landingshed#TestPRReview_Session` -> `func TestPRReview_Session(t *testing.T)`
- `plan:internal/runproc#RoleSession` -> `const RoleSession = "session"`

**Edit:**
- `plan:internal/landingshed#PullRequestDeps`
- `internal/landingshed#PRReview.Call`
- `plan:internal/landingshed#PRReview.wait`
- `plan:internal/landingshed#TestPRReview_Wait`
- `plan:internal/landingshed#prReviewTestDeps`
- `internal/landingshed/doc.go`
- `internal/landingshed#landingshedAllowedImports`
- `internal/landingshed#TestToldGeometryInvariant_AllowlistOnly`
- `internal/loomshed#InterruptPolicies`
- `internal/loomcli#landingDeps`
- `internal/loomcli#loomCLI.wireLanding`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/testkit/envkit#LandingDeps`
- `internal/battencli#intentionallyNil`
- `internal/loomcli#intentionallyNil`
- `internal/runproc/doc.go`
- `README.md`

**Uses:**
- `internal/shuttleengine#Spec`
- `plan:internal/shuttleengine#Runner.StartSession`
- `internal/shuttleengine#Runner.StopStrand`
- `internal/shuttleengine#OpenResult`
- `plan:contracts/stencils#LandingTemplatePRReview`
- `internal/runproc#ChildRecord`
- `internal/runproc#Register`
- `internal/testkit/stencilkit#SeedInto`
- `plan:internal/landingshed#PRReview.holdWhileCovered`
- `internal/loomcli#driverWaitMark`
- `plan:internal/pullrequest#UpdateRecord`
- `internal/loomshed/doc.go`
- `internal/loomcli/doc.go`

**Intent:**
D7: one interactive session per run, in its own window in the task worktree, which the operator talks to; the row never waits on its output.
`PullRequestDeps` gains `Sessions prSessions` (`StartSession(shuttleengine.Spec) (shuttleengine.OpenResult, error)` and `StopStrand(guid string) error`, which `*shuttleengine.Runner` satisfies), `SessionLive func(guid string) (bool, error)`, `RegisterSession func(guid string) error` and `BoardEntryPath string`.
Card 11's fail-loud wiring guard covers `Sessions`, `SessionLive` and `RegisterSession` too, and `prReviewTestDeps` fills them with `fakePRSessions`, so every `TestPRReview_Call` and `TestPRReview_Wait` row runs with them.

`ensureSession`, called by `Call` after the record is written and before `wait`, so after card 13's `holdWhileCovered`, never under a pause, and never for a `ClosedForContinue` record card 11's step 6 keeps, whose review the operator already ended: a recorded strand that `SessionLive` reads live is reused; a recorded session id whose strand is not live is resumed by `StartSession` with `ResumeSessionID` set; with none recorded a new session starts.
The spec: the prompt is `landing-template-pr-review` read at call time through `stencilstore.Read(Deps.StencilsDir, ...)` and filled with the parent directive `internal/parentdirective` renders from `Deps.ParentName` and the edit directive `internal/editdirective` renders, as `internal/mergeresolve`'s spec does, and with paths only: the pull request URL, `WorktreeRoot`, `TaskBranch`, `ParentBranch`, `DescriptionPath`, and `run_material` listing whichever of `DecisionRecordPath`, `PlanDir` and `BoardEntryPath` is non-empty and exists on disk;
`Role: prReviewerRole`, `Segment: segmentcolor.Landing`, `Interactive: true`, `Unawaited: true`, `DenyPublish: true`, `Skills` `languageconfig.AppendSkills([]string{"scribe:prose", "scribe:conversation"}, Deps.LanguageSkills)`, as `internal/mergeresolve/spec.go` builds its list, so a language skill repeating one of the session's own loads once, and the model choice `shuttleengine.NewModelChoice` over `Deps.Registry.Resolve` of `Config.PRReviewer`.
After a start or resume it calls `RegisterSession` with the strand GUID, logging a failure at Warn, since the record only lets a teardown end the pane process, and writes the strand GUID, name, session id and engine into the record through `pullrequest.UpdateRecord`, mutating only those fields, so a `Continue` the agent wrote meanwhile survives.
A `SessionLive` error is an error stop of the row.
A failed start is returned through `transientFailure` when shedtransient marks it (an `ErrNotStarted` start), and is otherwise Stuck with Halt naming the start error and `lym loom resume`.
The session is stopped through `Sessions.StopStrand` before each verdict read of `wait` that ends or judges an approval or a merge (merged: stop, then re-read; approved: stop, re-read, and resume through `ensureSession` only when it stopped a live session for this read and the read keeps waiting), on the continue flag before its re-read, resuming through `ensureSession` when it stopped a live session for that read and the read keeps waiting on a dirty tree or an ungated head, open or closed, so the agent can still clean the tree; at the first covered wake of a pause, and on every Done, Stuck and non-cancel error; the first uncovered wake after a pause during which it stopped a live session resumes it through `ensureSession`; a cancelled context leaves it running, so the re-call after an interrupt finds it live.
The wait's wakes start no session, so an operator who closes the window ends the agent until the next entry of the row.
Bound: the denies of `git push` and `gh` are a guardrail over static text (card 6), so a command that slips past them can push, approve, merge or close; a head it pushes lands through lym only after the row gates it, a merge of an ungated head halts after the merge (card 13), a merge of the gated head lands through Finalize's already-landed path, and a close is the closed halt.

`TestPRReview_Session` over `fakePRSessions` and `fakePullRequestAPI`, with the stencils seeded into `Deps.StencilsDir` through `stencilkit.SeedInto`: a first entry starts one session with the stencil's markers filled with paths and no file content, `Interactive`, `Unawaited` and `DenyPublish` set and the `pr_reviewer` model, registers it and records its GUID, session id and engine; `run_material` lists the decision record and the board entry snapshot that exist and leaves out a plan directory that does not; a re-entry with a live recorded strand starts none; a re-entry with a kept `ClosedForContinue` record starts none; one with a recorded but dead strand resumes by its session id; a start failing with `ErrNotStarted` is a returned error and one failing otherwise halts naming `lym loom resume`; a `SessionLive` error is an error; a failing `RegisterSession` is logged and the record still written; a cancelled context leaves the session unstopped; Done, a Stuck halt and an error each stop it, and a covered wake stops it and the next uncovered wake resumes it.
`TestPRReview_Wait` gains the stops before the merged and approved re-reads and the resume after an approved read that keeps waiting, and after a continue read that keeps waiting on a dirty tree, for an open and for a closed pull request.
Both `intentionallyNil` maps, battencli's and loomcli's, gain an entry for each `Landing.PullRequest.*` seam this card adds that `wire()` leaves nil, with the reason the sibling `Landing.*` entries of that map give.
`InterruptPolicies`' `NamePRReview` comment says the row reattaches the live PR session or resumes it rather than spawning a second.
`runproc` gains `RoleSession`, the role of an interactive agent session's pane process; its doc's role list names it.
loomcli: `landingDeps` sets `Sessions` to the runner and `BoardEntryPath` to the run's board entry snapshot; `wireLanding` sets `SessionLive` and `RegisterSession` from reed's strand table (a strand live and not retiring; the pane's `PanePID` registered with `runproc.Register(fabricengine.RunProcDir(l.HubPath), seedSlug(l.WorktreeName), runproc.ChildRecord(pid, runproc.RoleSession, nil))`), as it sets the wait-mark and conflict-stopper seams, and `TestLandingDeps_EveryFieldPopulated` names them in its `unfilled` map with that reason; envkit stubs the new seams.
`landingshedAllowedImports` gains `github.com/Knatte18/loomyard/internal/parentdirective` and `github.com/Knatte18/loomyard/internal/editdirective`, which `ensureSession` imports to render the two directives, and `github.com/Knatte18/loomyard/internal/stencil`, which it imports for the `stencil.Fill` call that fills the `landing-template-pr-review` markers, and `github.com/Knatte18/loomyard/internal/languageconfig`, which it imports for `AppendSkills`, both as `internal/mergeresolve/spec.go` does, so `TestToldGeometryInvariant_AllowlistOnly` passes at this card.
The landingshed doc's PR-Review paragraph describes the session. `README.md`'s Landing bullet's PR-Review sentence, as card 11 rewrote it, adds that the operator discusses the pull request with an interactive agent session in the task worktree.

**ImpactSummary:** Adds the D7 session to PR-Review over shuttle's unawaited start; the wiring changes sit in loomcli's landing seams, and the only other producer touched is the interrupt-policy comment.

**Commit:** 14: PR-Review starts and keeps one interactive session per run
