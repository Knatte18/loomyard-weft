# Round 2 fixer-report

## Changes

- F1 (BLOCKING), fixed as proposed.
  D4's Bound names two routes that file a proposal whose last round had open findings: the auto-accept key, and the soft-cap rule (`softCapRule`) with the key off.
  It states the soft-cap route's bound and that, with no `carry_over`, the accepted round's open findings are dropped from the record and stay only in the round's ledger.
  The D2 criterion now says the Bouncer never settles a spent-budget round with a design-wrong finding as an accept; a new D4 criterion covers the soft-cap accept.
- F2 (BLOCKING), fixed as proposed, second option.
  D6 names the window between the board write and the mark.
  A Raddle-File call that finds the converted entry filed writes the mark, then clears any run status on the entry, then writes the store, each step idempotent.
  New D6 criterion: a store write failing after the board write, then resumed, leaves the project with no status and releasable by D8.
- F3 (MEDIUM), fixed as proposed, no-Subject-seam option.
  D2 states the split perch's Bouncer has no Subject seam, so it keeps no settle or generation record, and its entry over the earlier settled verdict clears its run directory, giving the full budget.
  The settle-record clear is dropped. The D2 criterion asserts the full budget.
- F4 (MEDIUM), fixed as proposed.
  D2's re-arm and D4's set-aside apply on every fresh entry into Raddle-Write, a Raddle-File return or a goto, as `freshRaddleEntry` judges. The D2 criterion covers both.
- F5 (MEDIUM), fixed as proposed, limiting option.
  D6 says the filed parts are the entries the same write adds under the slug, each a discussed draft task, read from the write's result; `order` and the body's part list name them in dependency order.
  The admission is limited to an entry whose run status names the `Raddle-File` producer. The Bound says no writer limit applies and why that is accepted.
  The D6 run-lock criterion covers the producer limit.
- F6 (MEDIUM), fixed as proposed, with a narrower D9 sentence after the self-check.
  The Open risk bullet and the support-log ledger's Open line are dropped.
  D9 names c8e43ac4 as the commit that landed the Motif-first rule in `plugins/ly/skills/board-policy/SKILL.md`, and says the run takes main's text of that file.
  The proposal also named `contracts/orchrole/orch-role.md`, which `git show --stat c8e43ac4` shows the commit did not touch.
- F7 (LOW), fixed as proposed.
  The raddle-store sentence in D6 is deleted, and the support log's Rejected alternatives gains the raddle-store key bullet.
- F8 (NIT), fixed as proposed.
  The last Constraints bullet is deleted; D1's sentence keeps the reason.

## Self-check

- fix-consistency, LOW, support-log Turn 6 bullet `bound-breaker 6` stale after F5: fixed as proposed.
- fix-consistency, LOW, new D4 criterion placed among the D2 criteria: fixed as proposed, moved into the D4 group.
- fix-consistency, LOW, and fix-way-forward, LOW (merged), D6's accepted other-writer filing pointed at a "retry" path D6 defines only for retries: fixed with own change.
  The filed-entry path now applies to any Raddle-File call that finds the entry filed, on a retry or after another writer's filing.
- fix-claims, MEDIUM, and fix-proposal, LOW (merged), D9's sentence claimed c8e43ac4 changed both files: fixed as proposed, narrowed to board-policy.
- fix-way-forward, MEDIUM, and fix-proposal, LOW (merged), a failed status write at the move into Raddle-File makes the producer gate refuse the filing: fixed as proposed, first option.
  Raddle-File writes its own run status before the filing write when the status names another producer.
- fix-proposal, LOW, the D2 criterion claimed no route from the round reaches Raddle-File, which an escalation settled as an accept contradicts: fixed with own change.
  The criterion now pins the Bouncer's own settle, never an accept, rather than the run.

## Disputed

None.

## Deferred

None.

## Rulings

None.
