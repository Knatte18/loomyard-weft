---
verdict: BLOCKING
rationale: "All eleven round-1 findings are fixed in the revised record, but D8 leaves findings that reach run exit with no record-batch after them (the integration fork and Master's post-last-batch activity) without a disposition: read literally a policy finding hard-fails a done run, against D2 and D12, and a correctness finding has no named way forward per D1"
---

# Round 1 — Bouncer verdict

## Round-1 findings: all fixed

The artifacts under review already carry the round-1 fixes, and each one holds up against the report's own fix text:

- **F1** — D5 and D8 name record-batch and run exit as the two audit sites, recover-batch is gone from both, and D8 has the run-exit cross-check pass dispositioned findings.
  The must-cover `named-spawn` scenario now ends with the cross-check passing.
- **F2** — D4 states the split by location, correctness checked first and covering everything under `_lyx`.
  Master's contract files (`outcome.yaml`, `summary.md`) stay exempt, because `CheckParent` never raises `parent-write` for them, so the `_lyx` rule creates no new false positive.
- **F3** — D11 decides the history-only `goto` outcome, the `Segment`-scoped episode boundary and loom's coherence acceptance, and Constraints records the status-file history contract change.
- **F4** — D12 binds only modules with a section in `refusal-spec.md`.
- **F5** — D11 decides unrestricted targets and the `Loom-Preflight` way forward, the open risk is gone, and ledger item 17 is closed.
- **F6** — D6's digest names suspect `parent-write` paths for the recovery strand, and the remaining exposure is an open risk.
- **F7** — D10 refuses a changed card set per begun or terminal batch number.
- **F8** — D5 defines a failed verify and routes a pre-start failure to D2's grouped transient row.
- **F9** — goto writes `paused`.
- **F10** — the per-row test criterion is split into halting rows and policy rows.
- **F11** — Auto-mode assumptions point at their decisions.

The round-1 report has no `## Focus departures` section, so there is nothing to ratify.
Its "focus directive verdicts not raised as findings" are sound: the plan-writer notes name identifiers rather than the signatures or `file:line` citations the writer's stencil forbids, and the subsection is optional.

## Blocking: findings no record-batch sees reach run exit with no disposition

**Location:** `decision-record.md` D8, second paragraph; also D2 and D12.

D8 has the run-exit cross-check pass every dispositioned finding and "hard-error only on a finding no record-batch dispositioned".
Such findings are reachable in a normal done run.
Master spawns the plan-level integration fork (`webster-template-master.md`, `## verify:` bracket) only after every batch is recorded, and no record-batch runs after it.
So the integration fork's own transcript, and anything in Master's parent transcript after the last record-batch, reach `runExitAuditCrossCheck` undispositioned.

The record never says what happens to them:

- **Policy class.** Read literally, a `fabric-reference` in the integration fork, or a named spawn of that fork, hard-fails a done run.
  That is a policy guard halting, which D2 and D12 forbid, and exactly the false stop the Goal names.
  "Applies the same classification" hints at a warning but does not say so, and does not say where the warning is recorded.
- **Correctness class.** A `fork-contract-write` by the integration fork, or a Master `parent-write` after the last batch, still hard-errors at run exit.
  D1 requires that refusal to name a way forward, and none is decided: every batch is already terminal-done, so D6's `recover-batch` does not apply.
  The Plan would have to invent this disposition.

**Fix direction:** decide both cases in D8.
An undispositioned policy finding at run exit becomes a warning recorded through D5's path (state, envelope, and `summary.md`'s `## Audit warnings`) and does not fail the run.
An undispositioned correctness finding either fails the run and names its way forward, or gets a D6-style disposition.
Add one must-cover scenario for a policy finding in the integration fork's transcript that ends in a passing run exit.
