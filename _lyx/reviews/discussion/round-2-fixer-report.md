# Fixer report — discussion round 2

All three findings validated against `contracts/specs/refusal-spec.md` and D2, and fixed.

## Changed

- **F1 (BLOCKING), reversal-failure halt has no admissible way forward.**
  Confirmed: "What a way forward is" admits a `lyx` verb, a self-taken transition or a plain `git` command, and the `status: FAILED and the orch runs it` clause covers only a role-guarded `lyx fabric` verb.
  D2 now has the verbs write a never-tracked renumber journal under `.lyx` (path declared in `internal/lyxdirs`) holding the pre-call bytes of every file they rewrite, rename or delete and the names of files they create, before the first apply step.
  A reversal failure or a killed process leaves the journal; both verbs refuse while it exists, and that refusal and the reversal-failure halt name a new `lyx loom restore-renumber` verb.
  Its bounds are stated in D2: same lease, refusal with no journal, writes only top-level card and overview base names, idempotent re-run on I/O failure, and the stated cost that it discards edits since the interrupted call.
  Scope, D3, Auto-mode assumptions and Acceptance criteria now name the third verb; the support log records the rejected alternatives (hand reversal, refusing a dirty plan directory, implicit restore) and ledger Q10.
- **F2 (MEDIUM), reversal source and apply order underspecified.**
  D2 now says the verbs read pre-call bytes before applying and that the reversal restores from the journal, removes created files and then the journal.
  The apply order places `remove-card`'s deletion before the ascending moves and `insert-card`'s new-file write after the descending moves, the Card Index rewrite last and the journal removal as the final step.
  The acceptance criterion injects a failure at each apply step of each verb.
- **F3 (MEDIUM), mid-apply I/O failure outside the loom raw-I/O row.**
  Confirmed: the row at `contracts/specs/refusal-spec.md:314` names status, seed, lock, park marker and records files only.
  D2 and Scope now widen that row to name `rename` and the plan directory's card and overview files, and the refusal acceptance criterion checks the widened row.

## Disputed

None.

## Deferred

None.
