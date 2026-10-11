---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/08-burler-seat-table.md (Probe and resume)
    summary: ProbeRound on seatengine.Probe stops a live reviewer whose chair is gone, yet card 8 calls the other-combination handling unchanged and keeps two test rows that pin the old probe
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/08-burler-seat-table.md (decideRound, reviewer done with its review gate failed)
    summary: the gate-failed reviewer branch has no input that reaches it, since ReviewGateEntry is PassOnCap and never Terminal, and no named test row
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/08-burler-seat-table.md (Resume refuses when the probed table is absent)
    summary: the new absent-table Resume refusal names neither the input that reaches it nor a test row
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/10-remove-await-review.md
    summary: the decision record's burlercli doc.go update is carried by no card, and internal/burlercli has no doc.go
  - id: F5
    severity: LOW
    class: design
    location: _lyx/plan/03-seat-gate-fork-round.md (Seat.ForkSubagents)
    summary: per-seat fork authorization states what it lets through but not what bounds it
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/08-burler-seat-table.md (Delete group)
    summary: card 8 removes the last callers of Shuttle, runnerShuttle and RunnerShuttle and deletes them in the same card, while the plan splits FixStart into cards 7 and 11
  - id: F7
    severity: NIT
    class: design
    location: _lyx/plan/08-burler-seat-table.md (The seam)
    summary: Engine holds the seat engine behind an unexported three-method interface with one implementer, and no Create bullet declares that interface
---

### [MEDIUM] ProbeRound's seat probe settles a reviewer-only round itself, contrary to card 8's "unchanged"

**Location:** `_lyx/plan/08-burler-seat-table.md`, "Probe and resume (D11)".

**Issue:**
Card 8 has `ProbeRound` run `seatengine.Engine.Probe`.
When the chair is not live, `Probe` stops every live advisor and returns an empty `LiveTable` (`internal/seatengine/resume.go:27-60`).
So a live reviewer next to a gone fixer is stopped inside `ProbeRound` and then reported as not live.
The card still says "Any other combination is returned to `BurlerProducer`, which stops what is live and runs a fresh round, unchanged."
D11 says the same: the other combinations are "returned for `BurlerProducer` to settle by stopping what is live".
That stops being true for the reviewer-only case, and `BurlerProducer.probeLiveRound`'s `case reviewLive:` branch becomes unreachable from the real engine (`internal/shedadapters/burler.go:546-552`).
Two existing rows pin the old behavior, and the card's test bullet ("probe classification per seat") does not mention changing them:
- `TestEngine_ProbeRound`, row "found run with its output file on disk stays live" (reviewer live, fixer gone, wants `HalfLive, HalfGone`), `internal/burlerengine/engine_test.go:1565`;
- `TestEngine_Resume`, row "a round without both halves live is refused and nothing is stopped", `internal/burlerengine/engine_test.go:1648`.

When `Probe` fails to stop that reviewer, it returns `ErrSeatNotStopped` as a probe error.
The card wraps `ErrSeatNotStopped` in `ErrHalfNotStopped` only on `Run`'s path, not on `ProbeRound`'s.
Executed as written, the card hits these failing rows with no recorded decision about which behavior is correct.

**Fix:**
Record the decision in card 8.
The recommended route accepts the seat engine's behavior.
The card says `seatengine.Probe` stops a live reviewer whose chair is not live, so `ProbeRound` reports it `HalfDone` or `HalfGone` by its file and `BurlerProducer` runs a fresh round, the same outcome as today.
Its test bullet updates the two rows above: the probe row wants the reviewer stopped and not live, and the resume row wants it stopped by the probe.
`ProbeRound` wraps a probe stop failure in `ErrHalfNotStopped` as `Run` does.
The alternative is a `ProbeRound` that classifies seats without `seatengine.Probe`'s stop, but that departs from D11's "runs the seats engine's probe".

### [LOW] decideRound's gate-failed reviewer branch is unreachable and untested

**Location:** `_lyx/plan/08-burler-seat-table.md`, the decideRound bullet "the reviewer ended the step done with its review gate failed".

**Issue:**
The card prescribes an error branch for a reviewer that ended done with its gate failed.
The upstream check that rules it out is `ReviewGateEntry` itself.
It is `PassOnCap: true` (`internal/burlerengine/verdict.go:176`), so a spent budget lets the run through.
`ReviewGate` never sets `Terminal`, which shuttle refuses on a PassOnCap entry anyway (`internal/shuttleengine/gate.go:32`).
No real reviewer run can reach the branch.
The card's own clause "which the seats engine's generic rule alone can produce" names no input that produces it.
`TestEngine_Run_RoundFailureRules`'s list of rows does not name it either.

**Fix:**
Keep the branch as a fail-loud wiring guard, since `decideRound` still has to map every shape a seat result can take, and say so in the Intent.
Alternatively, name the `TestEngine_Run_RoundFailureRules` row that scripts the fake reviewer's done result with `Gate.Passed` false and asserts the error.

### [LOW] Resume's absent-table refusal names no input and no test

**Location:** `_lyx/plan/08-burler-seat-table.md`, "`Resume` refuses, as today, unless both halves are live, and also when the probed table is absent".

**Issue:**
The absent-table refusal is new.
The input that reaches it is a `LiveRound` whose two halves are `HalfLive` but which no `ProbeRound` built, the shape `shedadapters`' tests hand-build with `roundWith(burlerengine.HalfLive, burlerengine.HalfLive)` (`internal/shedadapters/attachprobe_test.go:90`).
The card names neither that input nor a `TestEngine_Resume` row that reaches the refusal.

