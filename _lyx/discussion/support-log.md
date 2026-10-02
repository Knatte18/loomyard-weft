# Support log — gate-list

## Interview

The session ran autonomously; every answer below is a self-pick recorded in the question ledger.

1. **Task read.**
   The board entry (`gate-list`, id 8) fixes most of the shape: a list of gates, per-gate `attempts` and `pass_on_cap`, first-failure stop, consecutive-failure counts reset on pass, `attempts: 0` off, existing rows migrate unchanged, `GateOutcome` reports per gate.
2. **Code read.**
   `shuttleengine` holds one `GateSpec{Gate, Attempts}` with `Attempts <= 0` meaning default 3; Wait's gated-Done branch re-prompts while `gateSent < attempts()`, then finalizes Done with a failed `GateOutcome`, which `shedadapters` maps to Stuck.
   `shedrecipe.resolveGateSpec` reads `gate`/`gate_attempts` for six rows in `loom-recipe.yaml`; `batten-recipe.yaml` has none.
   `burlerengine` wraps the closure to append a report-repair instruction.
   Consumers read `Result.Gate.Passed`, `.Attempts`, `.FindingsPath`; the shed status history stores one `GateAttempts` integer.
3. **What "arrival" means.**
   Picked: a gated `OutcomeDone` inside one shuttle run, the turn boundary Wait already gates on.
   Counts stay in-memory per run, as `gateSent` is.
4. **What a capped `pass_on_cap` gate does.**
   Picked: not run again on that run, counted as let through, count not reset.
   Running it and ignoring the verdict would waste an expensive review; resetting would let a review re-fire after a later gate's re-prompt.
5. **`attempts: 0`.**
   Board says off.
   Picked: skipped regardless of `pass_on_cap`, the old default removed, `attempts` required in YAML because no default is safe once 0 means off.
6. **GateOutcome shape.**
   Picked: keep aggregates with unchanged meaning (sum of re-prompts, failing gate's findings path) and add a per-gate slice.
   This leaves every consumer and the status file untouched and makes single-gate rows identical to today.
7. **Findings file and re-prompt text.**
   Picked: unchanged, one file overwritten by the failing gate, so existing rows send byte-identical re-prompts.
8. **Recipe key.**
   Picked: one `gates` list of `{name, attempts, pass_on_cap}` maps; old keys removed, not aliased.
9. **Burler wrapper.**
   Picked: wrap every entry.
10. **Docs.**
    Picked: recipe comments, type doc comments, and a short paragraph in `contracts/specs/shed-recipe-spec.md`; no roadmap or invariant change.

## Rejected alternatives

- **Keep `gate`/`gate_attempts` as a legacy single-gate form beside `gates`.**
  One recipe file exists and migrates in the same commit; two accepted forms is a second parser path that only hides stale keys.
- **`Attempts: 0` keeps meaning "default 3".**
  Contradicts the board's "0 turns a gate off" and gives 0 two meanings.
- **`attempts` optional with a default of 3 in YAML.**
  With 0 meaning off, an omitted key would silently mean something the author did not write.
- **A capped `pass_on_cap` gate still runs and its verdict is ignored.**
  Spends a review for nothing and makes "one-time request" untrue.
- **A let-through resets the count.**
  The review gate would re-fire after any later gate's re-prompt, turning a one-time request into a loop.
- **`Result.Gate` becomes `[]GateOutcome`.**
  Breaks every consumer for no gain, and the status file would still need an aggregate.
- **Per-gate findings files (`gate-<name>-findings.md`).**
  The re-prompt names one file at a time anyway, and existing rows would see a different path.
- **Per-gate counts persisted in the shed status file or across attach.**
  Not asked for; today's `gateSent` is in-memory and the board does not change that.
- **A run-wide cap across all gates to bound alternating failures.**
  The board chose reset-on-pass; the run deadline already bounds it, and no row has two gates yet.
- **Dropping off (`attempts: 0`) entries at recipe build so the run looks ungated.**
  Hides the configured gate from the per-gate report and makes `Result.Gate` nil-versus-non-nil depend on a budget value.

## Review rounds

_No rounds yet._

## Question ledger

| # | Question | Resolution |
|---|----------|------------|
| 1 | Does the board task exist? | Resolved: yes, `gate-list` id 8. |
| 2 | What is an "arrival"? | Auto-pick: a gated `OutcomeDone` within one shuttle run; counts in-memory per run. |
| 3 | Does a capped `pass_on_cap` gate run again? | Auto-pick: no; let through for the rest of the run, count not reset. |
| 4 | What does `attempts: 0` do with `pass_on_cap`? | Auto-pick: off either way; reported off. |
| 5 | Is `attempts` required in YAML? | Auto-pick: yes; 0 means off, so no safe default. |
| 6 | Is a list of only off entries gated? | Auto-pick: yes; `Result.Gate` non-nil, `Passed` true. |
| 7 | Shape of per-gate reporting? | Auto-pick: aggregates kept, per-gate slice added with five states (passed, failed, let through, off, not reached). |
| 8 | One findings file or one per gate? | Auto-pick: one, unchanged name and re-prompt text. |
| 9 | Keep old `gate`/`gate_attempts` keys? | Auto-pick: no; removed and rejected as unknown. |
| 10 | Empty `gates` list, duplicate names? | Auto-pick: both construction errors naming the row. |
| 11 | Which closures does `repairReportBeforeGate` wrap? | Auto-pick: every entry. |
| 12 | Do new construction errors need refusal-spec rows? | Open for Plan: confirm against `contracts/specs/refusal-spec.md` whether recipe-build errors are reachable through a bound verb. |
| 13 | Bound on two alternating non-cap gates? | Auto-pick: none beyond the run deadline; recorded as an open risk. |
| 14 | Docs touched? | Auto-pick: recipe comments, type doc comments, `shed-recipe-spec.md` paragraph; no roadmap or `CONSTRAINTS.md` change. |
