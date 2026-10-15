# Card 4 — tackler-call-to-sheddriver

**Create:**
- `internal/sheddriver#AgentRun`
- `internal/sheddriver#AgentShuttle`
- `internal/sheddriver#RunnerShuttle`
- `internal/sheddriver#RunnerShuttle.Start`
- `internal/sheddriver#RunnerShuttle.AttachIfLive`
- `internal/sheddriver#RunnerShuttle.StopStrand`
- `internal/sheddriver#CauseTimeout`
- `internal/sheddriver#CauseDied`
- `internal/sheddriver#CauseMissing`
- `internal/sheddriver#CauseUnparseable`
- `internal/sheddriver#CausePrecondition`
- `internal/sheddriver#HandBackWayForward`
- `internal/sheddriver#AdmitRelaunch`
- `internal/sheddriver#verdictInEffect`
- `internal/sheddriver#TacklerDeps`
- `internal/sheddriver#TacklerInput`
- `internal/sheddriver#TacklerCall`
- `internal/sheddriver#NewTacklerCall`
- `internal/sheddriver#TacklerCall.Settle`
- `internal/sheddriver#ErrTacklerUnavailable`
- `internal/sheddriver#ErrTacklerResumeReturned`
- `internal/sheddriver#ConsumeTacklerAttempt`
- `internal/sheddriver#TacklerPromptInput`
- `internal/sheddriver#agentFake`
- `internal/sheddriver#agentFakeRun`
- `contracts/stencils/shed/shed-template-tackler.md`
- `contracts/stencils#ShedTemplateTackler`
- `internal/loomcli#loomTacklerRecipe`
- `internal/loomcli#darnTacklerRecipe`

**Move:**
- `internal/loomcli/tackler.go`
- `internal/loomcli/tackler_test.go`
- `internal/loomcli/tacklerconsume.go`
- `internal/loomcli/tacklerconsume_test.go`
- `internal/loomcli/tacklerprompt.go`
- `internal/loomcli/tacklerprompt_test.go`
- `internal/loomengine#TacklerSettings`

**Delete:**
- `internal/loomcli#arbiterRun`
- `internal/loomcli#arbiterShuttle`
- `internal/loomcli#runnerArbiterShuttle`
- `internal/loomcli#arbiterCauseTimeout`
- `internal/loomcli#arbiterCauseDied`
- `internal/loomcli#arbiterCauseMissing`
- `internal/loomcli#arbiterCauseUnparseable`
- `internal/loomcli#arbiterCausePrecondition`
- `internal/loomcli#admitRelaunch`
- `internal/loomcli#verdictInEffect`
- `contracts/stencils/loom/loom-template-tackler.md`
- `contracts/stencils#LoomTemplateTackler`

**Edit:**
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`
- `contracts/stencils#wordingClaims`
- `internal/loomengine#ResolveTackler` -> `func ResolveTackler(cfg Config, reg modelspec.Registry) (sheddriver.TacklerSettings, error)`
- `internal/loomengine#TestResolveTackler`
- `internal/loomcli#arbiterDeps`
- `internal/loomcli#arbiterCall.settleExisting`
- `internal/loomcli#arbiterCall.settleFresh`
- `internal/loomcli#arbiterCall.concludeWait`
- `internal/loomcli#arbiterCall.stopRunAndHandOn`
- `internal/loomcli#arbiterCall.conclude`
- `internal/loomcli#loomCLI.arbiterDepsFor`
- `internal/loomcli#consumeArbiterAttempt`
- `internal/loomcli#loomCLI.goDriverDepsFor`
- `internal/loomcli/drive.go`
- `internal/loomcli/drive_test.go`
- `internal/loomcli#arbiterFake.Start` -> `func (f *arbiterFake) Start(spec shuttleengine.Spec) (sheddriver.AgentRun, error)`
- `internal/loomcli#TestArbiterVerb_Rerun`
- `internal/loomcli#TestArbiterVerb_VerdictOutcomes`
- `internal/loomcli#TestConsumeArbiterAttempt`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Move the tackler call (spawn through shuttle, attempts and budget, deadline and pause extension, verdict validation, consume, prompt fill and spec) out of `internal/loomcli` into `internal/sheddriver` (D1), with the agent-shuttle seam, the hand-back causes and the relaunch admission it shares with loom's arbiter.
In the same card the tackler becomes recipe-neutral: its stencil moves to a shed stencil and is sharpened (D7), and the rows a `resume` may name become a seam (D2).
The arbiter itself stays loom's code and imports these from `sheddriver`.
The moved tests, over the shared fake below, cover the move; the new rows below cover the stencil and the goto seam.

