# Gate list: several gates per row, each with its own budget and pass_on_cap

## Goal

A producer row carries an ordered list of gates instead of one.
Each gate has its own attempt budget and a `pass_on_cap` flag,
so a gate can be either a check that must pass or a one-time request that lets the run through once its budget is spent.
This task changes no existing gate's behaviour;
it lays the shape `parent-review` builds its review gate on.

## Scope

In:

- `shuttleengine.GateSpec` becomes an ordered list of gate entries, each with a name, a closure, an attempt budget and a `pass_on_cap` flag.
- The Wait loop's gated-Done branch and `finalize`'s verdict site evaluate the list with per-gate consecutive-failure counts.
- `GateOutcome` keeps its aggregate fields and gains a per-gate report.
- `shedrecipe` reads a new `gates` list key in place of `gate`/`gate_attempts`, which are removed.
- Every gated row in `contracts/recipes/loom-recipe.yaml` migrates to the list form with identical behaviour.
- `burlerengine`'s report-repair wrapper applies to every gate in the list.
- Docs: the recipe file's own comments, the code doc comments on the changed types, and a short gates paragraph in `contracts/specs/shed-recipe-spec.md`.

Out:

- Parallel gates, dependencies between gates, any new gate validator (the review gate belongs to `parent-review`).
- Persisting per-gate counts across a driver restart or an attach; counts stay in-memory per run, as `gateSent` is today.
  After an attach a `PassOnCap` entry starts from zero and can fire again, so "one-time" holds within one shuttle run only;
  the `PassOnCap` doc comment states this limit and Decision 4's, and that a request which must not repeat makes its own closure idempotent.
- Per-gate detail in the shed status file; `HistoryEntry.GateAttempts` stays one integer.
- A run-wide cap across all gates.

## Decisions

