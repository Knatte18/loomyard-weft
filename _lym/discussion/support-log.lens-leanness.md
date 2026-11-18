```yaml
lens: leanness
outcome: reported
```

Task: review the decision record and support log through the leanness lens, looking for content to move or cut.

**Findings**

1. **LOW — Auto-mode assumptions vs D2, D3, D5, D7.** The section repeats three decisions instead of recording separate assumptions.
   - Evidence: "pinned worktree … waits it out" restates D7's last rationale sentence. "pick runs at a worktree's making only…" restates D2 and D3. "status line replaces the operator's own…" restates D5's Facts.
   - Fix: keep one place for each, either each decision with an assumption that only references it, or the assumption without the repeat in the decision. The support log's Question ledger already holds the auto-pick reasoning.

2. **LOW — Auto-mode assumptions, last bullet.** "The command names `lym claude account` and the file `accounts.json` are this record's; Plan may rename…" is a hint for the plan writer, not an assumption about the design.
   - Fix: move it to an optional `## Notes for the plan writer`.

3. **LOW — Constraints, the "Size" bullet.** The 1,200–1,600 line estimate does not bind the design. The board asks the Discussion to "name its estimate", so it must stay in the record.
   - Fix: move it to `## Goal` (or Notes) so Constraints holds only rules the implementation must obey.

4. **LOW — Constraints, `PATTERN-shuttle-provider-seam` appears twice.** The first bullet covers the seam. The separate "shuttle stop" bullet restates D7's "stops the strand through its stop verb".
   - Fix: merge it into the first bullet, or cut it and rely on D7.

5. **LOW — Scope Out, the adopt bullet.** "…this record only picks the moved worktree an account there (D3)" repeats D3's pick moments.
   - Fix: trim the bullet to the exclusion.

6. **LOW — D2 rationale.** The sentence "The adopt design keeps run state in git, but…" argues why nothing goes into git, and D2's body already decides that ("Nothing of it goes into git… a cache in `PATTERN-lym-files`' sense").
   - Before moving it: the board asks the Discussion to weigh the git question, so keep one short clause in the record and move the rest to support-log Rejected alternatives, which already has the bullet "Keeping the account entry in a tracked file for adopt".

7. **LOW — D9 rationale.** It mostly repeats D1's description of `account_pins.beams` ("with them every orch and dobby").
   - Fix: keep one of the two statements.

**Other notes (outside leanness)**

- **MEDIUM — D6, claim about existing code.** "The orch's own resume path builds its lines through the engine as well" is unverified. The exploration named `orchcli/start.go` and `tenderengine/resume.go` as the orch's resume paths and did not establish that they go through the claude engine's `Prepare` or resume builder. If they don't, D6's copy and D4's export miss the orch, and D9's acceptance criterion fails. Verify this, or state that the orch's resume line must be built by the engine.
- **LOW — relevant.md.** It lists files the writer knows only from subagent reports, not files it read itself. The spec says "records only what it already read". This is borderline, since the files came from this session's own exploration.

I made no further tool calls.
