```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths, the paths that already write, reset or read what the record changes.

**Paths traced, per decision**
- D1/D2, the board's detached sync `commitDirty` → `Bolt.Commit` (`commitWeftAt`, a whole-repo commit under `board.lock`): it picks up `heddle.json`. Survives.
- D2, the foreground scoped commits `Bolt.CommitWritten` (boardengine/ownerset, battencli/owner, orchcli/rules, configcli/hubwide, hubreconcile/walk): they stage exact paths under the same lock, so they never capture a half-written `heddle.json`. Survives.
- D2, `ensureLockfilesIgnored`: it ignores only `*.lock`, `*.swaplock` and the manifest. `heddle.json` stays tracked. Survives.
- D3, Claude Code's own instruction loader (default `claude-md-or-agents-md` mode) reads whether a `CLAUDE.local.md` is present. **Breaks**: see F1.
- D3, the Webster/Merriam base (`RunDeps.InstructionFiles` → `MerriamBase`, at a fresh partition and at `rebaseline`): it counts `CLAUDE.local.md`, which Claude now loads. It reads the new state correctly. Survives.
- D3/D5, `lym heddle render` and "took in" in a worktree that existed before landing, where `info/exclude` has no `/CLAUDE.local.md` line. **Breaks**: see F2.
- D4, `Prepare` is the only writer of `append-system-prompt.txt`. The resume line in `command.go` reuses the path, and an adopted resume runs `Prepare` again. Both match the record. Survives.
- D5, `unseedGitExclude` (from `unwire.go`, `anchorchange.go`, and `reconcile.go`'s stale-junction strip): it strips only `excludePatternFor(AnchorRel, junction)` and bare junction names, never a root `/CLAUDE.local.md` line. Survives.
- D5, `seedGitExclude`'s legacy bare-name drop: it touches junction names only. Survives.
- D6, the code-branch moves (`Fabric.Pull`'s warp arm, `ffWorktree` in `ReconcileOrigin`): covered as the record states. Survives.
- D7, `IsSeedCommit`: it matches only stencil and spec subtrees, so a `heddle.json` commit is never dropped as a seed commit. Survives.
- D7, `readOriginOwners`: it reads `boardFileName` at origin's tip and hands the bytes to `OwnerOf`. Reading the file name from the run ledger keeps that working. A missing ledger is unstated: see F3.
- D7, `readEndedSet` → `InstalledBoardReplay().Entries`: the run ledger replaces it. Survives.

**Findings**

**F1. BLOCKING, design. D3 (`claude-file` → `CLAUDE.local.md`): rendering `CLAUDE.local.md` stops Claude Code from reading the repository's own `AGENTS.md`.**
- In its default mode (`claude-md-or-agents-md`), Claude Code reads `AGENTS.md` only when no `CLAUDE.md` or `CLAUDE.local.md` exists in the working directory or above it, and `CLAUDE.local.md` counts.
- A repository with a team `AGENTS.md` and no `CLAUDE.md` loses that team text in every Claude session in a worktree heddle rendered into, lym-launched or not. That covers a Models-style repository that relies on `AGENTS.md`.
- The record handles the same "team instructions must not be hidden" problem for Codex (the `codex-file` row puts `AGENTS.md` first) but says nothing about it for Claude. The local Claude Code is 2.1.316, and the rule applies from 2.1.277.
- Possible fixes:
  - a `claude-file` render carries the repository's `AGENTS.md` when the worktree has no `CLAUDE.md`, as `codex-file` does;
  - or lym sets `pluginConfigs."cc-plugin-agents-md@builtin".options.instructionFiles: claude-md-and-agents-md` in its `--settings` file, which covers lym-launched sessions only.
- Evidence: Claude Code memory docs (code.claude.com/docs/en/memory), the "AGENTS.md" table rows "An `AGENTS.md` and a `CLAUDE.md` or `CLAUDE.local.md` … → Your `CLAUDE.md` files only", the "Count, so Claude reads them instead of `AGENTS.md`" bullet, and the note "Because `CLAUDE.local.md` counts, adding one … stops Claude from reading `AGENTS.md`"; `claude --version` gives 2.1.316.

**F2. MEDIUM, design. D3's exclude-list refusal and Open risks' "until `lym heddle render` runs": in a worktree made before landing, `render` writes an un-excluded file.**
- D3 refuses a target whose file is not on fabric's exclude *list*, which is D5's config key, not the seeded `info/exclude` line.
- D5 seeds lines only when fabric creates or re-wires a worktree, or on `lym fabric reconcile`.
- So the record's own way forward for an existing worktree (`lym heddle render`, or a sync take-in) writes `CLAUDE.local.md` or `AGENTS.override.md` with no exclude line. The file shows as untracked in warp, against D3's own aim "never shows as an untracked file", and dirties the worktree's status.
- Possible fixes:
  - the refusal checks the seeded line through a fabric read;
  - or the way forward names `lym fabric reconcile` before `render`.
- Evidence: decision-record.md D3 "It refuses a target whose file is not on fabric's exclude list (D5)", D5 "whenever it creates or re-wires a warp worktree (`Topology.Add`, `AddLocal`, `CloneHub`, `lym fabric reconcile`)", Open risks line 4; `internal/fabricengine/junction.go:703` (`seedGitExclude` runs only from the wiring path at `junction.go:138`).

**F3. LOW, consistency. D7: the origin owner read with no run ledger registered is unstated.**
- D7 states the failure for a replay and an ended-set read with no ledger, but not for the owner read.
- `readOriginOwners` today reads `boardFileName` itself and returns nothing only when `OwnerOf` is nil. Under D7 it needs the ledger's file name, so with `OwnerOf` set and no ledger its behaviour is undefined.
- Evidence: `internal/fabricengine/originreconcile.go:823-830`; decision-record.md D7, the last bullet.

**Tool calls**
1. `boardengine/sync.go` `commitDirty`, `ensureLockfilesIgnored`: whether the detached sync commits `heddle.json`; the manifest held only `Sync`.
2. `junction.go` `unseedGitExclude` body and `reconcile.go:880-912`: whether an existing unseed strips the new root lines; the manifest held only the callers' line numbers.
3. `websterengine/runlevel.go`, `webstercli/rebaseline.go`: how Merriam's base reads instruction files; the manifest held only the lookup site.
4. `bolt.go` `Bolt.Commit`/`CommitWritten`, `stencilcommit.go` `IsSeedCommit`: the commit scope and seed-commit classification for a `heddle.json` commit.
5. `originreconcile.go:815-870` and the `CommitWritten` users: how the owner read takes `board.json`, and which foreground commits share the lock.
