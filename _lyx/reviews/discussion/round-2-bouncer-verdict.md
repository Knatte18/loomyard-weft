---
verdict: APPROVED
rationale: "All four round-2 findings are fixed in the artifacts as judged: D8 now dispositions run-exit-only findings (policy warns and keeps the outcome, correctness demotes done to stuck naming suspect paths with a git-revert-then-re-step way forward) with a must-cover scenario for each; D5 adds a run-level warning list and lyx webster run's envelope; ledger item 18 is resolved; the refusal-kind rule sits once in Constraints without a tally. No new material gap."
---

# Round 2 verdict

## Round-2 findings against the current artifacts

- **F1 (BLOCKING) — fixed.**
  D8's second paragraph now splits undispositioned run-exit findings by class.
  A policy finding becomes a recorded warning and the run keeps its outcome, which honours D2 and D12's no-halt rule for policy guards.
  A correctness finding demotes Master's `done` to `stuck`, modelled on the integration stage's existing regression demotion (`internal/websterengine/runlevel.go`, the `stuckReason != "" && runResult.Outcome == outcomeDone` branch).
  Its way forward is a `git` revert on the warp followed by a re-step of the Webster row, a D1-valid way forward.
  The rationale explains why a bare re-step would not be enough.
  The Acceptance criteria add both must-cover scenarios: an integration-fork `fabric-reference` ending `done` with one run-level warning, and a post-last-batch Master `parent-write` ending `stuck`, then `done` after revert and re-step.
- **F2 (MEDIUM) — fixed.**
  D5's recorded-warning bullets now cover both audit sites: a run-level list in `state.json` beside the per-batch ones under the same once-per-identity rule, the raising verb's envelope (record-batch's or `lyx webster run`'s), and the shared `## Audit warnings` section.
- **F3 (LOW) — fixed.**
  Support-log ledger item 18 is resolved from `manifest/roadmap.md`, and nothing was pushed into the decision record, as the fix asked.
- **F4 (NIT) — fixed.**
  Scope Out no longer carries the refusal-kind bullet.
  Constraints states the rule once without the "five values" tally, and D11 keeps its property sentence.

## Focus departures

The round-2 report declares none, so there is nothing to ratify or reject.
Both round-2 focus entries were addressed through F1 and F2.

## Independent pass

I re-read both files for new gaps, relocation candidates and rubric exclusions.
None rises to a finding:

- The decision record carries decisions, constraints and short rationales.
  No deliberation narrative needs to move to the support log, and no support-log content carries a requirement the plan writer is missing.
- The Notes for the plan writer are optional and correctly marked unverified.
  Call-site enumeration is excluded by the rubric and by the writer stencil's rule against gathering exhaustive pattern research.

One non-blocking observation, not raised as a finding:
support-log ledger item 19 still says the run-exit cross-check "hard-errors only on undispositioned ones", which D8 as revised no longer does (item 22 carries the current answer).
The support log is never read by `Plan-Write`, and D8 itself is unambiguous, so the stale wording cannot mislead the plan.
