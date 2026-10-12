---
round: 4
focus:
  - "D10 in `_lyx/discussion/decision-record.md`: for a landing mapped by the time join to a run whose pair is not yet torn down and which has no archive tag, does the decision give a source for the `hand` test, for the run's identity and for the `land-incomplete` flag, given that D4 defines a run's loom records and batten span only from its archive tag and Scope Out says a run counts only once its tag exists? Settle whether the checkpoint commit written by Finalize (`internal/landingshed/finalize.go`) makes such a run never `hand`."
  - "D10's time-join window in `_lyx/discussion/decision-record.md`: when a slug is reused (one pair per run) or Finalize's retry path commits a second `loom: run records checkpoint for landing <slug>` commit, does the decision say which checkpoint opens the window, which board done commit closes it and which commit date every comparison reads?"
  - "D10 and D11 in `_lyx/discussion/decision-record.md` with `landing.yaml` `squash: false` (`internal/landingshed/config.go`): the parent-side merge after Finalize's catch-up merge-in is a fast-forward, so every task commit is a first-parent commit on the parent branch. Does the decision count one landing and avoid blaming the other task commits into the `unmapped` share?"
  - "The acceptance criteria in `_lyx/discussion/decision-record.md`: does every classification rule across D4 to D11 that carries a requirement have a criterion a test can fail on, and is a failure episode of Land or Worktree-Teardown defined so that one episode is told from two? Walk all rules, not only the generic wake kind, keep-alive pings, Worktree-Teardown failure wakes and per-point alignment outcomes."
  - "The window-exclusion criterion near line 201 of `_lyx/discussion/decision-record.md` against D10's rule that the time join takes the first first-parent commit dated in the window and excludes only commits already claimed by the exact join: do the two state the same outcome for an orch hand landing or direct push in the window?"
---

Round 4 focus targets the D10 landing join, its window and fast-forward behaviour, and the completeness of the acceptance criteria.