1. **GateSpec shape.**
   `GateSpec` becomes a list of entries (working name `GateEntry`: `Name`, `Gate` closure, `Attempts`, `PassOnCap`), carried as one value through every hop exactly as today (`StartGated`, `RunGated`, `AttachGated`, `RunOpts.Gate`, `websterengine`'s `StartMaster`).
   An empty or nil list means ungated.
   Rationale: one value at every hop keeps the existing "one GateSpec at every hop" contract; a named list keeps the zero value meaning ungated.

2. **Arrival evaluation.**
   An arrival is a gated `OutcomeDone` (the turn boundary Wait already gates on, plus `finalize`'s single verdict site).
   At each arrival the entries run in list order and evaluation stops at the first that fails; a later entry runs only when every earlier one passed, was off, or was let through on that arrival.
   The verdict is memoised per arrival exactly as `gateVerdict` is today, and cleared before each re-prompt.
   `finalize`'s arrival is the final one: it reads the memo when Wait left one (exhausted, re-prompt send failed) and evaluates the list afresh otherwise (session lost or deadline after a re-prompt), under the same ordering and cap rules, but it never re-prompts and never changes a count.
   Rationale: the board's ordering rule, and it keeps "the gate runs exactly once per settling".

3. **Per-gate counting.**
   Each entry keeps its own count of consecutive failures on this run.
   When an entry fails with count < `Attempts`, Wait re-prompts and that entry's count increments.
   When an entry passes, its count resets to zero.
   Rationale: a validator listed before a review re-runs at every arrival, and its budget starts afresh after it passes.

4. **Cap behaviour.**
   An entry whose count has reached `Attempts`:
   without `PassOnCap`, its failure on that arrival ends the loop and the run finalizes Done with a failed `GateOutcome`, as an exhausted gate does today (the producer then maps it to Stuck);
   with `PassOnCap`, the entry is not run on that or any later arrival of this run, counts as let through, and evaluation continues to the next entry.
   A let-through does not reset the count, so once capped a `PassOnCap` entry never runs again on this run.
   An entry that passes before it is capped resets per Decision 3 and re-runs at later arrivals, so if it fails again it starts a fresh streak of up to `Attempts` re-prompts;
   a request that must not repeat after it passed makes its own closure idempotent.
   Rationale: a capped request (review) must not re-fire after a later gate's re-prompt, and an expensive gate should not run when its verdict would be ignored.

5. **`Attempts: 0` is off.**
   An entry with `Attempts` 0 is skipped at every arrival regardless of `PassOnCap`, and reported as off.
   `defaultGateAttempts` and the "0 means default" rule are removed.
   A run whose list holds only off entries is still a gated run: `Result.Gate` is non-nil with `Passed` true.
   Rationale: the board defines 0 as off; a hidden default would make 0 mean two things.

6. **GateOutcome.**
   Aggregate fields keep their meaning so every consumer (`shedadapters` single-LLM and burler, `burlerengine`, the shed status history) is untouched:
   `Passed` is true only when every entry without `PassOnCap` that is not off passed at the final arrival, so an entry without `PassOnCap` that failed or was not reached makes it false, however the run ended (exhausted, re-prompt send failed, session lost, or deadline);
   `Attempts` is the total re-prompts actually sent on the run, summed over entries;
   `FindingsPath` is the findings file of the entry that failed at the final arrival, empty when `Passed`.
   A new per-gate slice reports, in list order, each entry's name, its re-prompts sent, and its state at the final arrival: passed, failed, let through, off, or not reached.
   Rationale: the board asks for per-gate reporting; keeping the aggregates means a single-gate row's result is identical to today's.

7. **Findings file and re-prompt text unchanged.**
   One `gate-findings.md` in the run directory, overwritten by whichever entry failed; `gateRepromptText` unchanged.
   The re-prompt Warn log line gains the failing entry's name.
   Rationale: the agent reads one current complaint at a time, and byte-identical re-prompts are what "behaves exactly as before" means for existing rows.

8. **Recipe key.**
   `gate`/`gate_attempts` are replaced by one `gates` key, a list of maps:
   ```yaml
   gates:
     - name: discussion
       attempts: 3
   ```
   Per element: `name` required, from the existing closed vocabulary (`discussion`, `plan`, `rework-plan`, `description`) with the same `Env` requirements as today;
   `attempts` required, a non-negative integer;
   `pass_on_cap` optional bool, default false;
   any other element key is an error.
   An absent `gates` key is ungated.
   A present but empty list, a duplicate `name` within one list, and a negative `attempts` are construction errors naming the row.
   The old keys are not accepted; each gate-capable entry's `configRejectUnknown` drops them and accepts `gates`.
   Rationale: one recipe file in the repo, migrated in the same commit, so a compatibility path would only hide a stale key.
   `attempts` is required because 0 now means off, so an omitted key has no safe default.

9. **Burler wrapper.**
   `burlerengine`'s `repairReportBeforeGate` wraps every entry's closure in the list.
   Rationale: whichever gate fails, the round's review and fixer report were written before it ran.

10. **Docs.**
    The `loom-recipe.yaml` header and row comments describe the list form;
    `contracts/specs/shed-recipe-spec.md` gains a short paragraph on the `gates` key, beside its existing `commit_seam` selection rule.
    No `manifest/roadmap.md` change and no new `CONSTRAINTS.md` invariant.
    Rationale: the recipe key is an author-facing contract; nothing here is cross-cutting.

## Constraints

- Told-Geometry Invariant: `shuttleengine` and `shedrecipe` stay bound; a gate closure captures its own told paths.
- Shed Producer-Seam Invariant: `shedengine` is not touched beyond what already exists; `HistoryEntry.GateAttempts` keeps its shape.
- Transient Stop Invariant: a gate verdict never carries the transient mark.
- Shed Recipe Registry Invariant: `resolveGateSpec` stays the single place gate config is read; no self-registration.
- `completionsignal_enforcement_test.go`'s tripwire: the negative-verdict `Errorf` marker stays reserved for the two finalize call sites.
- Refusal Way-Forward Invariant: recipes are embedded, so the new shedrecipe construction errors are unreachable from any on-disk state a run can produce;
  they are wiring guards covered by each section's grouped row in `contracts/specs/refusal-spec.md`, and no refusal-spec row is added.
- Markdown semantic line breaks in every `.md` touched.

## Auto-mode assumptions

- Arrival means a gated `OutcomeDone` within one shuttle run; counts do not survive a new run, a bounce, or an attach (auto-pick).
- A `PassOnCap` entry stays let through for the rest of the run once capped (auto-pick).
- An `Attempts`-0 entry still resolves its closure and `Env` at construction, so a typo in an off gate fails loud (auto-pick).
- The per-gate state at the final arrival uses the five states named in Decision 6; exact type names are the Plan producer's call (auto-pick).
- New shedrecipe construction errors are wiring guards, not refusals needing their own rows, because the recipes are embedded (auto-pick).

## Open risks

- Two non-cap gates that alternate failures reset each other's counts and can re-prompt until the run's deadline (`spec.Timeout`); the board chose reset-on-pass, and the deadline is the bound.
  No current row has two gates, so nothing in production can hit this before `parent-review`.
- `GateOutcome.Attempts` as a sum loses which gate spent the re-prompts in the status file; the per-gate slice carries that only on the in-process `Result`.
- `contracts/recipes/loom-recipe.yaml` is also edited by `rework-generations` (its PR-Rework rows carry `gate: rework-plan`), which runs in parallel; whichever lands second resolves the conflict at Publish by keeping that task's row changes in the `gates:` form.

## Acceptance criteria

- Every existing gated row (`Discussion-Write`, `Discussion-Burler`, `Plan-Write`, `Plan-Burler`, `Describe`, `PR-Rework`) is migrated to `gates:` and behaves as before: same re-prompt count, same findings file, same re-prompt text, same `GateOutcome.Passed`/`Attempts`/`FindingsPath`, same Done/Stuck mapping.
  The existing shuttleengine, shedadapters, burlerengine, shedrecipe and loomrecipe tests keep their assertions, with spec construction migrated to the list form and an explicit `Attempts` of 3 wherever a test relied on the removed default;
  shedrecipe's tests of the removed `gate`/`gate_attempts` keys are replaced by the `gates` tests below.
- `go build ./...` and `go test ./...` pass.
- shuttleengine tests (fake engine, scripted gate results) cover:
  - two entries, first fails then passes: the second runs only on the arrival after the first passed;
  - consecutive-failure counting with reset on pass: an entry that fails, passes, then fails again starts its budget afresh, `PassOnCap` or not;
  - a final arrival (deadline or failed re-prompt send) that stops at a failing `PassOnCap` entry, with an entry without `PassOnCap` after it, finalizes `Passed` false with that later entry reported not reached;
  - exhaustion without `PassOnCap` finalizes Done with `Passed` false and the failing entry's findings path;
  - `PassOnCap` with `Attempts` 1: one re-prompt, then let through without the closure running again, the run finalizes `Passed` true, and the entry stays let through after a later entry's re-prompt;
  - `Attempts` 0 is never called, with or without `PassOnCap`, and is reported off;
  - per-gate report states (passed, failed, let through, off, not reached) and aggregate `Attempts` as the sum;
  - an empty list is ungated (`Result.Gate` nil), and a list of only off entries yields a non-nil passing `Result.Gate`;
  - a gate closure error stays an infrastructure error, never a failed verdict.
- shedrecipe tests cover: a valid `gates` list per vocabulary name; absent key ungated; empty list, duplicate name, negative attempts, missing `name`, missing `attempts`, unknown element key, unknown name, and a leftover `gate`/`gate_attempts` key each rejected with the row named; `pass_on_cap` parsed.
- burlerengine test: the repair instruction is appended to a failing second entry's findings, not only the first.
- `loom-recipe.yaml` contains no `gate:` or `gate_attempts:` key.

## Notes for the plan writer

- Entry points: `internal/shuttleengine/gate.go`, the gated-Done branch in `internal/shuttleengine/wait.go`, `evaluateGate` and `finalize` in the same file, `internal/shedrecipe/entries_gate.go`, and each gate-capable entry's `configRejectUnknown` list.
- `shedrecipe/config.go` has no list-of-maps reader yet; `configMap` and `configStringSlice` show the existing style.
- `shedadapters/gateattempts.go` maps `GateOutcome.Attempts` onto the history; it stays unchanged if the aggregate keeps its meaning.