**Fix:**
Name that input in the Intent and add a `TestEngine_Resume` row that calls `Resume` with a hand-built both-live `LiveRound` and asserts the refusal with nothing started or stopped.
Alternatively, state that the refusal is a fail-loud wiring guard, since `BurlerProducer` only passes `ProbeRound`'s answer.

### [LOW] burlercli's doc update has no card

**Location:** `_lyx/plan/10-remove-await-review.md`.

**Issue:**
The decision record's Scope lists `doc.go` of `burlercli` among the same-commit docs.
`internal/burlercli` has no `doc.go`, and its only package-level comment is `cli.go`'s file header.
Card 10 removes a verb from that CLI, an observable CLI change.
It edits `resolvePersistentPreRun`'s comment but writes or names no burlercli module doc.
No other card carries that Scope item.

**Fix:**
Card 10 adds `internal/burlercli/doc.go` under `**Create:**`, describing the verbs that remain (`run`, `validate-review`) and that the fixer waits with `lyx shuttle await`.
Alternatively, its Intent names where burlercli's module doc lives and updates that doc.

### [LOW] Per-seat fork authorization is unbounded in its card

**Location:** `_lyx/plan/03-seat-gate-fork-round.md`, `Seat.ForkSubagents`.

**Issue:**
`ForkSubagents` is a new seat flag that lets a seat's session spawn in-session forks.
The Intent says what it authorizes but not what bounds it.
The seat engine audits no forks, so a table that sets the flag gets no check from `seatengine`.
The only bound is the caller's own audit, which for Burler is the cluster audit in `accept` (card 8).
The rubric's attack-surface rule requires the introducing card to state that.

**Fix:**
Add a bound to card 3's Intent.
The flag only passes through to the seat's `shuttleengine.Spec.ForkSubagents`.
A table that leaves it unset authorizes no forks.
The seat engine audits none, so the caller that sets it owns the fork audit, as Burler's `accept` does.

### [LOW] Card 8 deletes symbols whose last callers it removes, unlike the plan's FixStart split

**Location:** `_lyx/plan/08-burler-seat-table.md`, the `**Delete:**` group.

**Issue:**
The plan follows the spec's rule that removing a symbol's last caller and deleting the symbol are two cards for `FixStart`: card 7 removes the uses and card 11 deletes the declarations.
Card 8 instead moves every caller of `burlerengine.RunnerShuttle` and `Shuttle` (`wireHub`, `wireStandalone`, `loomCLI.wire` and both smoke tests).
It also deletes them in the same card, along with `runnerShuttle`, its two methods and `shuttle_test.go`.
That makes card 8, already the plan's largest, larger still.

**Fix:**
Move `Shuttle`, `runnerShuttle`, `RunnerShuttle`, `runnerShuttle.StartGated`, `runnerShuttle.ProbeGated` and `internal/burlerengine/shuttle_test.go` into a later `Delete` card, as card 11 does for `FixStart`.
Alternatively, fold card 11 into card 8's pattern and note why one card suffices.

### [NIT] An interface with one implementer

**Location:** `_lyx/plan/08-burler-seat-table.md`, "The seam".

**Issue:**
`Engine` holds the seat engine "behind an unexported field typed by the three methods the round uses", which `*seatengine.Engine` satisfies.
Per the shared decision `burler-builds-its-seat-engine`, burlerengine's tests drive the real seat engine, so the interface has one implementer and no second call site.
No `**Create:**` bullet declares the interface either.

**Fix:**
Hold a `*seatengine.Engine` field directly.

## Settled with no finding

- **Fidelity:** D1 through D11, the Constraints and the Acceptance criteria are each carried by a card (D11 has a gap, see F1).
  That includes the four `Table.Validate` refusals (cards 3 and 4), the `lyx burler run` caller move (`wireHub` and `wireStandalone`, cards 6 and 8), the `roleOpeningStencils` re-keying (card 8), and the doc.go updates for burlerengine, seatengine and shuttleengine (burlercli is F4).
  `await-review` appears in `internal/burlercli/cli.go`, `await.go`, `cli_test.go`, `internal/burlerengine/engine.go`, `handoff.go`, `doc.go` and `prompt_test.go`, `internal/burlermarker/doc.go`, the fixer stencil, `claims_test.go`, `docs/overview.md` and `SANDBOX-BURLER-SUITE.md`, and a card covers each of them.
  `fix_start` and `FixStart` appear in `loomengine`, `shedrecipe`, `loomcli`, `burlerengine` and `refusal-spec.md`, and cards 7 and 11 cover all of them.
- **Card order:** card 7 builds at its position, since only declarations remain for card 11.
  Card 8 runs on the old stencil bodies: every marker they name is in the seat values card 8 supplies, and their `parent_directive` and `edit_directive` markers are filled by the seat engine.
  Card 9 uses only `chair_name` and `advisor_names`, which the seat engine already reserves and fills (`internal/seatengine/table.go:27-39`).
  Card 10 follows card 9's move off the verb, and card 11 follows cards 7 and 8.
- **Attack surface:** cards 1, 4 and 5 each state their bound, and card 10's removal states its recovery.
  Card 3 is F5.
- **Edit-card glyphs:** cards 3, 4, 5 and 8 re-sign with the arrow every member whose signature changes (`startSeat`, `probeSeat`, `New`, `notStartedResult`, `halfOf`, `decideRound`, `composePrompt`).
  Struct and interface changes carry no arrow.
  No Edit card lists only Go file paths while naming symbols.
- **Test economy:** card 8 reuses the existing `burlerengine` test functions.
  The new `seatengine` tests in cards 4 and 5 cover behaviors no existing test covers: an early end by a required advisor, and the chair's hold deferral.
  Each card raises the `seatengine` budget row.
- **Verify:** `./...` covers every targeted package, including `contracts/stencils`, which belongs to the root module.
  The section's order matches the plan template.

