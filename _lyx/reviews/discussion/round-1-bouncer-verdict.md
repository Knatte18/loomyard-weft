---
verdict: APPROVED
rationale: "Round-1 review approved with six MEDIUM, one LOW and three NIT findings; the current decision record resolves all ten (D6, D9, D14 bounds; D18 empty allowlist with admission rule; D19 tmuxkit import ban; hub-key socket exclusion; D20 and Rationale narrative moved to support-log items 10-11; Notes trimmed; D4 reworded), and no new attack-surface gap remains."
---

The round-1 review returned `APPROVED` with ten findings, and the fixer report claims every one fixed.
I checked each claim against the current `decision-record.md` and `support-log.md` rather than the report.

- **F1 (D6):** D6 now names a `scankit` scan that every production package declaring `ConfigTemplate` is registered in `Modules()`, with a Bound line; the `configreg` acceptance criterion carries a planted-removal proof.
- **F2 (D9):** D9's Bound line states that `MaxBounces` and `Segment` are pinned only as pair-equal, except Discussion-Bouncer's budget via `resume_test.go`, and names the tests that keep `OnDone` order and the PR edges guarded.
- **F3 (D14, D20):** D14's Bound line makes deletion case by case, names the three `PrintStatusLinesOnChange` cases that move to `shedverbs` first, and D20 no longer claims `step_test.go` keeps every deleted assertion; the acceptance criterion requires every deleted case to be asserted in `shedverbs`.
- **F4 (D18):** the allowlist starts empty, admits only a package whose `TestMain` cannot call the entry point, and rejects "does not start tmux" as a reason.
- **F5 (D19):** D19, its Bound line and the Constraints line bar other kits from importing `tmuxkit`, and the acceptance criteria carry a planted-import proof.
- **F6 (Acceptance criteria):** a new socket is excluded only when its key equals `reedengine.ServerName` of an operator hub path; any other new socket fails, live or dead.
- **F7 (D20):** D20 keeps the list of unchanged invariants as the constraint; the per-invariant justifications sit in support-log Interview item 10.
- **F8 (Rationale lines):** the churn statistics are gone from the record and sit in support-log Interview item 11; D10 keeps only the second-writer clause, which backs its Bound.
- **F9 (Notes):** the reed `-L` and `validate.go` notes are gone; `internal/reedengine` is folded into D17.
- **F10 (D4):** D4 reads "the `go env GOMOD` execs are removed".

The relocations passed the completeness-before-leanness test: nothing moved to the support log carried a requirement the plan writer needs.

My own pass over the attack-surface question found every new escape hatch or downgraded guard bounded: D2 (shared harness), D5 (derived listing), D6, D7 (unlisted constructor), D9, D10 (second writer), D12 (dropped claims), D13, D14, D17 (sweep scope, unset `TMUX`), D18 (allowlist, over-approximation) and D19 (`os/exec` exemption).
The Rejected alternatives are in the support log, not the record, as the contract requires.
The review report has no `## Focus departures` section, so there is nothing to ratify.
