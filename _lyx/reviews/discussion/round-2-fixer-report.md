# Round 2 fixer report

All ten findings fixed in `_lyx/discussion/decision-record.md`; `lyx loom validate-discussion` exits 0.
`support-log.md` is unchanged.

## Changed

- F1, F2 (D1): the box is read after a provider-owned settle following each Enter; the match is the first 40 normalized characters of the sent text (all of it when shorter), normalized as `sendVerified` normalizes its needle; the collapsed-placeholder residual is the new R5.
- F3 (D9, R4): D9 names A2 and the board entry as its premise and says it supersedes the `doc.go` statement; the confirm-first clause is gone from R4 (formerly R5), so the skip is final.
- F4 (D8, acceptance): a boundary with one later turn end the watcher has not yet read, or none, types nothing and leaves the baseline; one acceptance line added.
- F5 (D12, Constraints): D12 says no standalone verb is added because of the no-new-verbs rule and that the re-prompt carries the same parse error; the Constraint and the Rationale no longer claim the pattern is met, and the Constraint names the verb half as not delivered.
- F6 (D5): removals are logged.
- F7 (D14): names `PATTERN-test-isolation` and `PATTERN-testkit`.
- F8 (D6, acceptance): the bound and the criterion cover a driver that died before recording the voucher, mid-producer or right after a finished step.
- F9: the Constraint bullet on the prune ordering is deleted, and the old R4 with it; R5 is renumbered R4.
- F10 (D8): the rationale no longer says the session re-read what it needed.

## Deferred

None.

## Note

F5 is the one place a pattern's verb obligation and the board's no-new-verbs rule pull apart.
I recorded the departure in the Constraints instead of adding a verb; the operator may want to settle whether the pattern should grow a verb for the review-parse gate.
