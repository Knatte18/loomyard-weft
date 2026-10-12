# Card 9 — arbiter-eligibility

**Create:**
- `internal/loomcli/arbitereligibility.go`
- `internal/loomcli/arbitereligibility_test.go`
- `plan:internal/loomcli#arbiterStop` -> `type arbiterStop struct`
- `plan:internal/loomcli#arbiterEligibility` -> `func arbiterEligibility(stop arbiterStop) error`

**Edit:**
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Decide D2's closed list of eligible stops in one pure function, so the verb's refusals are testable without a run.

`arbiterStop` carries what Go reads off the status file and the stop envelope: `State`, `Producer`, `Kind`, `Transient`, `InterruptPolicy`, `EscalationPending` (from `shedadapters.PendingEscalation` at a Bouncer row) and `PublishHaltNamesTarget` (a `blocked` stop at the Publish row whose verify-failure record names a target).
`arbiterEligibility` returns nil for:

- an error stop (non-empty `Kind`) of kind `producer` or `bootstrap`, of kind `interrupted` under the `reinvoke` policy, or with a non-empty `Transient`, which reaching the driver means it failed twice;
- a halted `blocked` stop, unless `PublishHaltNamesTarget`;
- a halted `awaiting` stop whose `EscalationPending` is true.

Every other stop is refused with a way forward naming today's hand-back: an error of kind `busy`, `ownership` or `unseeded`, an error envelope with no kind, an `interrupted` stop under `handback`, the target-naming Publish halt (batten's retry owns the first failure and the parent the second, and the verb cannot tell a watched run from an unwatched one, so it never reads the watched marker), an `awaiting` stop with no pending escalation (PR-Gate, a `parent_notice` with no Bouncer escalation), and a `paused`, `done` or `running` state.
Whether the repair fork escalated is not in the status or the envelope; the driver stencil (card 14) bounds that half.
The function reads no seed driver (`PATTERN-driver-choice-single-site`).

`arbitereligibility_test.go` holds one table-driven test with a row per eligible kind and per refused kind, the Publish halt with and without a named target and the `awaiting` stop with a `parent_notice` but no escalation included, asserting nil or the refusal's way forward.
Raise `internal/loomcli`'s row in the test budget by one.

Add loom-section rows to `contracts/specs/refusal-spec.md` under `arbiter: stop not eligible`, one per refused class (error kind, interrupted under handback, Publish halt naming a target, awaiting without an escalation, paused, done or running), each a correctness halt whose way forward is the hand-back to the run's parent.

**ImpactSummary:** a new pure function with no caller until card 10; refusal-spec gains rows.
