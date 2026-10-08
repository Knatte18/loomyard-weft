---
round: 1
focus:
  - "D12 (spawn-order boot): does the claim that concurrent booters each create a session only after its predecessor exists hold, given sequential ups outside the booting worktree's op lock and a booter that finds the server already up? Settle whether two booters can race on creation order or wait on each other."
  - "D8 and D9 (server-wide root-table keys, replaced `MouseDown1Status`): for each, does the stated bound match what the binding can actually skip or intercept, including windows without `@lyx_strands`, standalone mode, and a hub server that also hosts non-reed sessions?"
  - "D6 versus D4, D5 and D7: the scoping of `status`, `status-format`, `pane-border-*` and the geometry row budget. Do the session-scoped, global and window-scoped pins together produce the two-line bar and the exact layout fill the acceptance criteria claim, in the prime (with batten windows) and in a pair?"
  - "D2, D3 and D14: are the paths that put `@strand`, `@strand_color` and `@lyx_strands` on a pane or window complete for fresh, resumed and pre-change strands, and does the persisted segment on the strand record cover a resumed strand's color without a new unstated writer?"
  - "D15 (writes the user's VS Code `keybindings.json`, a file outside the repo): is the marker-block edit bounded for a JSONC file with comments or trailing commas, and are the `]` placement and skip rules consistent with the idempotence and byte-identical criteria?"
  - "Cross-decision consistency: wait marks (D4, D16, `PATTERN-wait-mark-display-only`), the `lyx reed switch` ordering (D5, D10, D12) and the acceptance criteria. Does any criterion contradict or outrun a Decision?"
---

Where to look first is the set of decisions that add a binding, a verb or a file write, since the attack-surface question applies there.
None of the entries targets the optional notes subsection, rejected alternatives, or call-site enumeration.
