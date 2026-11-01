```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each Decision's mechanism on one concrete case and checked the end state against the Decision's stated purpose.

**List**
- D1, case: `set` adds entry `b` beside entry `a`, which carries an unknown key. The types keep unknown keys and the rewrite goes through them. **Purpose met.**
- D2, case: `lym heddle set x` saves under `board.lock`, then `SpawnSync`. `commitDirty` calls `bolt.Commit("board sync")` on the whole checkout, so `heddle.json` is committed and pushed. **Purpose met.**
- D3, case 1: a repository with a root `AGENTS.md` and no `CLAUDE.md`, plus one `claude-file` entry. Heddle writes `CLAUDE.local.md`. Claude Code, by default, then stops reading the team's `AGENTS.md`. **Purpose missed** (finding 2).
- D3, case 2: `lym heddle render` in a worktree that existed before landing, before any Add, CloneHub or reconcile. The config list holds `CLAUDE.local.md`, so the refusal passes and the file is written. `info/exclude` has no `/CLAUDE.local.md` line yet, so the file shows as untracked. **Purpose missed** (finding 3).
- D4, case: a fresh launch with two `claude-prompt` entries. `Prepare` writes lym's one-line sentences, a blank line and the passages, and the file rides `--append-system-prompt-file` on both lines. **Purpose met.**
- D5, case: `Topology.Add` of a pair whose anchor is `sub/`. `mutateGitExclude` resolves the shared `info/exclude` and the root-anchored `/CLAUDE.local.md` is added once. **Purpose met.**
- D6, case: `Fabric.Pull` fast-forwards warp over a commit changing `AGENTS.md`. The paths are root-relative and match D3's re-render path, so `AGENTS.override.md` is re-rendered. **Purpose met.** On "made", the event's payload is never stated (finding 4).
- D7, case: one board-sync commit carrying ended run R's `board.json` change and a `heddle.json` change. The ledger part is skipped, the original is tagged `set-aside/R/<sha>`, and heddle applies its part. **Purpose met.** The empty-slug tag fallback still names the board (finding 5).
- D8, case: `targets` changed on both sides. Origin's value wins and is reported lost, and the unknown key is kept through D1's types. **Purpose met.**
- D9, case: `bolt-dir-rename` changes `lymcwd`'s constant. To read it, `runproc.HubOwner` must import `lymcwd`, which `runproc`'s closed import set forbids. **Purpose missed** (finding 1).

**Findings**

1. **BLOCKING, design.** D9, second bullet: setting `runproc.HubOwner` from the `lymcwd` constant breaks PATTERN-leaf-packages.
   - `runproc` may import only the standard library, `internal/proc`, `internal/state` and `internal/logger` (`pattern/PATTERN-leaf-packages.md:84`).
   - Its current imports are only those (`internal/runproc/*.go` import lines).
   - `## PATTERN changes` does not declare a change to that entry.
   - D9's own rationale rules out `fabricengine` as the source because of an import edge, but never checks the `lymcwd` edge. The board entry's "set from `BoltDirName`" is ruled out the same way.
   - Evidence: `decision-record.md:174`, `pattern/PATTERN-leaf-packages.md:82-85`.

2. **BLOCKING, design.** D3: rendering `CLAUDE.local.md` hides a repository's own `AGENTS.md` from Claude sessions, and the record never says so.
   - Under Claude Code's default `claude-md-or-agents-md`, a `CLAUDE.local.md` in the working directory or above "count[s]", so Claude reads the `CLAUDE.md` files only and drops `AGENTS.md`. The docs warn about exactly this case.
   - Claude Code 2.1.316 is installed here, past the 2.1.277 floor for reading `AGENTS.md`.
   - This defeats the board entry's aim that "the team's instructions are not hidden". D3 handles that aim for Codex but not for Claude.
   - D3's evidence sentence cites the same docs page and still misses the interaction.
   - Evidence: code.claude.com/docs/en/memory, "Your repository has" table rows for `AGENTS.md`, the `CLAUDE.local.md` note and the Project instructions table; `decision-record.md:84,96`.

3. **MEDIUM, design.** D3's last bullet and D5: the refusal tests the config list, not the seeded exclude line.
   - D5 seeds the lines only on Add, AddLocal, CloneHub or reconcile.
   - Open risks sends existing worktrees to `lym heddle render`, and a take-in from `Fabric.Pull` also renders without seeding.
   - So the first render on a hub where none of those has run leaves `CLAUDE.local.md` and `AGENTS.override.md` showing as untracked in warp. That misses D3's stated purpose, "a render never shows as an untracked file in warp".
   - Evidence: `decision-record.md:94,115,206`; `internal/fabricengine/junction.go:703` (`seedGitExclude` is called only from the wiring path, `junction.go:138`).

4. **LOW, design.** D6: the "made" event names no payload, but heddle needs absolute paths.
   - Under the record's Constraint and PATTERN-cwd-resolution, heddle imports no `lymcwd`, so the event must hand it the warp root and the Bolt checkout.
   - The "made" event carries nothing stated, and "took in" carries only the changed paths.
   - So the step's input, as stated, does not give heddle what it needs to find `heddle.json` and the warp root.
   - Evidence: `decision-record.md:127-128,185`.

5. **LOW, consistency.** D7's closing sentence says no board name is left in fabric's replay, but `setAsideTag` still falls back to the slug `"board"`.
   - A run-ledger set-aside with an empty slug would still be tagged `set-aside/board/…`.
   - D7 renames only the non-ledger case.
   - Evidence: `internal/fabricengine/replay.go:487-491`; `decision-record.md:153,158`.

**Tool calls**
- `boardengine` `commitDirty`: the manifest held `Sync` but not what the commit step stages (needed for D2).
- `runproc` import lines and `PATTERN-leaf-packages.md` grep: the manifest held no `runproc` import set (needed for D9).
- `fabricengine` `tagSetAside` and `setAsideTag`: the manifest lacked the tag naming (needed for D7).
- `PATTERN-leaf-packages.md` `runproc` section: to read `runproc`'s closed import set exactly (needed for D9).
