# Card 7 — drift-verbs-cli

**Create:**
- `internal/driftengine#VerbStatus`
- `internal/driftengine#RecordScope`
- `internal/driftengine#Accept`
- `internal/driftengine#Withdraw`
- `internal/driftengine#Defer`
- `internal/driftengine#Waive`
- `internal/driftengine#CheckVerifyRow`
- `internal/driftengine#TestRecordScope`
- `internal/driftengine#TestAcceptWithdraw`
- `internal/driftengine#TestDeferWaive`
- `internal/driftcli/doc.go`
- `internal/driftcli/testmain_test.go`
- `internal/driftcli#Command`
- `internal/driftcli#RunCLI`
- `internal/driftcli#RunCLIIn`
- `internal/driftcli_test#TestCapture_OutlivesCaller`

**Edit:**
- `cmd/lyx#newRoot`

**Extends:**
- `internal/driftengine/doc.go`
- `docs/overview.md`

**Uses:**
- `internal/driftengine#UpdateScope`
- `internal/driftengine#CountedCapture`
- `internal/driftengine#TakeRun`
- `internal/driftengine#CheckMainCapture`
- `internal/driftengine#StartCapture`
- `internal/driftengine#RunCapture`
- `internal/driftengine#RunSideCapture`
- `internal/driftengine#RunCompare`
- `internal/driftengine#ProbeLock`
- `internal/testkit/lyxbin#Build`
- `internal/hubforge#NewHub`
- `internal/hibernatecli#RunCLIIn`
- `internal/runproc#End`
- `internal/driftengine#ErrRunsSpent`
- `internal/driftengine#AgentSessionOpen`
- `internal/driftengine#CaptureJobName`
- `internal/proc#HoldNamedJob`

**Intent:**
Add the `lyx drift` verbs (Scope, D3–D7): their rules as `driftengine` functions over the run's status, and the cobra subtree that wires them.

`VerbStatus` is what a verb judges: the run status's current row, state and error (`shedengine.Status`), whether the recipe has `Drift-Await` (loom true, darn false), whether the capture lock is held, and `AgentSession`, whether the agent-session mark reads open (`AgentSessionOpen`).
The agent session is a `running` status at `Drift-Verify` with `AgentSession` true (Shared Decision "run status decides the caller").
A verb whose current row it does not admit refuses naming the row it runs at and `lyx loom status`.

`RecordScope` (D3) refuses unless the current row is `Drift-Baseline`, once a scope record exists (`drift: the scope is already recorded as <scope>; way forward: write baseline.md alone and end your turn`), `--skip` with any scope but `partial`, and `partial` with no `--skip`; otherwise it writes the record with `recorded_at`.

`Accept` (D7) refuses unless the current row is `Drift-Verify`.
A caller is the row's live agent session when the state is `running` with `AgentSession` true, and a caller at a halt when it is `blocked`; a `running` status with `AgentSession` false is the row's own compare or pre-spawn checks, and refuses naming the wait for the row's halt (`lyx loom status` shows it) and the parent's ruling there; any other state refuses naming `lyx loom resume`, so no acceptance settles an error or transient stop (D6).
A decision halt is a `blocked` status whose error opens with `shedadapters.DecisionHaltPrefix`.
The agent session's accept of a withdrawn test refuses naming the parent's ruling (`the run's parent withdrew <test>; way forward: fix its drift or report it in verify-report.md; only the parent re-accepts it at a decision halt`).
A caller's accept refuses a withdrawn test, and under scope `full` any test, unless the halt is a decision halt, naming the parent's ruling at a decision halt as the way forward.
An admitted accept appends or replaces the test's `Acceptance` with the reason, `by` (`agent` or `caller`) and, under `full`, ruling `unruled` for the agent session and `ruled` for a caller; a caller's accept of a withdrawn test takes it off `withdrawn`.
What this lets through, and its bound: under `partial` an accept may skip any test the compare runs, bounded by the agent's judgement and recorded with its reason in the committed scope record; under `full` the agent's acceptances stay `unruled` until the parent rules at a decision halt (card 11-drift-verify-producer).

`Withdraw` (D7) refuses unless the current row is `Drift-Verify`, and refuses a test not on `accepted` naming `lyx drift status`; it moves the entry to `withdrawn`.

`Defer` (D5) refuses unless the status is `blocked` at `Drift-Await`, or `blocked` at `Drift-Baseline` with an error opening with `CaptureHaltPrefix`, naming the halt's own ways forward otherwise; it refuses while the capture lock is held, naming `lyx loom resume`, whose row waits on the running capture.
It sets `awaited_at` with `deferred: true`.
What it lets through: code may change with no baseline taken; bound: `Drift-Verify` then halts on no counted capture unless a passed capture exists, and only the parent waives there.

