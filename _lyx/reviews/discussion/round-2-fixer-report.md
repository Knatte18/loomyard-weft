# Fixer report — discussion round 2

## Changes

- F1 (MEDIUM, design): fixed differently.
  The premise holds: in the `ErrConfigAbsent` message at `internal/gatecli/run.go:95`, the clause for a session that is not the prime is the only place it appears, and D3 asked for none.
  D3 now says every gate load failure message names the step for a session that is not the prime, keyed on the session's role: "a session that is not the prime takes none of these steps; it reports status: FAILED and the orch runs them".
  D3's Bound now says this clause is what keeps a pair from taking the removal step.
  The acceptance criterion for a machine file at `go_parallel: 0` now requires the clause in the message.
  Proposed change not made: reusing the `run.go:95` wording verbatim ("a session lyx refuses the verb from reports status: FAILED and the orch runs it").
  That wording only applies when lyx refuses the verb, and lyx refuses a pair no literal file removal, as D3's Bound already states.
  So the verbatim clause would not have covered the removal step the finding is about.

## Self-check

- fix-proposal, MEDIUM: the verbatim run.go clause does not cover removal, which lyx never refuses. Fixed differently from its own suggestion only in wording: the clause is now keyed on the session's role, as it proposed.
- fix-claims, LOW: same defect as fix-proposal, merged into it. Fixed by the same change.
- fix-way-forward, MEDIUM: same defect as fix-proposal, merged into it. Fixed by the same change, and the Bound now says the clause covers removal because it names the steps by role.
- fix-consistency, LOW: next to the new clause, the hub-file-only message's "way forward only" left unclear whether that message also carries the clause. Fixed as proposed: D3 now puts the clause in every gate load failure message, below the hub-file-only sentence.

## Disputed

None.

## Deferred

None.
