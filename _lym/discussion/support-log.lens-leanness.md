```yaml
lens: leanness
outcome: reported
```

Task: leanness review of the decision record and support log. I looked for content to move or cut, and noted anything else in passing.

**MEDIUM** — `## Constraints`: most bullets repeat what a decision already says.
- Evidence: each pairs with a decision that already carries the same rule and reason:
  - PATTERN-fabric-git and the marker: D3's rationale.
  - PATTERN-tolerant-records: D1.
  - PATTERN-config-layers and Cross-pair: D5.
  - PATTERN-shuttle-provider-seam: D4's rationale.
  - "The rendered text reaches every agent… only what is specific": D1's last paragraph.
- Fix: keep only the constraints no decision states (PATTERN-cwd-resolution's unchanged import set, PATTERN-sparse-hub-base, cli-cobra/cliwire, spawn and run-process). Cut the rest, or replace them with one line pointing at the decision.

**LOW** — D1 rationale: "The 'possibly also when it renders' condition is left out: nothing asks for one yet."
- Evidence: this is a rejected alternative. The support log's `## Rejected alternatives` already has it ("A `when` condition on an entry"), and `## Auto-mode assumptions` repeats it a third time.
- Fix: cut it from D1. The spec says `## Decisions` carries no rejected alternative.

**LOW** — D9: the import-graph argument is made three times.
- Evidence: D9's first bullet explains why the literal sits in `lymcwd`; D9's Rationale says it again ("the import graph allows no other single home"); `## Open risks` makes the argument a third time.
- Fix: keep the evidence in the first bullet and cut the Rationale's second clause. Open risks can say only that this departs from the entry's wording and name D9.

**LOW** — D5 last sentence ("Fabric's code names no heddle file… D3's refusal keeps the two in step") repeats D3's exclude-list refusal bullet and D7's closing line.
- Fix: cut it, or shorten it to "Fabric's code names no heddle file."

**LOW** — `## Notes for the plan writer`, third bullet, repeats D5 ("same exclude-file mutation… anchored at the warp root").
- Fix: cut it.

**LOW** — D2: the flag spellings (`--text-file`, `-` for stdin, `--order`, `--engines`, `--delivery`) and "JSON envelopes as `lym board` prints" are interface detail the plan works out.
- The verb set is a design choice and stays.
- Fix: keep "a new entry needs all four fields, an update changes only those given" and drop the flag names. Optional, since the verbs were the task's to shape.

**LOW** — D6's two "(operator, 2026-10-31)" sentences repeat the board entry word for word. They are constraints the plan needs, so they stay. Dropping the attribution would make them read as decisions.

**LOW** (outside my lens, noted only) — D4 says `--append-system-prompt-file` reads the whole file, so multi-line passages are fine. No source or Claude Code version is given, even though the design depends on it; other facts in the record are cited with their source. Cite a source or test it.

Held, nothing to cut:
- D3's rationale on root placement is evidence the design depends on.
- D7's run-ledger bullets carry requirements.
- `## Auto-mode assumptions` is a required section and stays terse.
- The support log's turn narrative is distilled.

Tool calls: none. Everything I needed was already in context.
