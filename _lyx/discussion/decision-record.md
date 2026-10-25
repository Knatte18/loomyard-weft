# Decision record — adopt-runs-tests

## Goal

A `lyx adopt` that takes something over starts the hub's full test suite on the adopting machine, so a break that a machine change brings (Ubuntu 26.04, Ubuntu 24.04, Windows 11) shows at adopt time, not inside the next run's gate.

## Scope

- `internal/adoptcli`: a new last step that starts the suite, a new `adoptDeps` seam for it, its hub implementation, and the envelope fields that report it.
- `internal/lyxdirs`: one new directory-name token for the suite's log.
- The `lyx adopt` long help and the `internal/adoptcli` package doc describe the new step.
- Out of scope: a CI job, a PATTERN entry for an Ubuntu floor, any change to `lyx gate test` itself, and a creel notice for a red suite.

## Decisions

### The full suite is the `finalize` gate key

Decision: the suite is `lyx gate test --gate finalize --wait`, the tiers and scope the hub's `gates.finalize` key holds.
Rationale: the key is the hub's own statement of the whole-tree check that guards a landing, it already leaves out the manual tier, and a hub that retunes it retunes the adopt suite with it.
A tier the green record already holds green for the clean tree is carried and runs nothing, as for any gate call.

### One suite per `lyx adopt` call, in the prime

Decision: an adoption session is one `lyx adopt` call.
The call starts at most one suite, in the hub's prime worktree, after the dobbies, and only when it recorded this machine as the owner of at least one entry or of the prime.
Rationale: the prime holds the landed code; adopting the prime and two beams in one call starts one suite, and a call that records no owner took nothing over.
Bound: two separate adopt calls each start a suite; the gate slots serialize them, and the second carries whatever tier the first left green for the same tree only when it starts after the first finished.

### A red suite only reports

Decision: the suite never refuses the adopt or undoes any step.
Rationale: the suite runs after the owner write and the resumes, which have already taken effect, and the operator decided that what fails is fixed on the machine that shows the break.

### The suite runs detached, and the adopt returns at once

Decision: adopt spawns the suite as a detached process through `proc.Detach`, its output written to a log in the hub's board-level `.lyx` directory under a new `lyxdirs` token, truncated at each start, and returns without waiting.
The envelope carries a `suite` field with the pid and the log path; a spawn that fails is a `suite_warning` naming `lyx gate test --gate finalize` in the prime as the way forward, and stops nothing else.
Rationale: the full suite runs for many minutes, and the adopt's own steps must not wait on it.
Bound: nothing reads the log for the operator; the operator reads it, and the gate's own run records hold each tier's result.

## Constraints

- PATTERN-spawn-observability: the spawn is logged through `internal/logger`.
- PATTERN-durable-vs-ephemeral-state: the log is never-tracked, under `.lyx`, named only through `internal/lyxdirs`.
- PATTERN-gate-slot: the suite runs through `lyx gate test`, which holds the gate slots itself.
- A test binary never re-executes itself: the hub seam starts nothing under `go test`, as the other detached spawns do.
- PATTERN-test-speed: tests fake the seam in the existing `adopt` table and stay at the existing tiers.

## Auto-mode assumptions

- No interview is held; the four open questions of the board entry are answered by the decisions above as self-picks.
- The prime worktree's location is the one `battencli.ResolveBattenPrime` returns, which adopt already uses.

## Open risks

- A suite red because of a flaky test reports a break the machine change did not cause; the operator reruns the gate.
- Two adopt calls close together run the suite twice.

## Acceptance criteria

- An adopt that records an owner calls the suite seam once, after the dobbies, however many entries it adopted, and the envelope carries the seam's pid and log.
- An adopt that records no owner, or that is refused, never calls the seam.
- A failing seam leaves every slug's outcome as it was and sets `suite_warning` naming the way forward.
- The hub seam starts nothing under `go test`.
