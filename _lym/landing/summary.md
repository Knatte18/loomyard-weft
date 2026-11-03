# Compare again before Describe when code changes after Drift-Verify

Drift-Verify compared the writer's code once, before the Code-Review perch.
A `Code-Burler-Fix` commit that rewrote code every regression case renders landed with no compare after it.
The accepted tests' recorded values then described code the pull request no longer held.

Now a mechanical `Drift-Recheck` row follows the perch in both the darn and the tapestry recipe, and compares again any code committed since the last compare.

- Both recipes now run `… → Drift-Verify → Code-Review → Drift-Recheck → Describe → …`.
  `Code-Bouncer`'s `on_done` names `Drift-Recheck`, which carries `entry_gate: drift.enabled`, `on_stuck: Drift-Verify` and `on_done: Describe`.
- `Drift-Recheck` spawns nothing.
  It is Done with no compare when no scope is recorded, the scope is off or waived, or HEAD is still the commit of the last compare.
  Otherwise it compares HEAD and records the result in the scope record, so the PR's `## Regression check` names the commit actually compared.
- With no test accepted, it compares against the baseline.
  With tests accepted, it compares against the accepted profile, leaving out only the scope's skipped tests.
  An accepted test whose output differs from the output its acceptance saw therefore shows as drift again.
- Drift-Verify captures the accepted profile when it ends Done with tests accepted.
  The capture runs the declared capture command into `.lym/drift/accepted/` at HEAD, and `scope.json` records HEAD as `accepted_profile_head`.
  `lym drift accept` and `lym drift withdraw` clear that commit, so a profile counts only for the acceptances it was captured under.
  A capture that does not pass halts Drift-Verify, naming `lym loom resume`.
- When the recheck shows drift, finds a dirty tree or an undeclared compare, or finds no compare recorded, its Stuck routes the run to Drift-Verify.
  Drift-Verify judges the drift as it does the writer's, and the perch reviews its fixes in turn.
  Drift against the accepted profile, or no profile at the last compare's commit, first lapses every acceptance, so Drift-Verify compares those tests against the baseline again.
- `Closes #N`: none; the task resolves no tracked issue.

## Design decisions

- A recheck row after the perch, not a second edge into Drift-Verify.
  Routing `Code-Bouncer` back to Drift-Verify would re-run the perch after every pass.
  The recheck costs nothing when the perch committed no code, and spends one compare when it did.
- The accepted profile is captured from the code itself, not read from the acceptance's reason text.
  An acceptance records only a free-text reason, and the compare command's output is opaque to lym.
  A profile taken once the compare passed holds each accepted test's accepted output in the form the compare command reads.
- Drift against the accepted profile lapses every acceptance, not only the changed ones.
  The compare's result is an exit code and a log, so lym cannot tell which accepted test moved.
  Under scope `full` this means the parent rules again on whatever the Drift-Verify agent re-accepts.
- The recheck's Stuck is a route, not a halt: it hands every question it cannot settle to Drift-Verify, which owns the halts and the ways forward.
  The shed's default bounce budget bounds the loop.
- `lym drift compare` and the other verbs still run at Drift-Verify only.
  The run now compares by itself after a late code change, and a routed-back run sits at Drift-Verify, where the verbs work.

## To check by hand

- Not run: a darn or tapestry run on a real hub with `drift.enabled`, where a review fixer changes covered code after an acceptance.
  Watch that `Drift-Recheck` routes back, that the lapsed tests show as drift, and that the PR's regression check names the final commit.
- The accepted profile is ephemeral under `.lym/drift/`, but its commit is recorded in the durable `scope.json`.
  On an adopting machine the directory is missing, so the recheck's compare reads an empty profile, shows drift and lapses the acceptances.
  Confirm a fresh judgment at Drift-Verify is the right outcome there.
- An in-flight run already inside the perch exits to `Drift-Recheck` instead of `Describe`.
  A run past the perch keeps its position and is not rechecked.
- Each recheck after a code change spends a whole compare run, and each Drift-Verify Done with acceptances a whole capture run.
  Confirm the cost is acceptable for hubs whose capture command is slow.
