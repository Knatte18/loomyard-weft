---
round: 1
focus:
  - "D2 and D7: check that `lym batten pause` really stops every mover of the task branch (row, batch commit, Parent-Align merge-in) and that `lym batten run <slug>` lifts it; the record states the behavior without a pin (test, recorded output or doc)."
  - "D2: check the claim that the live writer can be reached by `lym creel post --expect-reply <strand>` in each named row, Webster master session included, and that a paused run keeps that session alive; settle whether this is a deferred fact."
  - "D4 step 3: the orch edits the same entry's body or stored record after cancel; check the stated bound (draft clears the run lock) against `internal/boardengine/runlock.go` and `internal/boardengine/decision.go`, and that no step edits the entry while the run still holds it."
  - "D4 step 2 and Constraints: check the cancel claims (`archive_tags` lists the code tip only, partial finish, `--remove` deletes both branches) against `internal/battencli/cancel.go` and `internal/battencli/cancelcmd.go`; the carrier's records-tag pattern `archive/<slug>/` rests on them."
  - "Open risks: the re-run of the same slug (D5) depends on entry `stale-records-ref-blocks-rerun`; decide whether the pruning-fetch workaround is stated with its evidence or left for later discovery."
  - "Constraints: verify the cited `PATTERN-owner-machine`, `PATTERN-stencil-ownership` and `PATTERN-heddle-instructions` ids exist and say what the record claims, and that D4's 'owner follows the parent' reading of `internal/boardengine/owners.go` holds."
  - "Consistency across D1 to D8, Scope and Acceptance criteria: the two-orch release case (D4 step 4), the D3 trigger wording against D2's writer list, and whether any Scope or Constraints line is deliberation that belongs in the support log."
---

Seed focus for round 1. Each entry names where to look and the question to settle; none pre-states a verdict.
