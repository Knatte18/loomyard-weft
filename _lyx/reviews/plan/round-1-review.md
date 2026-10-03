---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/07-anomalies-under-step.md
    summary: Card 7 runs Tier 1 anomaly filing under step, contradicting the decision record's Out item that Tier 1 stays run-only; the overview, cards 2, 6, 10 and 11 carry its dependent parts
  - id: F2
    severity: BLOCKING
    location: _lyx/plan/08-anomaly-pending-retry.md
    summary: Card 8's pending-retry marker for failed Tier 1 filings is licensed by no Decision, Constraint or acceptance criterion in the decision record
  - id: F3
    severity: BLOCKING
    location: _lyx/plan/00-overview.md
    summary: The step error envelope stays extras-free decision drops the friction status from a failed halt's envelope under step and run, so D2's acceptance criterion for failed is unmet
  - id: F4
    severity: MEDIUM
    location: _lyx/plan/06-halt-notes.md
    summary: Card 6 makes both verbs reflect, and so file public issues, at blocked and failed halts, but its Intent states no bound beyond the held-lock skip
  - id: F5
    severity: MEDIUM
    location: _lyx/plan/01-reflect-reentrant.md
    summary: Card 1's settle-then-spawn sequence can block one call for an attached wait plus a fresh spawn, up to twice friction_timeout_min, breaching D1 and D2's bound without saying so
  - id: F6
    severity: LOW
    location: _lyx/plan/01-reflect-reentrant.md
    summary: Card 1 leaves the record's read, parse and write failures and the settle path's spec-build failure unspecified against the nil-error constraint
  - id: F7
    severity: LOW
    location: _lyx/plan/06-halt-notes.md
    summary: No card tests that a done step whose row reflection failed reports friction failed on its envelope, and the done-when scenario names no fake issue filer
  - id: F8
    severity: LOW
    location: _lyx/plan/09-drive-skill-filing.md
    summary: Card 9's rule that every stop report records the envelope's friction value is not licensed by D4
---

### [BLOCKING] Tier 1 under step contradicts the decision record's Out list

**Location:** `_lyx/plan/07-anomalies-under-step.md`; dependent parts in `_lyx/plan/00-overview.md` (summary line, "operator additions relayed by the hub", "who files a halt", "one fakeable reflection seam"'s `fileIssue` mention), `02-step-after-hook.md` (Intent's "file Tier 1 anomalies"), `06-halt-notes.md` (`tierOneOn` and the ownership line and its test), `10-reflection-stencil.md` (the "Tier 1 files the halt itself" rule), `11-friction-capture-docs.md` (the CONSTRAINTS clause on Tier 1 under step and the selfreport module line).

**Issue:** The decision record's Scope Out reads "Tier 1 anomaly filing (`detectAndFileAnomalies`) stays `run`-only."
No rework round exists (`_lyx/loom/rework/` is absent), so the decision record alone is the answer key.
Nothing in it licenses Tier 1 under step: no Decision, Constraint or acceptance criterion mentions it.
The overview's "operator additions relayed by the hub" section is plan text, not answer key; the plan overrides the answer key on its own authority.
The board entry does carry the addition, which means the right route is amending the decision record (a `goto` to Discussion), not a plan card that contradicts it.
The halt-note ownership line, the stencil's "do not file the halt" rule and the CONSTRAINTS clause exist only because card 7 does; with Tier 1 run-only, D2's halt note under step has a single filer (the reflection), and under `run` the decision record asks for no ownership split.

**Fix:** Remove card 7 and every dependent part listed above, renumbering the cards after it.
Leave a Shared Decision recording that the board's post-Discussion addition is not carried because the decision record keeps Tier 1 `run`-only, so the hub can take it back to Discussion.

### [BLOCKING] Card 8's pending-retry scope is unlicensed

**Location:** `_lyx/plan/08-anomaly-pending-retry.md`; the overview's "resumed-halt loss in Tier 1" finding.

**Issue:** The decision record never mentions `internal/loomengine/anomaly.go`, the filed-title marker or retrying failed Tier 1 filings.
Card 8 changes the machine-local marker format and the behaviour of a public-issue filer on the strength of a board item the decision record did not carry.

**Fix:** Remove card 8 and the overview's finding section, recording it beside F1's note as not carried.

### [BLOCKING] A failed halt's envelope drops the friction status

**Location:** `_lyx/plan/00-overview.md` "the step error envelope stays extras-free"; `02-step-after-hook.md`; `06-halt-notes.md` (`loomAfterStep`'s failed arm, `loomPostRun`'s failed arm).

