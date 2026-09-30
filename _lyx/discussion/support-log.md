# Support log — status-strand-by-driver

## Interview

Autonomous session: no operator answered.
Every turn below is a question the agent posed to itself, with the pick it made after reading the code.

1. **Is gating `start` enough?**
   The board names only `start`'s call to `ensureStatusStrand`.
   Reading the code showed a second caller, loom's `PreStep` hook, which both `lyx loom step` and `lyx shed step` reach.
   ly-drive drives a run through `lyx shed step`, so an llm run would get its status strand back on the first step.
   Pick: `step` must stop adding the status strand too.
2. **How does `step` decide, given it has no driver?**
   The Driver Choice Single-Site Invariant confines the driver read to the bootstrap verb, and `loomPreStep` documents that it discards the driver on purpose.
   The go driver runs detached `lyx loom run` and never steps.
   Pick: `step` does no strand work at all, driver-blind, and keeps only reed `Up`.
3. **Where does the stale-strand removal live?**
   The board says a resume of an llm run removes a leftover status strand.
   Resume is `lyx loom start`.
   Pick: `start`'s llm arm removes it; `step` does not.
4. **Does the invariant permit a driver branch that removes a strand?**
   The current text says the branch "selects a spawn and nothing else".
   Removing the status strand is choosing the session's strand set, the same kind of decision, not behaviour gating.
   Pick: amend the invariant wording in the same commit rather than add a carve-out.
5. **Where must reed `Up` go?**
   `ensureStatusStrand` owns the `Up` call today; the llm arm still needs a live session for its driver strand, and `step` needs one for its producers' agent panes.
   Pick: `Up` moves out of the helper to run unconditionally before the branch.
6. **Removal failure — fatal or not?**
   Pick: warn and continue, matching today's failed-replace stance.
7. **Sidecar on removal?**
   Pick: delete it best-effort, so the scratch tree does not name a strand that is gone.
8. **Pane height.**
   The board states `collapsed_rows` already carries the ly-drive height in hub config.
   Pick: no code or config change in this task.

## Rejected alternatives

- **Gate only `start`, as the board literally describes.**
  Rejected: ly-drive's `lyx shed step` re-adds the status strand on every step through `loomPreStep`, so the llm session would still show both surfaces.
- **Have `step` read the seed's driver and gate the status strand on it.**
  Rejected: it adds a second driver read outside the bootstrap verb, against the Driver Choice Single-Site Invariant, and contradicts `step`'s documented choice to discard the driver.
  It buys nothing, since the go driver never steps.
- **Have `step` remove any status strand unconditionally.**
  Rejected: an operator stepping a halted go run by hand would lose that run's status band.
- **Add a `driverFieldReadCarveOuts` entry or a new invariant carve-out.**
  Rejected: the read stays in `internal/loomcli`, which the tripwire already admits; only the invariant's prose needs to widen from "spawn" to "driving surface".
- **Make status-strand removal fatal to `start`.**
  Rejected: a leftover band is cosmetic; failing the resume over it would block the run.
- **Leave the sidecar after removal.**
  Rejected, narrowly: harmless to `resolveStatusStrandAction` (it adds when no strand carries the name), but a file recording a GUID for a removed strand is misleading scratch state.
- **Symmetric cleanup: `start` on a go run removes a stray ly-drive strand.**
  Rejected under YAGNI: seeds are immutable per run, so a go run never had a ly-drive strand; no reported case needs it.
- **A new config key for the ly-drive pane height.**
  Rejected: the board rules it out, and `collapsed_rows` already covers it once the status strand is gone.

## Review rounds

_No rounds yet._

## Question ledger

- **Q1 — Does gating `start` alone meet the goal?** Resolved (auto-pick): no; `step` must stop adding the strand too. Evidence: `lyx shed step` for the loom recipe arms loom's own `PreStep`, which calls `ensureStatusStrand`.
- **Q2 — Should `step` read the driver?** Resolved (auto-pick): no; `step` stays driver-blind and does no strand work.
- **Q3 — Where does leftover-strand removal happen?** Resolved (auto-pick): `start`'s llm arm only.
- **Q4 — Invariant amendment or carve-out?** Resolved (auto-pick): amend the Driver Choice Single-Site Invariant's wording to "driving surface".
- **Q5 — Where does reed `Up` run?** Resolved (auto-pick): unconditionally in both `start` and `step`, outside the status-strand helper.
- **Q6 — Removal failure handling?** Resolved (auto-pick): warn and continue.
- **Q7 — Status sidecar on removal?** Resolved (auto-pick): delete best-effort.
- **Q8 — Pane height work?** Resolved (auto-pick): none; hub `collapsed_rows` already set per the board.
- **Q9 — Remove every same-named status strand or the first only?** Resolved (auto-pick): every one, since an older build may have left duplicates.
