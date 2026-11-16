---
round: 1
focus:
  - "Granularity, card 3: it lists many private helpers (`nameDeps`, `nameResult`, `hubOwners`, `pullOwners`, `checkCollision`, `rewriteFile`, `sweptHubs`, `productionDeps`) as separate Create targets. Settle which are private supporting types or helpers of one surface and which are reused across packages."
  - "Test economy, cards 3 and 4: compare `TestNameSet` rows (card 3) with `TestNameSweep` rows (card 4) for overlap, such as the unreachable-origin hub and the origin-only owner. Compare `TestNameShow` and `TestNameRefusesWithoutWriting` with the existing `internal/machinename` tests. Name the covering test for any finding."
  - "Test economy, card 2: check whether the new `TestSetOwners` rows and the extra subtest repeat behavior `TestSetOwners` or `CompetingWriteLosesToOrigins` already cover."
  - "Edit-card glyphs and the re-sign arrow, card 2: `OwnerRequest` gains a field and `SetOwners` and `writeOwners` change behavior. Settle whether any listed member's parameters, results or receiver change, and whether the member glyphs fit the Intent."
  - "Card 2 targets `PATTERN.md` under `**Edit:**` while its Intent leaves the line there exactly as fenced. Settle whether an edit target is justified, and whether `Extends` or no target is the right typing. Settle whether the `pattern/PATTERN-owner-machine.md` bullet rewrite is correctly an Edit."
  - "Attack surface, card 2: `Replace` lets a write take owners that read as another machine's. Settle whether the Intent states what it skips and what bounds it. Check the same for the collision check in card 3 against the card 4 sweep."
  - "Reachability, cards 3 and 4: for each refusal or guard branch, such as the unresolvable config directory, the invalid name, the hub checked locally only and the failed hub, settle whether a named input and a named test reach it, and whether an upstream check rejects the input."
  - "Fidelity to the answer key: read `_lym/discussion/decision-record.md`. Settle whether every Decision and Constraint (D5, D6, D8, the added-after-Discussion entry for `HostnameName`, the window-bound recoveries) is carried by a card, and whether any card adds scope the record does not license."
  - "Card order and research steps: card 4 edits symbols card 3 creates, and card 3 uses `HostnameName` from card 1. Settle whether any card leaves a design fact to be found while it runs. Check that no card edits this task's own board entry."
  - "`ImpactSummary` content: the Edit and Extends cards' summaries (cards 1 to 4). Settle whether each carries a real blast-radius conclusion and not a restatement of Intent. Card 4's `Extends`-like summary claims no other package changes while the card calls `boardengine.SetOwners`."
---

Seed focus drawn from reading the five plan files only; no `Do not flag` item is promoted into an entry.
