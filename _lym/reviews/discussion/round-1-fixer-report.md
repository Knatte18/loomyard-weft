# Fixer report — discussion, round 1

Target files: `_lym/discussion/decision-record.md`, `_lym/discussion/support-log.md`.
F12 (`decision`) went to the parent and is left out here.

## Changes

- F1 (BLOCKING), fixed as proposed (first option): D1 caps a message's `MessageLine` at the 8 KiB claim budget at `post`, with the existing file way forward, in place of the 64 KiB `maxBodyBytes`. An old-binary line past 10,000 characters is left out of every claim. The Constraints bullet now states the bound the rule gives, and a criterion covers a post over the budget.
- F2 (MEDIUM), fixed differently in part. Research in the 2.1.312 bundle's query loop: every turn, a rewake turn included, starts Claude Code's block count at 0 with `stop_hook_active` carried in, true for a rewake. A tool continuation resets the count and keeps the flag. This is recorded in Constraints.
  - Done as proposed: resets are applied before the cap check, and a ring starts a fresh count.
  - Not done as proposed: "the Stop of a turn a D5 bell started claims the mail". D5's at-cap trigger is dropped instead. After the cap `Stop` the strand is idle, and D4's ring, which resets the count, wakes it. That needs no typed line, where the bell types into the pane for a strand whose waiter is live, against D6's direction.
  - Criteria added for three successive idle wakes, and for the cap followed by a ring and delivery.
- F3 (MEDIUM), fixed as proposed: a starting daemon sets every strand's count to the cap, and the count is never journaled (D2). D4's Bound adds the possible re-ring after a restart. A criterion covers a restart mid-chain.
- F4 (MEDIUM), fixed as proposed: D4 names a new `Idleness.NoTurn` field and states which stalls hold the ring. D5 uses the guarded send's own `Delivery.Idle`, so a draft, a too-short pane or a missing input box holds the bell.
- F5 (MEDIUM), fixed as proposed: D5 bells only strands carrying a new opaque reed field, `CreelHooks`, which shuttle sets on launches that render the new hooks. Hand-added strands, `lym shuttle open` sessions and pre-deploy sessions lack it. The Open risks clause is dropped.
- F6 (LOW), fixed as proposed (first option): `hook-read` writes straight to the hook's output and exits 0 exactly when it wrote a block. The merged command decides on recording from that exit status. The Bound states that a killed hook leaves the claim to come again.
- F7 (LOW), fixed as proposed: a blocked `Stop` appends neither the stamp nor the payload. The stamp is written only with the record. The existing criterion now says so.
- F8 (LOW), fixed as proposed, way forward changed in the self-check: D7 refuses `--interrupt` before any key for a receiver without the mark. A store failing after the key answers that the turn was interrupted and the post not stored. Criterion updated.
- F9 (LOW), fixed as proposed: D6's rationale is corrected, and its Bound names the unmarked strand's lack of any signal.
- F10 (LOW), fixed differently: the fix proposed appending the sentence to the single `--append-system-prompt` argument. Instead, the deny notice and the sentence go in one file passed with one `--append-system-prompt-file` flag, which 2.1.312 accepts. That argument cannot carry the sentence's `"` reliably on Windows (F11), so a single argument would keep the defect. Criterion updated.
- F11 (LOW), fixed as proposed: the `internal/shell` Constraints bullet records that pwsh 7 keeps an embedded `"` only in `Standard` argument mode. `Legacy` mode drops it, and pwsh uses `Legacy` for `.cmd` and `.bat` targets from 7.3 on, and for every target before 7.3. Shuttle runs whatever `claude` resolves to. D9 therefore carries the sentence in a file.
- Support log: round-1 entry, revised Q10 and Q3, and new rejected alternatives.

## Self-check

Lens reports: `round-1-fixer-report.lens-fix-consistency.md`, `round-1-fixer-report.lens-fix-claims.md`, `round-1-fixer-report.lens-fix-proposal.md`, `round-1-fixer-report.lens-fix-way-forward.md`. Every fork returned a report.

- fix-consistency 1, MEDIUM (D1 says the claim is unchanged despite the exclusion): fixed as proposed.
- fix-consistency 2, MEDIUM (D9 file has no home for `lym shuttle open` sessions), merged with fix-way-forward 2: fixed with own change. D9 and the auto-mode assumption now exclude open sessions, which carry no deny notice today either.
- fix-consistency 3, LOW (D7 says "body size"): fixed as proposed, now "line size".
- fix-consistency 4, LOW (D6 and the criterion say "in full" despite the exclusion): fixed as proposed.
- fix-consistency 5, LOW ("restarted" is ambiguous), merged with fix-way-forward 4: fixed as proposed. Scope Out and Open risks now say a fresh shuttle launch.
- fix-consistency 6, LOW (Q3 in the ledger is stale): fixed as proposed.
- fix-claims 1, MEDIUM (a ring mid-chain in a woken turn resets the count late): fixed with own change. The daemon holds `NoTurn` false from a ring until a turn end recorded after it (D4), and D2's Bound now rests on that.
- fix-proposal 1, MEDIUM (LOOM's decision notice cannot take the file way forward): fixed with own change, not the headings-only fallback, which would drop the text a role that reads no record needs. LOOM sends one notice per entry when the joined line would pass the budget. `lym loom decision add` refuses, before recording, an entry whose own notice would pass the budget, and names the way forward. Criterion added.
- fix-proposal 2, LOW (the cap needlessly refuses long replies to `EXTERNAL`): fixed as proposed. Replies to `EXTERNAL` keep the 64 KiB body cap.
- fix-proposal 3, LOW (a killed merged command can lose the turn-end record): fixed as proposed, as a stated window in D2's Bound.
- fix-way-forward 1, MEDIUM (D7's refusal names ways forward that deliver nothing): fixed as proposed. The way forward is now `lym creel inject --interrupt <receiver> <text>`, or a post left for `lym creel read`.
- fix-way-forward 3, LOW (the store-failure error names no way forward): fixed as proposed. The way forward is to post again without `--interrupt`.

## Disputed

None.

## Deferred

None.
