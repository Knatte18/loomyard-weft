Two LOW findings, both on `contracts/stencils/orch/orch-template-usage-limit-wake.md:4`.

1. LOW, final clause "and restart what stopped".
   The role says a halted or paused run is resumed only on the operator's word or after judging the stop (`contracts/orchrole/orch-role.md:47`), so the line still tells the orch to restart on its own.
   The clause also follows the "only for a stop a notice names" semicolon, so it can be read as restarting every stopped run, notice or not.

2. LOW, "wait for the notices of the runs that stopped".
   The wording leaves no fallback when no notice arrives for a stopped run.
   This follows the role's own notices-only rule; the lens calls it a soft concern, not a defect.

Checked, no findings: F3 matches the "never `cd`" rule and no test pins the old attach wording; F4 fails if `orch-template-role` is registered again; the wake line stays one line with `{{.reset}}` as its only marker.

Tool calls: one, a grep for `reed attach`, `check each run`, `restart what stopped` and `never for you to run` across `contracts` and `internal`, because the context lacked any search for other readers or pins of the old wording.
