---
round: 1
focus:
  - "Fidelity: for each D3 refusal (former name equals another machine's current name, current name equals another's former or current name, copied machine.json, unresolvable config directory), find the card that carries it and the test that reaches it."
  - "Fidelity: D5's bolt-hold rules (no entry for an owed push or the adopt-fix fix task, no lift of a bolt-hold entry under a hold, tender resume refusal naming the hold's machine) and the upgrade's first read order (push intents and hold first, drops after); find the card that carries each."
  - "Card order: cards 7-9 move comparisons onto the registry, and 20-21 call the catch-up (18) and pause-intent entries (19); check that no card uses a symbol a later card creates, especially around cards 5, 12, 15, 18, 19 and 22."
  - "Reachability: cards 4, 10, 11 and 13-17 prescribe refusals; check each names an input and a test that reaches it, and that no upstream check rejects the input first."
  - "Attack surface: the new --forget-former verb (card 11), the hold write (card 5) and the local-mode pair removal in the hand-over (card 17) must each state in Intent what they can skip and what bounds it."
  - "Granularity and test economy: cards 7, 20, 21 and 23 are the largest; check for per-symbol tests and for tests duplicating coverage that already exists, such as TestReadRun_ReadsTheSameStateWithoutTheCache."
  - "Board entry and research steps: check that no card changes this task's own board entry body or scope, files or promotes a task, or leaves a fact for the run to discover."
  - "Edit-card shape: check member glyphs and re-sign arrows on the Edit cards whose Intent changes signatures, such as OwnedElsewhere, OwnedWorktrees, EnsureLive and ReadRun, and Extends-versus-Edit typing."
  - "Card 25 and the overview's PATTERN changes: check the PATTERN-owner-machine line against the decision record, and each card against the PATTERN entries listed under Constraints."
---

The plan carries no Verify lines or Custom cards on a first grep, so those checks are left out of the focus.
