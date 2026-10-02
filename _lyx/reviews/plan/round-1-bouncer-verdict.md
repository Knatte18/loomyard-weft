---
verdict: APPROVED
rationale: "All seven round-1 findings (F1-F7) are fixed in the current cards, each Decision 1-10 and every Constraint is carried, Card 1's Edit list covers every file that reads or builds a GateSpec field, Card 2 covers all six gated loom-recipe rows, and verify runs ./... with and without -tags integration"
---

The plan under judgment is `_lyx/plan/00-overview.md`, `01-shuttle-gate-list.md` and `02-recipe-gates-key.md` as they stand after the round-1 fixer pass.
The round-1 report returned APPROVED with seven non-blocking findings, and the fixer report claims all seven fixed.
I checked each fix against the cards:

- **F1** — Card 1's Callers paragraph maps an absent, zero or negative `gate_attempts` to `Attempts` 3, names the changed `AbsentGateAttemptsYieldsPackageDefault` assertion, and moves the ungated subtests to an empty-list check.
- **F2** — the count slices are sized from `len(run.gate)` on first use, and `internal/shuttleengine/attach.go` is under Uses.
- **F3** — `resume_test.go` and `smoke_gate_test.go` are in Card 2's Edit list with their comment changes named, and the `gatequiescence_test.go` sentence covers its comments and failure messages.
- **F4** — the doc-comment sweep covers every comment naming `gate.Gate`, `opts.Gate.Gate` or a nil `Gate`, naming the `engine.go` and `singlellm.go` sites.
- **F5** — both ImpactSummary lines state a concrete blast radius.
  Card 2's claim holds on the tree: outside `shedrecipe`, only `gatequiescence_test.go` reads the old keys.
- **F6** — `resolveGateSpec` wraps helper errors for a `gates` element with the entry and index, and the leftover-key test asserts the unrecognized-key error names the old key.
- **F7** — a Shared Decisions line records Told-Geometry, Transient Stop and Refusal Way-Forward.

I also checked the plan against the tree myself:

- Every file that reads `.Gate`/`.Attempts` off a `GateSpec`, builds `GateSpec{Gate: ...}`, or touches `gateSent`/`defaultGateAttempts` is in Card 1's Edit list.
  The remaining `GateSpec` mentions compile unchanged under a slice type: `GateSpec{}` in `attach.go` and `wait_test.go`, pass-through parameters in `webstercli`/`loomcli`, and `GateOutcome` aggregate reads in `shedadapters/burler.go` and `gateattempts.go`.
- Card 1's Wait rule finalizes once a failure count has reached `Attempts`.
  That matches today's `run.gateSent >= run.gate.attempts()` at `wait.go:266`, so a single-entry row still gets three re-prompts and then finalizes.
- `loom-recipe.yaml` has the six gated rows Card 2 names, and no other row.
- `smoke_gate_test.go` is `//go:build smoke`, so `go vet -tags smoke ./...` compiles it, which the rubric accepts.
  Every targeted package runs under `go test ./...` and `-tags integration`.

The round-1 report's focus directive verdicts, including the granularity ruling on Card 1, are ratified: the `GateSpec` struct-to-slice change breaks every reader at once.
The report has no `## Focus departures` section.

One non-blocking observation that I did not record as a finding: the Webster-Burler row comment at `contracts/recipes/loom-recipe.yaml:268` says "No gate key, deliberately".
Card 2's sentence on the "header and row comments describing the gate" covers this comment, and the implementer should reword it to the `gates` key.
