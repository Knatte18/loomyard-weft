---
round: 1
focus:
  - "D10 step 4 and Open risks: Finalize's fixer edits code under the parent lock that no perch reviews. Does the Decision state what the fixer can now let through (a test-weakening or gate-bypassing edit under the 40-line cap) and what bounds that, beyond the line count?"
  - "D4 and D10: the fixer runs with the edit directive in `finalize` mode (fix failing tests only) and `merge-in` mode (align and fix). Is each mode's permitted change stated with a bound, and does the `fixed` verdict over a passing gate (the aligned-tip advance) state what it skips?"
  - "D2: Merge-In reads the local parent tip when the push is skipped or no upstream exists, then merges after releasing the lock. Does the Decision state what that path can let through and what bounds it, given the removed lock?"
  - "D14 and D13: retired keys are dropped with a Warn and an empty `deploy_branches` deploys nothing. Does each Decision state what a stale or empty value can now skip, and what bounds it?"
  - "D11: the Go driver hands back landing halts and no resume is automatic. Check the escape and routing edges (reroute-cap prefix, landing-stop prefix, batten's per-build re-step) for an unbounded or unstated skip."
  - "Open risks, D3, D10 and D14: does any Decision, Constraint, risk or acceptance criterion leave a fact to be found out later by Plan or an implementer, for example reconcile's behavior over `profiles.yaml`, the merge record's field migration, or which readers take the merge base?"
  - "Any Decision that asks the run to change this task's own board entry body or scope, through `lyx board` or otherwise, including Scope's amendments of other entries and the Finalize board-done step; settle which of them write this task's own entry body or scope."
  - "Relocation: `decision-record.md` Scope Out entries, the D4 and D10 rationale, and the Open risks cost figure. Apply the completeness-before-leanness test first: does each carry a requirement the plan writer needs, which must be extracted into Decisions or Constraints before any deliberation moves to `support-log.md`?"
  - "Internal consistency between D4, D10, D12 and the Acceptance criteria: the aligned tip versus the merge record's reviewed-tip and merged-tip fields, a darn Finalize with no merge record, and the landing failure record's writers and removal."
---

Focus targets the rubric's attack-surface, deferred-research, own-board-entry and relocation bullets over the Decisions D2, D4, D10, D11, D13 and D14.
No entry touches a missing "Notes for the plan writer", rejected alternatives in the record or call-site enumeration.
