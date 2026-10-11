# Advisor 1 notes — twb-combo-1 (Webster stability)

All code cited is read at the bench base `800a0e892` through `git show` and `git grep`, as the board entry requires.
Quarry is an external module (`github.com/Knatte18/quarry v0.2.1`, `go.mod`), read from the module cache; it is not editable inside this repo.

## Module boundaries

- Items 1 and 2 touch `internal/shuttleengine` (wait loop, gate re-prompt), `internal/websterengine` (Master spawn, verify gate) and `contracts/stencils/webster/webster-template-master.md`.
- Items 3 and 4 touch `internal/websterengine` (`recoverbatch.go`, `runwrites.go`, `state.go`), `internal/webstercli/recoverbatch.go` and `contracts/stencils/webster/webster-prefix-recovery.md`.
- Items 5 and 6 touch `internal/planglyph` (`donecheck.go`, `resolve.go`, `create.go`) and, if quarry itself is to honour build constraints, the separate quarry repository.
- `internal/websterengine/doc.go` and `internal/shuttleengine/doc.go` carry the built design for all of it, so each changed behaviour edits its doc in the same commit (CLAUDE.md "Docs land in the same commit").

## What the code already does at base, per item

### 1. Gate shell cut off at a turn end

The issues (#505, #509, #517) describe a pre-base wait: a shell expiring after `background_shell_wait_min`.
That is gone at base: `expiredTurnEnd` and `ShellWaitBound` in `internal/shuttleengine/wait.go` make the bound a log and display threshold only, and `doc.go` says "No background shell expires".
Shells outstanding at the run's end are recorded: `Result.EndedShells`, a Warn log, a `webster-background-shell` friction note and a `RunResult.Warnings` entry (`internal/websterengine/doc.go`, "Background shells in Master's wait").
The residual hole is `countsAtOnce` in `wait.go`:

- A gated run counts a waiting turn end at once when every output file exists, the run was started fresh, no gated arrival has happened yet and every outstanding task is an unawaited shell.
- Master's output files exist at that first arrival, and `lyx gate test --tags integration` is not an awaited shell: `internal/websterengine/runlevel.go` sets `AwaitedShellPrefixes: []string{masterAwaitedShellPrefix}` with the single prefix `lyx webster recover-batch`.
- The verify gate then runs, the run finishes `done`, shuttle removes the strand and the shell dies.
  The loss is recorded as a warning and a note, but the outcome still reads `done`.
- A shell label is the command itself (`payloadTask` and `launchKindAndLabel` in `claudeengine/background.go`), and `awaitedLabel` is a prefix match, so `cd x && lyx gate test` or a wrapped command would not match a prefix.

### 2. Master fixes a verify failure itself

The stencil rule exists: `webster-template-master.md`, section "A gate failure: spawn one fixer fork".
The message Master receives is built in `shuttleengine/gate.go` `gateRepromptText`: "Gate findings recorded at %s — read it, fix every finding, and end your turn."
That text is provider-neutral and tells Master to fix the findings itself, which contradicts the stencil.
The findings file's text is `renderVerifyGateFindings` in `websterengine/verifygatereport.go`; it opens "Verify gate failed (attempt N of M)." and carries no instruction about the fixer fork.
So the rule is stated only in the stencil's rules list, as the board entry says, and the two places Master actually reads a failure both say "fix".
Go cannot spawn a fork of Master's session (forks are the Agent tool's), so a "verb that spawns the fixer" cannot do what the stencil's fork does.
`runlevel.go` already renders the fixer prompt and writes it up front (`verifyFixPromptPath`) before `NewVerifyGate` is built.

### 3. Second recovery treats the first recovery's files as foreign

`uncommittedPathsBlock` in `recoverbatch.go` groups paths by `loadRunWrites` (`runwrites.go`).
`loadRunWrites` audits `recordedSessions(st)`: `State.MasterSessionID` and each batch record's `SessionID`.
Both are Master's session; `AuditForks(session)` returns that session's own writes plus its forks'.
A recovery is a separate shuttle strand with its own Claude session; `recoverSpawn` resolves it with `FindRun` (runState.SessionID is available there) but never stores the session id, and builds a fresh `BatchState` that replaces the prior one.
So no recovery strand's write is ever in the evidence, and the first recovery's 19 files are "Not written by this run" by construction, not by a run-id mismatch.
`RunWrites` is also read by `contractevidence.go` (Master-versus-fork write ordering) and `reset.go` (own paths of a reset), so a change to `loadRunWrites` itself moves those too.

`recover-batch` exit code: `webstercli/recoverbatch.go` returns `output.Ok` (exit 0) for any terminal digest with a `Digest`, including `stuck` and `dead`, and only `batch_failed` goes through `ErrFields`.
The stencil's failure ladder reads terminal `stuck` and `dead` as digests, not refusals.
I could not find a code path that exits 0 on a terminal `failed` batch; the issue's claim needs the run's envelope to place.

### 4. Recovery prompt does not name its allowed gates

