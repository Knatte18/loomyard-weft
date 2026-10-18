# Fixer report — plan, round 3

## Changes

- **F1** (BLOCKING, decision): fixed as proposed in `_lyx/plan/10-status-seams-spawn-detached-push.md`.
  The 152-byte list, the "211 bytes owed" paragraph and the stop-and-ask-`ly:orch` instruction are gone.
  Card 10 now names a complete 368-byte cut and makes it first: line 3's introductory sentence and 37 entries, each given by name and its new text after the em dash, each keeping its rule, `(test)` marker and background link.
  I measured the cut with `wc -c` on a full candidate copy of `PATTERN.md`, which came to 15188 bytes, from 15556.
  With the plan's 371 bytes of growth that is 15559, 5 bytes under `HeadroomBytes` (15564).
  Entry names are unchanged, and no rewritten line got longer.
  Card 15 now states its two lines' 112 + 189-byte budget against card 10's cut, so a later change to its wording knows it has to pay for itself.
- **F2** (MEDIUM, design): fixed as proposed in `_lyx/plan/08-push-lock-pending-flags.md`.
  - Step 4 now builds the respawned request from the flag's validated side path and slugs only. A `pending.code` flag's request keeps only `WarpPath`, and a `pending.records` flag's request keeps only `WeftPath`.
  - "What this lets through" and "What bounds it" now say this.
  - "What bounds it" also says a flag whose `PauseSlug` or `TeardownSlug` holds a path separator or a `..` element is dropped with a Warn naming the flag, because `TeardownLockPath` builds a file path from the teardown slug.
  - `TestCoalesceFlagged_PendingFlags` gains the two proposed rows. Each row's flag carries the paired code worktree's `WarpPath`, so it gets past the side-path bound and reaches the new check.
- **F3** (LOW, scope): fixed as proposed. `TestCommandEntry` in card 15 gains a non-string `command` (`3`) row, under the same assertion as the other rows.
- **F4** (LOW, scope): fixed as proposed in card 15.
  - `minimalEntries`' `Command` row and `TestCommandEntry`'s building rows set `env.Setting` to `fakeSetting`, answering the switch and command references, and leave `ExecutableSetting` nil.
  - The producer-call row also sets `env.ExecutableSetting` to `fakeSetting`, answering the command reference.

## Self-check

- fix-claims: no findings.
- fix-affected-set: no findings.
- fix-consistency, LOW: card 8's new `..` `TeardownSlug` row did not say the flag's `WarpPath` is the paired worktree, so the side-path bound would drop the flag before the slug check runs. Fixed as proposed. The same fixture fix went into the sibling opposite-side `WeftPath` row.
- fix-proposal, MEDIUM: the `PATTERN-creel-store` rewrite dropped the second "only" and weakened the rule. Fixed as proposed: "only" is restored, and the `PATTERN-lyx-temp-root` rewrite's "run directory" becomes "run dir" to pay for it. The cut is now 368 bytes.
- fix-proposal, LOW: a 4-byte headroom margin depends on card 15's wording. Fixed as proposed: the margin is now 5 bytes, and card 15 states its budget.
- fix-proposal, LOW: card 8's slug-check row could pass on the side-path bound alone. Duplicate of the fix-consistency finding, fixed there.
- fix-proposal, LOW: `TestCommandEntry`'s producer-call row read the command through `ExecutableSetting`, which the fixed text left nil. Fixed as proposed: that row sets `env.ExecutableSetting` to `fakeSetting`.

## Disputed

None.

## Deferred

None.
