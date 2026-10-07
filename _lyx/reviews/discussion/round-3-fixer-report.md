# Round 3 fixer report

Every finding of `round-3-review.md` is fixed in `_lyx/discussion/decision-record.md`, and one question-ledger entry is added to `_lyx/discussion/support-log.md`.
No code, test or git command was involved.

## Changes

- **F1 (MEDIUM):** Decision 5's bound now states that in the prime a rejected parent-branch push returns the error envelope after the records side was pushed, rewrites nothing local, and names the rejection row of Decision 2 as the way forward; `sync` is unaffected.
  A new Open risks entry and an extended `lyx fabric push` divergence criterion cover it, and `support-log.md` gains Q13.
- **F2 (MEDIUM):** Decision 1 scopes the side-naming sentence to push failures and says a lock-wait expiry returns an error naming the push lock and no side.
  Decision 2 puts an expiry under the other-failure row, and a new criterion covers `commit-records` against a lock held past the bound.
- **F3 (LOW):** Decision 2 makes the envelope's fixed text side-neutral, leaves side names to the appended entry error, and says the row is picked with `errors.Is(err, gitrepo.ErrPushRejected)` or a side-neutral `fabricengine` predicate.
  It also says which row applies when the two sides fail with different kinds.
- **F4 (LOW):** Decision 2 names both rows as class transient.
- **F5 (LOW):** The Constraints entry cites the pattern's denied list instead of enumerating two forms.
- **F6 (LOW):** The one-side acceptance criterion now covers either side, asserts `errors.Is` for a rejected side, and ties the envelope's records-side claim to what is true.
- **F7 (NIT):** Decision 1 points at Decisions 3 and 5, and the repeated Open risks entry is folded into the hung-holder entry.

## Deferred

Nothing deferred.
