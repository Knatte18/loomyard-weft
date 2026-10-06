# Round 1 fixer report

All findings fixed in `_lyx/discussion/decision-record.md`; `support-log.md` unchanged.

## Changed

- F1 (D7): the remote update now runs before fabric's local reset, so a failed update changes nothing and a failed local reset leaves the remote at an ancestor of the local HEAD, where a re-run converges.
  The refusal compares the remote tip against the local HEAD, and its way forward (merge `origin/<task-branch>`, re-run) covers the already-diverged #404 state.
  The Rationale's guard sentence and the acceptance criterion follow.
- F2 (D7): the lease is the exact tip the ancestry check was run against; a remote that moved after the fetch makes the update refuse.
  The Bound and the acceptance criterion say so.
- F3 (D6): `stuck` joins `failed` and `dead` in the Decision, Rationale, Bound and the acceptance criterion.
- F4 (D3): the Bound now states what the relaxation lets through, what bounds it (the advisory and `record-batch`'s `delete-not-done` check), and how the Rework-Plan-Gate treats a removed target.
- F5 (D3, D4): D3 uses D1 and D2's "not-yet-completed" scope; D4 states its check runs over the batch's own begun cards against later not-completed cards.
- F6 (D6): the Rationale states that where `recover-batch` refuses a failed batch with uncheckable paths, the restamp does not make it re-drivable and `recover-batch`'s refusal names the way forward.
- F7 (D9, Auto-mode assumptions): D9 and the assumption say the note omits the what-ends-it clause, with no guess, when the plan establishes nothing.

## Deferred

None.
