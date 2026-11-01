```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of the record's decisions that act on the same state, then each acceptance criterion and constraint against the decisions it touches.

**Pairs checked**
- D1×D2 (store and verbs, unknown keys on `set`): held.
- D1×D3 (`targets` name rows of the target table): held.
- D1×D8 (replay writes through D1's types and refuses an entry that fails D1's validation): held.
- D2×D7 (a heddle write commits through the board's sync, so one commit can carry both files): held. `commitDirty` calls `bolt.Commit` with no pathspec, so the sync commits `heddle.json` too.
- D2×D9 (where the write lock sits): broken. See finding 1.
- D3×D5 (D3 refuses a target that is not on the exclude list; D5 seeds the lines only when a worktree is created or re-wired): broken for worktrees that exist before this lands. See finding 2.
- D3×D5 (D5 never unseeds a line; D3 removes a stale marked render): broken in a small case. See finding 4.
- D3×D6 ("made" renders after D5's lines are seeded; "took in" re-renders by D3's re-render paths): held.
- D3×D6 (a fast-forward over a commit that adds a tracked `AGENTS.override.md`; git overwrites the ignored render, after which heddle skips the file): held, and Open risks covers it.
- D4×D6 (a re-read on an adopted resume against "rules do not shift mid-run"): broken. See finding 3.
- D4×D1/D3 (`claude-prompt` read through the shuttle seam at `Prepare`): held.
- D5×D6 (`lym fabric reconcile` seeds the lines but does not fire "made"): held. The record does not ask "made" to fire there.
- D6×D7 ("took in" never fires for a Bolt branch, so a `heddle.json` change reaches no live worktree): held.
- D7×D8 (a mixed commit, each replayer merging its own paths, set-aside tags per replayer): held.
- D7×D9 (replayers claim Bolt-root-relative paths, and the board computes its own): held.
- D9×Constraints (an exported constant in `lymcwd` leaves its import set unchanged; `runproc` may import `lymcwd`): held.
- D9×Scope Out (`fabricengine.BoardDir` stays): broken. See finding 1.
- Each acceptance criterion against its decisions: held, apart from the findings below.

**Findings**

1. MEDIUM · consistency · Scope "Out" bullet 6 against D9 bullet 3 and D2.
   - Scope keeps "Board paths in fabric outside the records replay, the ended set and the owner read, such as `fabricengine.BoardDir` and the write lock's file name" out of the task.
   - D9 bullet 3 requires that the Bolt write lock, the `_lym` stencils, specs and config, and the `.lym` tree "take [their] place from `BoltDirName`, never from `BoardDirName`".
   - Today the write lock's path is `filepath.Join(BoardDir(hub), BoardWriteLockFile)` (`BoardWriteLockPath`, internal/fabricengine/junctionnames.go:92-94), and the board's sync takes it as `filepath.Join(boardPath, writeLockFile)` (internal/boardengine/sync.go:66).
   - Fabric locates the Bolt checkout through `BoardDir` (hubpull.go:74, anchormarker.go:175, hubworkspace.go:304, where the workspace folder is labelled "Bolt").
   - So D9 needs those `BoardDir` sites moved onto `BoltDirName`, and Scope's line forbids it. The plan writer cannot meet both as written.
   - D2's lock, "the one file `fabricengine.BoardWriteLockFile` names", agrees with both only because the two names hold the same value today.
   - Fix: narrow Scope's line to the lock's file name, and state in D9 that `BoardDir` sites meaning the Bolt checkout move to a Bolt-dir helper.

2. MEDIUM · design · D3's off-list refusal against D5's seeding times and the 4th Open risks line.
   - D3 refuses a target whose file is not on fabric's exclude list, "so a render never shows as an untracked file in warp". That check reads the config list.
   - D5 writes the `info/exclude` lines only when a warp worktree is created or re-wired: `Topology.Add`, `AddLocal`, `CloneHub`, `lym fabric reconcile`.
   - Open risks tells the operator to run `lym heddle render` in each worktree that exists when this lands.
   - Until fabric next creates or re-wires a worktree of that repository, no line is seeded. That render then writes `CLAUDE.local.md` as an untracked file in warp, which is what D3's guard exists to prevent.
   - Fix: D3's guard checks the seeded line, or `lym heddle render` / D6 seeds through fabric first, or D5 names when existing repositories get the lines.

3. LOW · consistency · D4's adopted-resume sentence and its Auto-mode assumption, against the operator's no-mid-run-shift rule (recorded in D6's rationale, from the board entry) and support-log's Rejected alternatives.
   - D4: "a resume that runs `Prepare` again, as an adopted session does, reads heddle afresh". So an adopted session's prompt passages change mid-run after a `heddle.json` edit.
   - The board entry says a `heddle.json` change "is not pushed into live worktrees, so an agent's rules do not shift mid-run".
   - support-log rejects "Re-reading heddle on resume ... since a resumed session's system prompt would change under it", the outcome D4 accepts for adoption.
   - The record states the rule two ways without reconciling them. Fix: either bound D4's re-read explicitly as the accepted exception to that rule, or drop the contradicting rationale in the rejected alternative.

4. LOW · design · D3's stale-render removal against D5's never-unseed and D3's own off-list refusal.
   - When an operator takes a path off `warp_exclude`, D3 refuses that target.
   - So the removal of a stale marked render never runs for it, and heddle's old file stays in the worktree. D5 leaves its exclude line, so the file stays hidden but is never cleaned up.
   - The record states no bound for this case.

**Tool calls**
- 1: `commitDirty` in internal/boardengine/sync.go, to check whether the board's sync commits `heddle.json` (D2×D7). The manifest held `Sync` but not the staging scope of `commitDirty`.
