```yaml
lens: fix-consistency
outcome: reported
```

Fix-consistency check of the round-1 change set: whether the changed text still agrees with its sibling decisions, criteria and cross-references, and whether any change left its twin unchanged.

Findings:

1. **LOW**: support log, question ledger Q7, which the fix left unchanged.
   - Summary: D2 now bars `%` in `label` and `confirm`, but Q7 still records the label-safety pick as refusing `#` and control characters only.
   - Evidence: D2 now reads "`label` and `confirm` hold no `#`, no `%` and no control character". Q7 still reads "Refuse `#` and control characters in `label` and `confirm`."

2. **LOW**: support log, Turn 3 lens-review entries, which the fix left unchanged.
   - Summary: Turn 3 still records D7's resolutions in the old wording, which D7 no longer holds.
   - Evidence, entry 1: "walkthrough 3, interactions 4, bound-breaker 7 (log location): fixed — `run-shell -c '#{session_path}'` (D7)". D7 now runs `run-shell -b -c '#{@lym_button_dir}'`.
   - Evidence, entry 2: "walkthrough 5 … fixed — the pinned line discards the verb's output". D7 now also requires exit 0 on every path, because discarding the output alone does not keep view mode shut.
   - The new Review rounds line points to the fixer report, but nothing in Turn 3 says these entries were superseded.

3. **LOW**: decision record D7, the failure-display bullet ("a client or worktree gone by then, as after CANCEL, only fails that message").
   - Summary: after the fix, CANCEL removes no directory that the verb uses, so "worktree gone" no longer describes a case CANCEL causes.
   - Evidence: the new D7 run-directory bullet says the run directory is the beam's anchor in a task session, "so no button process holds a task worktree as its cwd". `display-message -c <client>` does not depend on a worktree, so only "client gone" still applies after CANCEL.

Further tool calls: none. My context already held both files in their changed form.
