# Support log — notices-from-dobby

## Interview

A darn run holds no interview; the board entry's Seen and Design sections are the whole statement, and the calls below are self-picks.

## Rejected alternatives

- Moving the state-changed notice to dobby too: it waits on the driver's stop report, the arbiter and tackler in flight, the eligible-stop hold and the watched marker, all read by the pass that acts on the stop, and its driver-not-revived follow-up keys on a revive the pass makes.
- Keeping the marker in dobby's `.lym/beam/` directory: it would survive a re-exec but not a machine switch, which the entry asks for.
- Committing the marker from dobby on each write: dobby makes no git commits today, and batten's status commit already stages the run's records.
- A shared notice-rendering package for batten and dobby: the overlap is a one-line cutter and a path-prefix strip, cheaper to keep in each than a new package and allowlist entry.
- Moving `notice_quiet_min` into beam.yaml: it would change the hub config template and its reconcile for a key the Run-Shed row already holds.

## Review rounds

_No rounds yet._

## Question ledger

- Which notices move? Self-pick: the running child's quiet, driver-dead, api-error and usage-limit, with the api-error revive (D1).
- Where does the marker live under `_lym`? Self-pick: the run's records directory in the prime, committed with batten's status (D3).
- Where does dobby keep the episode and revive streak? Self-pick: its own state record's run entry (D3).
- What prefix do batten's remaining notices keep? Self-pick: `[batten notice]` (D4).
- Where does the quiet window come from? Self-pick: the Run-Shed row's `notice_quiet_min`, read by the wiring (D2).
