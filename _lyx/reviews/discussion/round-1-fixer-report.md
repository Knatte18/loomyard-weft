# Fixer report — discussion, round 1

Every finding's premise was checked against the code (`internal/planglyph/plangate.go`, `handle.go`, `internal/planparser/validate.go`, `targetdirs.go`, `verifypackage.go`, `internal/planglyph/scope.go`, `internal/websterengine/recordbatch.go`) and holds.

## Changes

All changes are in `_lyx/discussion/decision-record.md` unless named otherwise.

- F1 — fixed as proposed (the D7 option): D7 states that `isInterfaceMethod` answers false for every C# symbol, so `resign-interface-method` and the `func (Owner)` receiver synthesis stay Go-only; the re-sign and Rename acceptance rows assert it.
- F2 — fixed as proposed (the rewrite option): at record-batch, a C# re-sign whose minted id differs names the arrow again, resolves the minted id, raises the new `resign-not-done` unless found, and rewrites the old glyph to the minted id plan-wide through `planparser.RewriteRefs`.
  The unbacked "the done check and drift expect it" claim is gone, and the acceptance row matches the mechanism.
  The self-check added that later cards name the member by its old glyph, and that the batcher starts a new batch at the first later card naming a C# re-sign's target.
- F3 — fixed as proposed, with the substitution rule spelled out: the new identifier comes from D3's helper with the arity suffix stripped, the var and const synthesis is Go-only, and the substitution replaces the declared name token, the first occurrence of the old name outside attribute lists that a declaration delimiter follows.
  A Bound names the head it misreads (`Color<int> Color`).
  Acceptance rows were added for a C# method, const and same-named property Rename.
- F4 — fixed as proposed (the widening option): `plan-language-mismatch` also fires under `go` in the resolve pass when the answers name `.cs` files and neither a path target nor an answer names a `.go` file.
  The Bound states that a `go` plan whose only C# refs are namespace self glyphs is let through, and the Rationale was corrected.
- F5 — fixed differently.
  Proposed: restrict the file-pair rule to sides whose Go reading has a file extension or a slash.
  Not made, because a dotted namespace such as `Norce.Models` has the path extension `.Models`, so the extension test cannot tell the namespace pair apart.
  Made instead: the pair classification stays parse-only, and the plan gate raises the new `rename-namespace-pair` for a pending pair whose slash-free old side names nothing on disk and nothing an earlier card creates or renames into.
  Its way forward names Edit cards on the declaring and referring files, and an acceptance row was added.
- F6 — fixed as proposed: D3's Bound gains `redundant-file-target` and the `RefCovers` batch-boundary loss, and `ScopeGuard` receives the plan.
- F7 — fixed as proposed: D3 states that `internal/planparser` exports the Go-alphabet path reading, with D4 and the namespace-pair check as its only callers, and the chokepoint constraint line allows that one reading.
- F8 — fixed as proposed: D7's re-sign bullet gains the parameter-list presence bound, refused as `resign-head-mismatch` with a C# way forward, and `resign-overload-collision`.
  The minted ids join the one batched resolve by naming the arrows first.
  The self-check added an exception for an overload that the same or an earlier card removes.
- F9 — fixed as proposed: `Owner` leaves out the explicit-interface qualifier the head spells; an acceptance row was added.
- F10 — fixed as proposed: D6's listing bullet is deleted and the README.md criterion no longer mentions listings; Scope In's `listings` mention, D6's rationale and the support log's Turn 4 were trimmed to match.
- F11 — fixed as proposed: D1 takes a re-sign arrow's `Lang` from D3's parse of the target.
- F12 — fixed as proposed: D6 states the one rule, that `classifyAmbiguity` partitions only all-`.go` candidate sets, and D5 points at it.
- F13 — fixed as proposed (the bound option): D5's Bound states that a C# card's Verify package arguments, `lyx gate test` included, go unchecked, and the Rationale was corrected.
- F14 — fixed as proposed: the PATTERN background file bullet moved from Notes into Scope In.
- F15 — fixed as proposed: the Scope paragraph is cut (support-log Turn 2 records the departure), and so is the batch-estimate Open risk.
- Twin: `_lyx/discussion/support-log.md` records round 1 under Review rounds.

## Self-check

- fix-consistency, MEDIUM — the disk-based file-pair rule contradicts the support log's rejected disk-existence alternative: fixed differently; the pair classification now reads no disk, and the disk check is a plan-gate refusal, so the rejected alternative for parsing still holds.
- fix-consistency, LOW — support-log Turn 4 still says listings are read: fixed as proposed.
- fix-consistency, LOW — no criterion reaches `resign-not-done`: fixed as proposed (added to the re-sign row).
- fix-consistency, LOW — D8 does not teach the new refusals: fixed as proposed (D8 names them and the old-glyph rule).
- fix-consistency, LOW — support log has no round record: fixed as proposed.
- fix-claims, MEDIUM — `isFileRenamePair`'s callers hold no worktree root, and `SymbolRenamePairs` promises no disk read: fixed differently, merged with the first finding; the classification stays parse-only.
- fix-way-forward, MEDIUM — a disk test misclassifies a landed pair and a pair whose root file an earlier card creates: fixed as proposed (pending cards only, plan-created paths count).
- fix-way-forward, MEDIUM — the `rename-namespace-pair` way forward leaves references broken: fixed as proposed (it names referring files too).
- fix-way-forward, MEDIUM — `RewriteRefs` has no pending filter: fixed as proposed (plan-wide); merged with fix-proposal LOW 4.
- fix-way-forward, MEDIUM — the parameter-list refusal is unnamed and has no way forward: fixed as proposed (`resign-head-mismatch` with a C# detail).
- fix-way-forward, LOW — `resign-overload-collision` and `resign-not-done` give no way forward: fixed as proposed.
- fix-proposal, MEDIUM — the D4 resolve half refuses a Go plan whose `.go` side is member glyphs only: fixed as proposed (no answer may name a `.go` file either).
- fix-proposal, MEDIUM — `resign-overload-collision` refuses a plan whose earlier card removes the overload: fixed as proposed (exception added).
- fix-proposal, MEDIUM — the planner is never told to name a re-signed member by its old glyph: fixed as proposed (D7 and D8).
- fix-proposal, LOW — the rewrite reaches more than pending cards: fixed as proposed, merged above.
- fix-proposal, LOW — `resign-not-done` misfires when a same-batch card changes the member again: fixed differently; the batcher starts a new batch at the first later card naming a C# re-sign's target, so no batch holds both.

## Disputed

None.

## Deferred

None.