**Issue:** D2 says the step envelope carries the same `friction` key `run`'s does, and the acceptance criterion says a blocked step's envelope carries the friction status, "the same holds for `failed`, and for both states under `run`".
A failed halt always reaches the verb as a non-nil error from `shed.Step`/`shed.Run` (`internal/shedengine/run.go`'s producer-error and unrecognised-outcome arms), so it always prints an error envelope.
The plan's decision keeps both error envelopes extras-free, so a failed halt under either verb reports no friction status, and the criterion fails for exactly the case it names.
The Constraint "adding `friction` to step envelopes updates both" (plural) anticipates the key on error envelopes too.

**Fix:** Replace the overview's decision with one that carries the status on the error envelope:
the step error envelope's closed key set gains `friction` on every error envelope (the `AfterStep` return after a `shed.Step` error, empty on a `PreStep` or `BuildShed` refusal and with a nil hook);
the generic `run` body merges `PostRun`'s extras onto the error envelope as onto the success envelope, and `battenPostRun` returns nil on a non-nil `runErr` so batten's error envelope is unchanged.
Card 2 takes the `run.go`/`run_test.go`, `spec.go` doc and `battencli` edits and their tests; card 6's failed arms return their status instead of discarding it; the verify section adds `internal/battencli`.

### [MEDIUM] Card 6 states no bound for halt-time filing

**Location:** `_lyx/plan/06-halt-notes.md`, Intent.

**Issue:** Card 6 removes `shouldReflectFriction` and makes `step` and `run` reflect at `blocked` and `failed` halts, so the reflection agent now files public issues from every halt, `failed` and step halts included.
Its Intent mentions the held-lock skip in passing but has no statement of what it now lets through and what bounds it: D2's `friction_timeout_min` delay on the halt envelope, filing only through `lyx selfreport create` to `Knatte18/loomyard`, and D6's content rule.

**Fix:** Add a "What this lets through, and the bound" paragraph to card 6 naming those bounds.

### [MEDIUM] Settle-then-spawn doubles the reflection's time bound

**Location:** `_lyx/plan/01-reflect-reentrant.md`, `Reflect` steps 1 and 3.

**Issue:** `Runner.Attach` sets an attached run's deadline to a fresh `now+Timeout` (`internal/shuttleengine/attach.go`), so step 1's wait can last a full `friction_timeout_min`, and step 3 can then spawn a new agent for another full `friction_timeout_min` within the same call.
D1 bounds the row's block, and D2 the halt envelope's delay, at `friction_timeout_min`; the card neither keeps that bound nor states the breach.
The attach-probe identity itself is sound: `Attach` matches on the output-file set, which is the fixed report path, so the dead process's agent is found whatever note list the rebuilt spec carries; and one call spawns at most one agent, over notes outside the settled covered set, which keeps D5's spawn bound.

**Fix:** Bound the whole call by one `friction_timeout_min` measured from entry through `Deps.Clock`: the spawn after a settle gets the remaining budget as its timeout, and with none left it spawns nothing and returns `StatusFailed` with the remaining notes in place; add a test for the no-budget case.

### [LOW] New runtime failures in Reflect are unspecified

**Location:** `_lyx/plan/01-reflect-reentrant.md`.

**Issue:** The record adds failure points: reading or parsing it, writing it before the spawn, and building the spec on the settle path.
The card does not say what each returns, while the Constraint requires every runtime failure below Deps validation to return a nil error, and an unparsable record handled as a hard failure would block every later reflection in the task.

**Fix:** State that each returns `StatusFailed` with a nil error and leaves the notes in place, and that an unparsable record is logged and discarded like a record with no report and no live agent.

### [LOW] Acceptance criteria without a test

**Location:** `_lyx/plan/06-halt-notes.md` tests; `_lyx/plan/03-reflect-under-step.md` tests.

**Issue:** "a done stop's envelope carries `friction: failed`" is carried in behaviour (`loomAfterStep`'s done arm reports `rowFrictionStatus`) but no card tests it; card 1's engine-level failure cases stop at `Reflect`'s status.
The done-when scenario names "a fake shuttle and fake issue filer"; card 6's scenario names only the fake shuttle and does not say that no real filer is reachable.

**Fix:** Add a `halt_test.go` case: a done step after the row recorded `failed` reports `friction: failed` on the envelope.
State in the scenario that the fake shuttle stands in for the agent's `lyx selfreport create` filing and that the receiver's `selfreport` knob is off, so no Go filer runs.

### [LOW] Card 9 adds a stop-report rule D4 does not license

**Location:** `_lyx/plan/09-drive-skill-filing.md`, "The stop report of every stop records the envelope's `friction` value."

**Issue:** D4 licenses `friction_dir` records, the optional note, no filing, the `friction: failed` notification and dropping row names.
Recording the friction value in every stop report is new skill text the answer key does not ask for.

**Fix:** Remove the sentence.

## Focus items settled without a finding

The verify-coverage directive is settled with no finding: `contracts/recipes` has no tests of its own, `docs/overview.md` and `CONSTRAINTS.md` belong to no Go package, and the rubric's rule covers tests of a targeted package, not consumers such as `internal/loomrecipe` or `internal/lyxcwd`.
The attack-surface directive's card 2 (`AfterStep`) and card 10 (`TaskSlug` rejection) items are settled with no finding: card 2's Intent states when the hook runs and that it adds no extras to an error path, and card 10 adds a guard rather than removing one, whose wiring-bug error stays above the nil-error line.
The constraint-coverage directive is settled with no finding beyond F6: every Constraint bullet and every listed doc has a carrying card.
