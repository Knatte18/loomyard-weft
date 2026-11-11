```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of decisions that act on the same state, then each acceptance criterion and constraint against the decisions it touches.

**Pairs checked**
- D1 replace × D2 trailer-bounded history, normal path: held. The trailer's sha256 cuts at the newest version whose bytes match.
- D1 swept-bytes and empty-trailer-commit paths × D2 cut point: held. Versions come from commits that touch the path, so the cut lands on the commit that swept the bytes.
- D1 commit-failed restore path × D2: held. No trailer lands, so the history is read whole as before.
- D1 second replace failing after a sweep × D2: held. The older trailer cuts history, the check flags the drop, and re-running `replace` lands the trailer commit.
- D1 decision lock × D1 `writeCarryOver` × D1 rule that carry-over blocks stay byte-equal: held.
- D1 × D3, `replace` refreshing the board copy, and the refusal once filing is done: held.
- D1 × D3, `store` taking the lock and being admitted on raddle runs (the raddle recipe has no Discussion approval row): held. D3 states the widening.
- D1 × D5: held. A replace's board copy re-stamps `record_body_sha256` from the body as it stands, and promote reads that field only before raddle starts.
- D1 Webster-row refusal × D6 parent directive: held. D6 tells the parent to `replace` past approval and does not mention the Webster refusal, but that refusal's own way forward covers it.
- D1 × `add --amends-plan` amendment titles: held. The amendment record lists only pending titles (`internal/loomshed/amendment.go:38`), and D1 folds only after the amendment round settles.
- D3 × D4, family project after filing: held. The worktree exemption applies, and `arrasProjectOf` drops the status, so no run holds the entry.
- D3 × D4, non-family filing: held. The run entry is ended done, so D4 refuses it, and D3's way forward names the filed part.
- D4 worktree refusal × D1: held. The way forward chain is: `replace` in the worktree; then the Webster-row or pre-approval refusal; then `add`.
- D4 × D5: held. The stamp is taken from the body as the same write leaves it.
- D5 × D3 and Raddle-File: held. The family conversion's changed body is stamped by the filing write.
- D5 × Sketch conversion: held. The run lock and `endsAsSketch` leave the body unchanged.
- D4 × D2 × Record-Write, a raddle run on the motif pair's still-live weft (Open risks bullet 4): **broken**, see finding 1.
- Constraints × D3: **broken**, see finding 2.
- Acceptance criteria × D1 to D6: held, bar finding 2.

**Findings**

1. MEDIUM · design · D4 last bullet and Open risks bullet 4.
   - A raddle run that adopts the motif pair's still-live weft keeps the motif run's committed `_lym/discussion/decision-record.md`. So a record replaced with `--record-file` never reaches raddle, and the Open risk's stated consequence and remedy are wrong.
   - Evidence: `internal/loomshed/recordwrite.go:120-124,153` writes the record only when the file is missing ("A file that exists is never overwritten"). The `internal/fabricengine/add.go:150` doc says an adopted weft branch keeps its own run records.
   - Nothing clears `_lym/discussion` on a start: a grep of `internal/loomcli` and `internal/loomshed` found no removal of that directory.
   - The Open risk assumes the run's record is the folded board copy, so the dropped-entry check would flag the folded entries. In fact the record is the stale motif record and nothing flags it.
   - Its remedy, "a `replace` whose file is the record's own bytes", re-commits that stale design.
   - Net effect: in the remote-teardown-failed case, D4's purpose (raddle reads the current design) silently fails.

2. LOW · consistency · Constraints, `PATTERN-refusal-way-forward` bullet.
   - The bullet names "every new refusal of D1 and D4" but omits D3's new refusals. Those are `add`, `replace` and `store` refused on any run whose raddle filing is done, and `store`'s changed admission. All live in `internal/loomcli`, which the entry binds.
   - D4's refusals sit in `internal/boardcli`, which the entry does not bind (`pattern/PATTERN-refusal-way-forward.md:4`).
   - The constraint therefore names the wrong decision set.

**Tool calls**
- Grepped for readers of the plan amendment record's titles: needed to check D1 against the `--amends-plan` fold, and they were outside the manifest.
- Grepped `loomcli` and `loomshed` for removal of the discussion directory, and read the motif recipe's rows: needed to learn whether a start on an adopted weft clears the motif record, which the manifest did not show.