`Waive` (D6) refuses unless the status is `blocked` at `Drift-Verify` and no counted capture exists (`CountedCapture` over the records and `VerbStatus`'s recipe flag), naming `lyx loom resume` and the halt's own ways forward at any other `Drift-Verify` halt; it writes `waived_reason` and `waived_at`.
What it lets through: the whole regression question for the run; bound: it is admitted only at that halt, a decision halt under every scope, so only the parent waives (card 11-drift-verify-producer).

`CheckVerifyRow` is the row check `lyx drift compare` and `lyx drift capture --side` run first (D6): it refuses unless the current row is `Drift-Verify`, naming that row as the place they run.
The two verbs then take a run through `TakeRun`, and word an `ErrRunsSpent` refusal for the caller: the agent session is told to write what it found to `verify-report.md` and end its turn, which stops the row; a caller at a halt (`blocked`) is told that `lyx loom resume` spawns a fresh agent with a fresh budget, since a halt's compare spends the counter of no live session.

`internal/driftcli` follows `internal/hibernatecli`'s shape (PATTERN-cli-cobra): `Command`, `RunCLI`, `RunCLIIn`, every `RunE` checking `clihelp.ShouldAbort`, errors through `output.Err`, results through `output.Ok`.
Each verb resolves the worktree with `lyxcwd.Resolve` and builds its seams there: `driftengine.NewPaths(location.AnchorPath())`, `driftengine.LoadConfig(fabricengine.ConfigDirs(location))`, the run status read from `shedrun.StatusFile(location, shedrun.SelfRunID)` under its lock, the recipe from the run's seed, HEAD, the branch point (the merge base with the parent branch the origin record names) and the tracked dirty paths through `fabricengine`, the agent-session mark through `AgentSessionOpen`, the gate slots from `hubgeom.GateSlots`, and the run-process directory `fabricengine.RunProcDir(location.HubPath)` with the worktree name as owner.
Verbs, with audience `role` for an agent's and `operator` for the parent's, and `operator` for `capture`, whose `--side` form is the agent's:
`scope --scope off|partial|full [--skip <test>]...` (role), `accept <test> --reason <text>` (role), `withdraw <test>` (role), `compare` (role: `CheckVerifyRow`, `Config.Step("compare")`, `TakeRun` against `verify_max_runs`, `RunCompare`, printing the exit code and log), `capture [--side]` (operator; with `--side`: `CheckVerifyRow`, `Config.Step("capture")`, `TakeRun`, `RunSideCapture`; without it: `CheckMainCapture`, `Config.Step("capture")`, then `StartCapture` with `FallbackDetach`, which refuses while another capture holds the lock and returns once its own child's `running` record exists), `defer` (operator), `waive --reason <text>` (operator), `status` (operator: the scope and capture records, whether the capture lock and compare marker are held, and the run count), and the hidden `capture-child` (internal), the detached child's body: it holds `proc.HoldNamedJob(driftengine.CaptureJobName(worktreeRoot))`, calls `RunCapture` with `Check` re-reading the status, HEAD and dirt and calling `CheckMainCapture` under the lock, and releases the job only as its last act, after `RunCapture` returns, writing nothing to its log after that return, since `Drift-Await` may commit the log once the lock is free.
A cobra command carries one audience, so `capture`'s `Long` names `--side` as the `Drift-Verify` agent's form, which the verify stencil names directly (card 8-drift-stencils-tables); `accept` and `withdraw`'s `Long` each name the parent's use at a halt.
Each `Long` names the rules above in one paragraph and an example.
`newRoot` mounts `driftcli.Command()`.
`docs/overview.md`'s module tree gains the `internal/driftcli/` line; `driftengine`'s `doc.go` gains the verbs section.

Tests:
`TestRecordScope`, `TestAcceptWithdraw` and `TestDeferWaive` (tier0) are tables over `VerbStatus` and seeded records, one row per admission and refusal named above, each refusal asserting its way forward and that nothing was written, each admission asserting the record written: for accept the `by` and ruling under `full` and `partial`, a withdrawn test run again after `Withdraw`, refused to the agent session and to a caller outside a decision halt, admitted at one; an accept at a `running` status at `Drift-Verify` with `AgentSession` false refuses naming the wait for the row's halt and the parent's ruling, writing nothing; `Defer` refused while a test holds the capture lock; `Waive` refused at a `Drift-Verify` halt with a counted capture.
`TestAcceptWithdraw` also carries `CheckVerifyRow`'s rows: admitted at `Drift-Verify` running and blocked, refused at `Drift-Await` and `Webster` naming `Drift-Verify`.
`TestCapture_OutlivesCaller` (tier1, `!windows`) builds `lyx` with `lyxbin.Build` in a `hubforge` hub pair, whose origin record names the parent branch, and writes the run's seed with recipe `loom` and a status `blocked` at `Drift-Await`.
Its `drift.yaml` declares a capture command that fails while a marker file the test controls is absent and otherwise sleeps and then writes a file, and a fake compare command.
Rows: with the marker absent, `lyx drift capture` returns, the record later reads `failed`; with it present, a second `lyx drift capture` captures again, returns while the capture lock is held, and the record reads `passed` after the caller exited, which `lyx drift status` then prints; a third capture started by the test and still holding the lock is ended, with its command, by `runproc.End` over the pair's slug, so the owned-process end of a teardown reaches a capture on Linux; `lyx drift compare` at a status `running` at `Drift-Verify` with no `compare` declared refuses naming `lyx config drift`.
With the compare declared and `runs.json` written at `verify_max_runs`, `lyx drift compare` at a status `running` at `Drift-Verify` with the agent-session mark open refuses with the agent wording (write `verify-report.md` and end the turn), and at a status `blocked` at `Drift-Verify` refuses with the caller wording (`lyx loom resume` spawns a fresh agent with a fresh budget); each row asserts `runs.json` is unchanged.

**ImpactSummary:** `newRoot` mounts one new subtree, which the CLI tree walk and registration guards then cover; nothing else changes.
