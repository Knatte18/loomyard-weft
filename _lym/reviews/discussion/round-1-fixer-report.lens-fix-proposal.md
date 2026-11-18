```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each review proposal the fixer carried out as proposed (fix-proposal lens).

- **MEDIUM**: D4, the gate keys `harden-live` and `harden-live-copies` "ship in the gate template the way `code-burler-fix` is, with tiers `[0]`... lym's own hub sets both to `[2]`" (from F2).
  On a hub whose `gate.yaml` does not set either key, the `live` gate is green while running no live test, and nothing catches it.
  Hub gate keys merge per key over the template: the hub's `gates` map leaves out `card` and `pr-review`, yet both still resolve.
  So a hub that never wrote `harden-live` runs the template's `[0]`.
  A tier-0 run skips every tier-2 test (they call `tiergate.Require`), so the `live` gate passes with no tmux test run and no tmuxkit stray check.
  CONVERGED then reads a "green live gate" that drove nothing.
  The record names no writer for the hub's `[2]` setting, and Preflight does not refuse a live key whose tiers sit only at tier 0.
  The fix could have Preflight refuse a `harden-live` key holding no tier above 0, with the `lym config gate --set` write as its way forward.
  Evidence: `/home/knatte/Code/loomyard-LYMHUB/_bolt/_lym/config/gate.yaml` (the `gates` map has no `card` or `pr-review`); `internal/gateslot/template.yaml:29` (`card` and `pr-review` keys); D2's Preflight checks only `packages` and `reviewer`.

- **MEDIUM**: D9 `hard_max_bounces` "default 10" with D3 "expands ... cyclically to its hard cap" (from F6).
  A task's `run_settings` may override any key (D9), so `max_bounces: 12` with the default hard cap of 10 is accepted.
  The expanded list is then 10 entries long, and rounds 11 and 12 clamp to entry 10 (`pickRound`).
  Adjacent rounds then share a reviewer, which is the exact case F6 meant to close.
  Nothing in the record bounds `max_bounces` to at most `hard_max_bounces`, and Preflight's new refusal covers only the list's shape.
  Evidence: D9 bullet "A task's `run_settings` may override any of them"; `internal/burlerengine/models.go:30` clamp.

- **MEDIUM**: D4's soft exhaustion, "When they run out, the gate ends the row instead of halting the run" (from F7).
  The next round's `verify` gate can halt on the same deferred cause anyway.
  `harden-burler-fix` is "shipped the way `code-burler-fix` is", and lym's hub runs that key at tiers `[0, 1, 2]` over the diff.
  In round n+1, a diff touching a package whose tier-2 test the round-n live gate left red fails `verify`.
  `verify` exhaustion is not softened, so the run halts on the cause D4 meant to carry forward.
  Separately, "each failing test is recorded as a deferred finding" names no file or writer.
  D6's facts read the deferred findings the fixer reports, and the gate is not the fixer.
  Evidence: hub `gate.yaml` `code-burler-fix: {tiers: [0, 1, 2], scope: diff}`; D3 "`verify` gate runs ... `harden-burler-fix`, shipped the way `code-burler-fix` is"; D6 "lists the findings the fixer left deferred".

- **LOW**: D5's reap ends recorded processes "through `runproc.End` over every owner that hub's records name" (from F3).
  The records are written by the sandbox's `.dev-bin` lym, which is the binary under test, while the reap runs the run's own lym.
  A round that changes runproc's record format or location leaves processes that `End` cannot read.
  They fall to the `/proc` kill as "no record names" and are reported as orphans.
  This is the same kind of bound D5 already states for the socket-key derivation, but it is not stated here.
  Evidence: D5 Bounds "The reaps derive socket keys with the running lym's derivation; a round that changes that derivation can leave a server only the next round's reviewer will see"; Constraints "the sandbox's `.dev-bin` lym, which is the binary under test".

Proposals challenged with no finding: F1 (a bare `efforts` key with no `/` is never checked as a role, so stripping the role keys is complete), F4, F5, F8, F9, F12, F13, F14.

Tool call: read the rest of the hub `gate.yaml` `gates` map, the gateslot merge doc and agentconfig `checkEfforts`, because my context lacked the hub's gate keys past line 60 and how efforts keys are checked.
