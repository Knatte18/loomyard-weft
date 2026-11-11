```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked that every refusal and next step the fix introduced can be reached and leads to progress.

**Ways forward introduced:**
1. D1, the refusal from plan approval through Webster: use `add` now and `replace` once the run has left its Webster rows.
2. D1, the added-entry subset refusal: no way forward is stated.
3. D3, the wider filed refusal: `add` or `replace` on the filed part, or `lym board upsert --record-file` on the family's project.
4. D4, the live-weft refusal: wait for the teardown, or run `lym fabric remove <slug>`, then re-run the upsert.
5. New criteria: D1 Webster span, D1 subset, D2 restored-X sweep, the two D3 additions, and D4 live weft.

**Findings:**

- **BLOCKING.** The D4 live-weft bullet ("It is refused while the slug's weft branch is live…") has no exemption for a family project.
  - A filed raddle run that keeps its worktree still has its weft branch locally, so `Add`'s test reads that weft as live.
  - D4 exempts such a project only from the worktree refusal ("A family project whose raddle run's filing is done … is exempt from the worktree refusal"). The new weft refusal still refuses it.
  - This breaks the unchanged criterion "D4: `--record-file` on a family project whose raddle run has filed and keeps its worktree replaces the project's stored record".
  - It also blocks the D3 way forward "or `lym board upsert --record-file` on the family's project".
  - Before the fix, this path worked.
- **MEDIUM.** D3's widened done test ("when the run's entry is no longer a task, or when the board already holds the filing…") has a way forward that loops.
  - The case is a family filing that saved `board.json` but did not write the raddle store.
  - D3 points the caller to `--record-file` on the project. D4 exempts a project from the worktree refusal only when its filing is done "as the raddle store records", so here D4 refuses, and the live-weft refusal refuses as well.
  - D4's own worktree way forward names `lym loom decision replace <slug>`, which D3 now refuses, or `lym batten cancel`.
  - Neither path files the record. The way that makes progress is re-running the filing so the store marker gets written, and neither refusal names it.
- **LOW.** D4's live-weft way forward, "finish it with `lym fabric remove <slug>`", may not remove origin's branch.
  - `internal/fabriccli/remove.go:104` shows `lym fabric remove` can fall back to a local-only removal: "the removal was local and --remote was dropped".
  - A weft that is live only on origin, the failed-remote-step case, could stay live after the command, so the way forward may not clear the refusal.
- **LOW.** The D1 subset refusal ("Each of its `### Added after Discussion` entries must equal, byte for byte, an entry of the current record…") names no way forward.
  - The `PATTERN-refusal-way-forward` constraint line requires every new D1 refusal to name one, for example re-reading the record and copying its added entries unchanged.

Tool calls:
- grep of `internal/fabriccli/remove.go`, to see whether `lym fabric remove` removes origin's branch. My context did not hold that command's behaviour.
