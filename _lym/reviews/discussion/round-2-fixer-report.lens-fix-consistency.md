```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens. I checked whether the round-2 changes to `decision-record.md` and `support-log.md` still agree with the sections they touch, and whether any change was made in one place but not its twin.

1. MEDIUM — the new Constraints Fact on Codex `output_tokens`. Its cited evidence contradicts a sibling fact and D12.
   - The new fact cites `docs/research/codex-astra-context-parity-bench-2026-10-27/data/rollout-evidence.txt` as "every rollout total recorded".
   - The totals in that file carry `"cache_write_input_tokens":0` (lines 2-5).
   - The untouched sibling fact says Codex's usage carries no cache-write count. D12 says the same: "since the rollout carries no cache-write count".
   - The record now cites, as evidence, a file that shows a cache-write field, while two other passages say no such field exists.
   - Also, "every rollout total recorded under Codex CLI 0.164.0" is broader than what was checked. The 0.164.0 version comes only from the context-parity `report.md` (line 4). The p4-med bench's rollout totals have no `total_tokens` and were not shown to be under that version.

2. LOW — D10 and D9 disagree on what the store holds.
   - D10 now says the store holds the approved proposal with its hash, the Sketch's `created` stamp and the filed slugs.
   - D9 step 1 still stages only "the approved proposal, the cold proposal and a copy of the Sketch's record".
   - D9 step 3 adds the hash and the filed slugs. No step writes the `created` stamp, yet D2's start guard and the identity check depend on it.

3. LOW — D9's opening says step 3 is done when "the raddle store holds this run's Sketch with the approved proposal's hash". It does not say what makes a store directory "this run's".
   - The new D2 and D10 text decides that by the matching `created` stamp.
   - D9 should point to that rule, so its done check uses the same identity that D2 refuses on.

No further tool calls; everything checked was already in context.
