# Fixer report — discussion round 1

## Changes

- F1 (BLOCKING), fixed as proposed: D4's Bound now says the two moves never overwrite a present `lenses.yaml`, whatever its value, and only a target started from the template in the same pass takes burler's mappings.
  D4's Rationale explains why the general overwrite is given up.
  Open risks bullet 2 now follows that rule and names the dropped old-binary edit and the re-seeded file waiting on Bolt.
  The new criterion covers a migrated `lenses.yaml` equal to the template plus an old-template `burler.yaml`.
- F2 (MEDIUM), fixed as proposed: D2's Bound says a later template edit never reaches a hub whose file holds the name, and names the way forward, which is deleting the entry or editing it in.
  The Scope bullet on the template header says the same.
- F3 (MEDIUM), fixed as proposed, with the way forward corrected at self-check: D2's Bound says a template lens or fan cannot be removed and is written back as added.
  The way to turn a fan off is pointing the fan key entry at another fan, or setting it to `""`.
  The Scope header bullet names this bound.
- F4 (MEDIUM), fixed as proposed: D2's Bound gives the `--set` layers, the tune rule and the dot-free name grammar.
  It picks the `lenses` `CheckLoad` resolving every fan, so a bad fan is refused and rolled back.
  The `--set` criterion carries the refusal, and support log Q10 records the pick.
- F5 (MEDIUM), fixed as proposed: D2 declares the per-name kind on the `lenses` module's `configengine.Spec`, honoured by every template comparison that spec drives.
  A new criterion checks that a hub-only lens logs no unknown-keys warning.
- F6 (LOW), fixed as proposed: D3's Bound is rewritten to the `${env:...}` expansion, covering template and hub-held prose alike.
- F7 (LOW), fixed as proposed: D6 states that the board commit stages a written path's deletion, and Open risks bullet 1 is deleted.
- F8 (LOW), fixed as proposed: D1's Bound covers an empty `burler` object, and two criteria cover the kept-target drop at base and sparse layers and the `run_settings` refusal.
- F9 (NIT), fixed as proposed: D4's Bound picks a kept-target line that names the keys without rendering the mappings, and support log Q11 records the pick.
- Support log: the Review rounds entry for round 1, and ledger entries Q10 to Q12.

## Self-check

- fix-consistency, LOW: the Scope In list did not name the per-move opt-out D4 now needs. Fixed: the Scope key-move bullet names a per-move opt-out of `configsync`'s template-default overwrite.
- fix-consistency, LOW: D6's unconditional post-landing confirmation disagreed with the Open risk that a re-seeded `burler.yaml` can be on Bolt. Fixed: D6 says that reconcile with `--apply` removes such a file through the same moves.
- fix-claims, MEDIUM: "the dropped values stay in Bolt's history" was false at a sparse layer. Fixed: D4 says a base layer's dropped values stay in the parent of the deleting commit and a sparse layer's are kept by nothing. Support log Q11 was narrowed to match.
- fix-way-forward, LOW: the re-seed Open risk named no next step. Duplicate of the second fix-consistency finding, fixed by the same D6 sentence.
- fix-proposal, MEDIUM: a `lenses.yaml` scaffolded by `--set` or `--edit` before the first post-deploy walk makes that walk drop burler's tuned values under F1's rule. Fixed with my own change: D4's Bound states this window and its way forward, which is a start verb or a default-layer reconcile before the first `lenses.yaml` write.
  I did not take the fork's suggestion of a narrower rule keyed on Bolt tracking or the first walk, because it needs new state to tell a first walk from a later one. Support log Q12 records the tradeoff.
- fix-proposal, MEDIUM: "emptying that key" is refused by `ValidateFanKeys` (`internal/burlerengine/fankeys.go:62-63`). Fixed: the way forward is setting the fan key entry to `""`, in D2 and in the Scope header bullet.
- fix-proposal, LOW: `CheckLoad` stopping at the first failing fan in map order makes its error text nondeterministic for `setChecked`'s comparison. Fixed: D2 says fans resolve in name order and every failure is reported.
- fix-proposal, LOW (F9 sparse-layer history): duplicate of the fix-claims finding, fixed by the same change.

## Disputed

None.

## Deferred

None.
