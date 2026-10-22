# Fixer report — discussion, round 3

## Changes

- F1 (fixed as proposed): in D7's answer-key instruction, a finding for a named card now holds only while the entry's change is uncarried for that card, and carried is defined: the card carries it, or, for a named card of a done batch, a card inserted with `lyx loom insert-card` after the last begun batch carries it.
  D7 now requires a decision to name each card by slug as well as number, because `insert-card` renumbers later cards, and D8's named-cards sentence carries the slug rule.
  Added the acceptance criterion that an inserted card satisfies an entry naming a done card, and the next round raises no finding for that done card.
- F2 (fixed as proposed): D9 step 3 now resolves each member-glyph target of the live cards to its file through `internal/planglyph` at the record's `Head`, and matches a `not_found` answer to every live card whose targets hold the file, its self glyph, or a member resolving into it; an unresolved member falls back to the file and unit rule.
  The unused-import acceptance criterion now covers a card that targets only a member glyph.
  The `untraced` field text now reads "answered `not_found` and matching no card".
- F3 (fixed as proposed): after a commit error, D7's landed test now reads both files at HEAD, and reports success only when the amendment record equals this call's bytes and the decision record holds the entry's heading.
  When HEAD holds the amendment bytes without the heading, the verb keeps both files and refuses with `lyx fabric commit`.
  The one-commit sentence is softened for a status commit, which stages `_lyx/loom` whole (`statusCommitPathspec`).
  The failed-second-call acceptance criterion now has a partial-landing row.
- F4 (fixed as proposed): D7 says the amendment record is written before the append, and that it is restored when the append returns an error or check findings.
  The acceptance criterion now covers the findings refusal.
- F5 (fixed as proposed): D9 names the recorded head as `BatchState.CardSHAs`, and the failed or in-flight range clause is deleted, on the grounds of `verifyEveryBatchDone` and the order of the Webster, Merge-In and Publish rows.
  A batch record with no `CardSHAs` element now gives an empty range.
  This rule is stated in step 3, in the `cards` field text and in the rewritten acceptance criterion.

## Self-check

- fix-consistency, LOW: the `PATTERN-durable-vs-ephemeral-state` constraint still said the amendment record is always committed with the entry. Fixed as proposed: it now says so "unless a status commit lands it first (D7)".
- fix-consistency, LOW: the acceptance criterion still said "in one commit" with no exception. Fixed with my own change: the fork asked for the status-commit exception, and I scoped the criterion to "the verb's own commit" instead, since the partial-landing row in the same criterion already covers the exception.
- fix-proposal, MEDIUM: a member target that its card deletes, renames or moves out of the file does not resolve at `Head`, so an unused import it leaves stayed untraced. Fixed as proposed: such a target resolves at the start SHA of its card's batch, and the file and unit rule applies only when it resolves at neither.
- fix-proposal, LOW: a status commit could land the amendment bytes before an append refusal, which would leave HEAD with a title and no entry. Fixed as proposed: when HEAD holds this call's amendment bytes after the restore, the verb commits the restored amendment record before it refuses.
- fix-claims, MEDIUM: the claim that `AppendDecision` restores the decision record "in both cases" was false for a failed write, since it restores only after its check runs (`internal/discussionparser/append.go`). Fixed as proposed: on an error before the check, the verb restores the decision record itself.
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.