The six tackler files move with `git mv` to the same names under `internal/sheddriver/`, then take the edits the package boundary forces:
- The names `internal/loomcli`'s drive loop and wiring still use take exported names, declared above: `tacklerDeps`, `tacklerInput`, `tacklerCall`, `newTacklerCall`, the call's `settle`, `errTacklerUnavailable`, `errTacklerResumeReturned`, `consumeTacklerAttempt` and `tacklerPromptInput` become `TacklerDeps`, `TacklerInput`, `TacklerCall`, `NewTacklerCall`, `Settle`, `ErrTacklerUnavailable`, `ErrTacklerResumeReturned`, `ConsumeTacklerAttempt` and `TacklerPromptInput`; the rest stay unexported.
- `loomengine.TacklerSettings` moves into `internal/sheddriver/tackler.go` under the same name, and `loomengine.ResolveTackler` re-signs to return it; its body and defaults are unchanged here and card 8 replaces it.
- `TacklerDeps.Shuttle` is an `AgentShuttle`.
  `AgentRun` and `AgentShuttle` are the method sets of today's `arbiterRun` and `arbiterShuttle` (Wait, Stop, StrandGUID; Start, AttachIfLive, StopStrand), and `RunnerShuttle` adapts a `*shuttleengine.Runner` as `runnerArbiterShuttle` does, through a `Runner` field.
- The five shared causes keep their values as `Cause*`; the tackler's own `question` and `case-brief` causes stay unexported.
- `AdmitRelaunch` and `verdictInEffect` move from `internal/loomcli/arbiterconsume.go` to `internal/sheddriver/relaunch.go`, bodies unchanged but for the `StopKey` type.
- The tackler refusals ended in loom's `arbiterstop.HandBackWayForward`, which names `lyx loom status`; they now end in `HandBackWayForward`, which names no recipe verb, since a batten run reaches the same call (D4).
  `PATTERN-refusal-way-forward` holds: the clause still names the way forward, the hand-back to the run's parent; no `refusal-spec.md` row quotes the tackler's text.
- Error and log prefixes `loom: tackler` become `sheddriver: tackler`.
- The tests' `seededStencils` helper becomes a direct `stencilkit.SeedInto` call.

The moved tackler tests are internal `package sheddriver` tests, so a test kit importing `sheddriver` would be an import cycle; per `PATTERN-testkit` the cycle-barred package keeps exactly one local copy of the fake.
The moved `tackler_test.go` therefore declares `agentFake` and `agentFakeRun`, today's `arbiterFake` and `arbiterFakeRun` with the same scripted fields, implementing `AgentShuttle` and `AgentRun`.
loom's `arbiterFake` stays in `internal/loomcli/arbiter_test.go`, its `Start` returning a `sheddriver.AgentRun`.

In loomcli the arbiter's deps, call and wiring take `sheddriver.AgentShuttle`, `sheddriver.AgentRun`, `sheddriver.RunnerShuttle{Runner: c.runner}` and `sheddriver.Cause*`; `consumeArbiterAttempt` calls `sheddriver.AdmitRelaunch`; the drive loop and its wiring call the exported tackler names; and the listed tests take the exported causes.

The recipe seams on the tackler call:
- `TacklerDeps` gains `Recipe`, the one-line recipe description the arming module tells, and `GotoRoutedFrom`, whether a `resume` may name the row the halting history entry was routed from as well as the current row.
- `TacklerDeps.Snapshot` is optional: `settleFresh` records an empty `WorktreeSnapshot` on the attempt when it is nil, so a recipe that keeps no tamper comparison wires none (D4).
- `resumePrecondition` allows the current row always and the routed-from row only under `GotoRoutedFrom`; a goto naming any other row reads as today's precondition hand-back with cause `precondition`, so a batten tackler naming the routed-from row hands back (D2, D4).
- loom's `goDriverDepsFor` tells `GotoRoutedFrom: true`, today's rule, and `Recipe` from the run's recipe: the new `darnTacklerRecipe` for `shedrun.RecipeDarn`, else `loomTacklerRecipe`.
  Each is one line naming the recipe and what its run does, for example "loom: a phased task run (discussion, plan, webster's coded batches, reviews, landing)".

