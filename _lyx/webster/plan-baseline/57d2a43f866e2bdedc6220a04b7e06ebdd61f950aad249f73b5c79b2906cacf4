# Card 5 — The exclude census keys on the anchored pattern

**Edit:**
- `internal/fabricengine#namesWiredInSiblingWorktrees`
- `internal/fabricengine#unseedGitExclude`
- `internal/fabricengine_test#TestUnwire_LeavesSiblingWorktreeUndirtied`

**Uses:**
- `plan:internal/lyxcwd#NamedWorktree`
- `plan:internal/hubforge#ReanchorPair`
- `internal/fabricengine#excludePatternFor`

**Intent:**
D9.
`namesWiredInSiblingWorktrees` resolves each other registered worktree's Location with `lyxcwd.NamedWorktree(hub, filepath.Base(entry))` and checks for the link at that worktree's own anchor, not at `l`'s.
It answers per name two facts: whether some other worktree has the link at the same anchored path as `l` (its anchor equals `l.AnchorRel`), and whether some other worktree has it at any anchor.
`unseedGitExclude` strips the anchored line `/<l.AnchorRel>/<name>` only when no other worktree has a junction at that same anchored path, and strips the legacy bare `<name>` line only when no other worktree has that name wired at any anchor, since a slash-free line matches at every depth.
A sibling whose Location read fails, and a failed worktree listing, are answered as still wired, keeping today's conservative rule.

`TestUnwire_LeavesSiblingWorktreeUndirtied` becomes a table over pairs and anchors, each row unwiring one pair and asserting the exclude lines left and that every other worktree's `git status --porcelain` stays clean:
two pairs at the hub's anchor (today's row) keep the shared line;
the unwired pair re-anchored at `sub` by `hubforge.ReanchorPair` with no other worktree at `sub`, and a legacy bare `_lyx` line written into the exclude file beside the anchored ones: `/sub/_lyx` is stripped, and the bare `_lyx` line stays because the prime and the kept pair still wire `_lyx` at `.` (today's census, checking every sibling at `sub` only, strips it, so this row fails on today's code);
the same hub with the kept pair also re-anchored at `sub`: `/sub/_lyx` stays for the kept pair, which today's census also keeps, pinning that the per-anchor check still finds a sibling at the same anchored path.

**ImpactSummary:** two private exclude helpers in `junction.go`, reached by Unwire, unwire-junctions and reconcile's stale removal; one-anchor hubs strip exactly what they strip today.