`webster-prefix-recovery.md` has no gate section; the fabric ban sits in `webster-body-implementer.md` (lines about "never run a mutating `lyx fabric` verb").
The same body already says the batch gate runs as `run_in_background` calls, raw `go test` on the batch's packages only, and `lyx gate test` for module-wide or tmux-tier runs.
`renderCardGates` and `renderBatchGate` (`recoverSpawn`) already put the card and batch gate steps in the recovery prompt.
What is missing is the sentence that these are always allowed and that tmux-tagged steps are left to verify.

### 5. Create check fails on unexported targets (#511)

Quarry does not filter by export: `internal/engine/resolve.go` (`matchesFor`, `symbolsOfDir`) matches any name, and a same-package `_test.go` file belongs to the package's own unit (`walk.go` `unitFor`).
So the unexported/test-file premise in the issue is probably wrong; the two targets named in #511 are most likely the #500/#507 case: `tmuxAllowed` is declared in `allow_tmux.go` and `allow_none.go`, and `rusageStats` is probably per-GOOS.
Quarry's `statusForMatches` answers `ambiguous` for several matches, and states it deliberately does not evaluate build constraints.
`doneCheckVerdicts` (`planglyph/donecheck.go`) treats `ambiguous` as not resolved and reports "Create target %q still does not resolve", which does not carry the status or candidates.

### 6. Build-tagged and per-GOOS members

Same mechanism.
`resolved := Status == found || multipart` in `doneCheckVerdicts` is the single site for Create and Rename-new.
Plan-time: `statusFindings` (`planglyph/resolve.go`) blocks `glyph-ambiguous` for Edit and Delete targets; `createFindings` (`create.go`) blocks `create-already-exists` for an ambiguous Create target.
A Create target whose implementation will be per-GOOS cannot be seen as such at Plan time, because it resolves `not_found` then.
`go/build/constraint` is already used in this repo (`internal/impactset/guards.go`, `cmd/testtiming/spawnscan.go`), and `planglyph` is not in `PATTERN-leaf-packages`.

## PATTERN entries touched

- `PATTERN-refusal-way-forward`: a changed `create-not-done` message, a non-zero `recover-batch` exit, or a new hold/halt for a gate shell is a webster refusal or a changed row; it needs its row in `contracts/specs/refusal-spec.md` and a test in the same commit.
- `PATTERN-shuttle-provider-seam`: any shell-matching rule stays caller data (`Spec.AwaitedShellPrefixes` from webster), never a `lyx gate` literal under `shuttleengine` outside `claudeengine`.
- `PATTERN-completion-signal`: the wait changes must keep `allOutputFilesExist` consulted on every negative finalize; holding a turn end on a gate shell must not turn a finished run into `timeout` without that check.
- `PATTERN-stencil-ownership` and `PATTERN-producer-pointer-rule`: stencil edits live in `contracts/stencils/webster/`, read at call time; the recovery prompt's gate sentence must point at the existing card/batch gate blocks, not paraphrase another producer's format.
- `PATTERN-no-denied-recovery`: the named gates are `lyx gate test` and `lyx loom lint-comments`; neither is on the closed deny list, and the scan covers `contracts/stencils/`.
- `PATTERN-friction-capture`: the existing `webster-background-shell` note stays; a stop-and-record path must still write a Go-authored note.
- `PATTERN-gate-slot`: a gate shell Master or a fork starts runs in a hub slot; holding the turn end holds that slot for as long as the shell runs.
- `PATTERN-test-economy` and the test budget (`cmd/lyx/testdata/test-budget.yaml`): new tests extend existing tables (`donecheck_test.go`, `waiting_test.go`, `recoverbatch_git_test.go`) where they can; a card that pushes a package past its row lists the budget file under `Edit:`.
- `PATTERN-documentation-lifecycle`: designs land in the packages' `doc.go`, not in a new design doc.

## Board entries touching this task

- `fork-wait-noise` (Running · Discussion-Write) concerns forks ending turns while a background gate runs and waking Merriam; it overlaps item 1 and item 2 in the wait and stencil, and the two must not both rewrite `expiredTurnEnd`/`countsAtOnce` or the Master stencil's gate section independently.
- `discussion-premise-cites` (Ready) has the gate resolve glyphs a Discussion cites; it relies on the same `planglyph` resolve statuses that item 6 changes.
- `twb-combo-2`, `twb-twp-*` and `twb-twc-*` carry the same title and brief (bench siblings, not distinct work).

## Conflicts seen

- The generic re-prompt "fix every finding" (`gateRepromptText`) contradicts the Master stencil's fixer-fork rule; any stencil-only fix for item 2 loses to the later, shorter message Master reads last.
- Item 5's premise (unexported or test-file identifiers do not resolve) does not match quarry's code; the likely cause is item 6.
- Item 6's "quarry honours build constraints" option changes an external module and its documented position (no GOOS/GOARCH assumed); it cannot land in this repo's commit.
- Item 3's `recover-batch` exit: the stencil's ladder treats terminal `stuck`/`dead` digests as exit-0 envelopes; a blanket non-zero exit changes those rungs.