The stencil (D7):
- `contracts/stencils/loom/loom-template-tackler.md` is replaced by `contracts/stencils/shed/shed-template-tackler.md`.
  `contracts/stencils/stencils.go` swaps `LoomTemplateTackler` for `ShedTemplateTackler`, embedding the new file, in `entries` (as `shed-template-tackler`, beside the other `shed-template-*` entries) and in `roleOpeningStencils`' `tackler` row.
  `tacklerStencilName` names it, read at call time through `stencilstore` (`PATTERN-stencil-ownership`), with the parent directive and edit directive markers as today (`PATTERN-parent-directive`, `PATTERN-edit-directive`).
- Its loom-specific text goes: the title and body say the shed driver, not the loom tackler or the Go driver's loom run; the `{{.driver_log}}` line names the shed driver's log; `escalate` says a stronger agent or the run's parent takes the case, naming no arbiter; the case brief's first part is the question the next judge must answer.
- New required markers, filled by `tacklerPrompt`, which refuses an empty value of either with an error naming the marker (D7: a missing value is a wiring fault):
  - `{{.recipe}}`, the told `Recipe` line, under a "The run" heading;
  - `{{.goto_rows}}`, the rows a `resume` may name: the current row in backticks, and with `GotoRoutedFrom` "or" the routed-from row.
    `settleFresh` renders it from the status the call read into a new `TacklerPromptInput.GotoRows` field, which `tacklerPrompt` fills, so the prompt test can drive it directly.
  The goto rule names `{{.goto_rows}}` as the only rows a `goto` may name.
- A "Stops you receive" list, each stop kind with how it typically looks and its usual verdict, written from the rule table:
  - a transient error that outlived its re-step (`transient` set; usually `resume` once the class has cleared);
  - `producer` (a row's own error, usually a fix the row's re-run can make, else `hand-back`);
  - `bootstrap` (the step could not start the row, usually environment, `resume` when cleared);
  - `unseeded` (the run's seed or status is missing or wrong, usually `hand-back`);
  - `interrupted` under the reinvoke policy past its two re-steps (the step child died, usually `resume` once the cause is named);
  - `blocked` (a row halted with a stuck reason, usually `resume` with a `goto` when the reason names a fix the run can make, else `hand-back`);
  - a stop it already resumed that returned unchanged, which is `hand-back` or `escalate`, never a second `resume`.
- A "Reading order": the stop envelope and the stop report first, then the stop step's trace copy and stderr, and the child traces under `{{.trace_dir}}` only when those leave the cause open.
- The write rule stays: write the verdict file and, on an `escalate`, the case brief, and nothing else.

Tests:
- `TestTacklerPrompt` gains rows: one asserts the rendered body names no "arbiter", carries the stop-kind list and the reading order, and has a write rule naming only the verdict and case-brief paths; one more renders a told recipe line and asserts the line is filled under "The run".
  Card 8's `TestTacklerFor` pins the loom and darn lines and card 13's `TestBattenDriverDeps` pins `Tackler.Recipe` for batten, so the acceptance criterion's three recipes stay covered.
  It also asserts that a `GotoRows` of the halted row alone is the only goto row the prompt names, and that an empty recipe or goto value is refused naming the marker.
- `TestTacklerCall_VerdictOutcomes` gains rows: a `resume` naming the routed-from row hands back with cause `precondition` when `GotoRoutedFrom` is false, and is accepted when it is true; a fresh spawn's prompt file names the routed-from row only under `GotoRoutedFrom`.
  `tacklerFixture` gains a `gotoRoutedFrom` field defaulting to true in `newTacklerFixture`, so the existing rows that resume to `Plan-Write` keep their outcome.
- `TestTacklerCall_SpawnPath` gains a row whose fixture leaves `Snapshot` nil: the spawn succeeds and the recorded attempt carries an empty snapshot.
- `wordingClaims`' tackler entry moves to `shed-template-tackler.md` and asserts: present, `{{.recipe}}`, `{{.goto_rows}}`, `transient`, `producer`, `bootstrap`, `unseeded`, `interrupted`, `blocked` and the reading order's "only when"; absent, "arbiter" and today's moving verbs.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/sheddriver` row rises from 3 to 9 for the six moved tests.

**ImpactSummary:** three loomcli tackler files, one loomengine type, the shared arbiter seam and the tackler stencil move to the shed layer; loom's arbiter call, consume and wiring, the stencil registry and claims, their tests and loomengine.ResolveTackler follow, with the recipe line and goto rows as the one behaviour change.
