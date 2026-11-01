```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I checked pairs of decisions that act on the same state and ran the acceptance criteria and constraints against the decisions they touch.

**Pairs checked**
- D1 × D5 (status-format[2] and `status 3` against the row options rendered into session options): held. The re-pin restores the global formats, and the rows come from per-session options.
- D1 × D7 (stale-build re-pin against the `lymbtn*` click branch in the same binding): held. The click branch sits in `bindingArgvs`, and the re-pin reissues it.
- D2 × D3 (reed's groups, then the machine's, with the prime's fallback applied per source): held. D3 states the per-source fallback explicitly.
- D2 × D5 (validation at render for both sources against the rendered rows and spec): held.
- D2 × D6 (`#` barred in literal `label`/`confirm`, `#` doubled in values filled into `confirm`): held.
- D3 × D4 (a failed desktop load against the closure's failure rule): broken. See F3.
- D4 × D5/D6 (`{beam}` resolved only when used, the static values filled at render): weak. See F4.
- D4/D5 × D7 × D8 (placeholder and spec paths against the directory the run uses and the lym commands of the default set): broken. See F1.
- D5 × D7 (spec and rows rendered together, hash check on the confirmed path): held.
- D6 × D7 (`{strand}` read from `@lym_strand_name` at click, carried across `confirm-before`, checked against the name grammar): held.
- D2 `{strand}`-only-with-`window: agents` × D5 agent row × D7 refusal when no `@lym_strand_name`: held.
- D7 variable removal × D8 `lym tender pause` reading the caller from `LYM_STRAND_NAME`: held, as intended.
- D7 run directory × D8 CANCEL removing the task worktree, with the Windows template in force: broken. See F2.
- D5 refresh only at attach or boot × D4 a pair marked a beam after boot: held, since D5 states it.

**Acceptance criteria and constraints**
- D1, D2, D3, D5, D6 and D7 criteria: hold against the decisions they touch.
- D3 criterion (local layer refuses `buttons` in desktop and reed): holds. Both `CategoryExecutable` and `CategoryMachine` are refused by the local layer (`internal/configengine/layers.go` `ValidateLocal`).
- D8 criterion (CANCEL runs from the parent beam's worktree without the location refusal): broken in a subpath-anchored hub. See F1.
- psmux constraint: held.

**Findings**

F1 — BLOCKING, design. Location: D4 `{worktree}`/`{beam}`, D5 spec "the session's worktree path", D7 "runs `run` … in the spec's worktree", D8 `cd {beam} && lym batten cancel`, `lym reed window {worktree}`, D8 criterion.
- Summary: the record runs commands from, and fills placeholders with, worktree-root paths, but every lym verb only runs when its cwd equals the worktree's anchor. In a subpath-anchored hub, every default-set lym command and CANCEL's `cd {beam}` are refused with `ErrCwdOutsideAnchor`.
- D7 itself calls the session's start directory "the anchor". The pairing breaks: D7's click runs at the anchor, then the verb moves `run` to the spec's worktree root.
- Evidence:
  - `internal/lymcwd/lymcwd.go:71-83`: `Resolve` requires the cwd to equal the anchored directory exactly.
  - `internal/lymcwd/lymcwd.go:58-61`: `AnchorPath` is the worktree path joined with `AnchorRel`.
  - `internal/hubgeom/hubgeom.go` `reedGeometry`: `PaneCwd: l.AnchorPath()` but `WorktreeRoot: l.WorktreePath()`.
  - `internal/lymcwd/anchor.go:161-181`: per-worktree recorded anchors exist.

F2 — MEDIUM, design. Location: D7 (the run's directory is the session's own worktree) × D8 CANCEL on the Windows template.
- Summary: on Windows, a directory that is any process's current directory cannot be deleted. Clicking CANCEL in a task session leaves the `lym reed button` process, and the pwsh running `run`, with their cwd inside the task worktree that `lym batten cancel` is removing.
- pwsh's `cd` does not move its own process cwd. D7 expects only a gone client or worktree, but here the removal itself fails.
- Evidence:
  - `internal/hubgeom/hubgeom.go` `reedGeometry` (`PaneCwd` is the task anchor).
  - `pattern/PATTERN-pair-teardown.md`, "Batten bookend" (a producer that destroys a task worktree never runs from inside it).
  - D8: "The POSIX and Windows templates both ship the set".

F3 — LOW, consistency. Location: D3's last bullet × D4's failure bullet.
- Summary: D4's closure returns the machine's `buttons`, so a failed desktop load happens inside the closure. D4 says a closure failure shows `follow` alone, while D3 says a failed desktop load only drops the machine buttons. The record does not say which rule wins.
- Evidence: the decision record, D3's last bullet and D4's first and fifth bullets.

F4 — LOW, consistency. Location: D4, `{beam}` "resolved only when an entry of the session's row uses it" × D4's closure returning "the placeholder values and the machine's `buttons`".
- Summary: whether `{beam}` is used depends on the machine entries the same closure returns. The closure cannot both return `{beam}` eagerly with the other values and resolve it only when used, and the record does not say how the laziness reaches the closure.
- Evidence: the decision record, D4's first and fourth bullets.

Tool calls: none; every finding rests on context inherited from the handler's reading.
