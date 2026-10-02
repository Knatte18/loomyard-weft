# Decision record — operator-surface-fixes

## Goal

Deploying lyx and opening worktrees in VS Code leave every worktree runnable and clean, with no manual step after either.
Fixes #339 (a deploy's new config key makes loom refuse until reconcile), #340 (opening the prime in VS Code while `lyx orch` hosts the hub collapses the orch pane and stalls every cycle) and #334 (`lyx ide spawn` writes a lyx-managed block into a pair's tracked `.gitignore`).

## Scope

In:

- `internal/configengine`: a config file missing template keys loads with those keys filled from the template, for every module that loads through `Load` or `LoadOrTemplate`.
- `internal/reedcli` / `internal/reedengine`: a `reed add` flag that skips the add when a named live strand already exists in the worktree.
- `internal/vscode` / `internal/ideengine`: the interactive folder-open chain skips adding `claude` when the worktree hosts a live `orch` strand;
  the interactive `tasks.json` is regenerated on every `lyx ide spawn`;
  `lyx ide spawn` keeps `.vscode/` out of git through `.git/info/exclude`, never a `.gitignore`.
- Deleting `internal/gitignore`, whose only caller is the `.gitignore` write removed above.
- The repo `.gitignore`'s `mill-managed` block renamed to a plain section.
- Template comments and docs that tell operators to run `lyx config reconcile` to adopt a new key.
- `CONSTRAINTS.md`, `docs/overview.md`, `docs/shared-libs/configengine.md` and the reed package doc, in the same commit.
- Deleting CLAUDE.md's TEMPORARY items for #339, #340 and #334.

Out:

- Seed-only config modules (`models`, `burler`): they never load through `configengine`'s key check.
- The board repo's own `.gitignore` written by `internal/boardengine`: that repo is lyx-owned, not a user's code repo.
- Changing reed's `window-size` policy or the orch idle probe.
- Making `lyx config reconcile` obsolete: it still materializes absent files, prunes removed keys and writes new keys to disk.
- Pruning the mill-era entries inside the renamed `.gitignore` section.

## Decisions

### D1 — A missing template key resolves to its template default, in memory

Decision: `configengine`'s shared `load` body stops refusing when a present config file lacks template keys.
It merges the missing leaf keys from the template into the file's document in memory, logs one `logger.Info` line naming the module, the file and the filled key-paths, and resolves the merged document through `envsource.Build` and `yamlengine.Resolve` exactly as before.
It never writes the config file.
Both `Load` and `LoadOrTemplate` get this, since they share the body.
Bound: it fills only key-paths present in the module's embedded template, with the template's own value.
It still refuses on an absent `_lyx/`, an absent file under `Load`, a read error, unparseable YAML, and an env marker without a default (`${env:NAME}`) whose variable is unset.
A key present with any value, empty included, keeps that value; a present list keeps its length (the existing "a template list is a default, not a minimum length" rule).
Rationale: every template key carries a default, so a key a deploy adds needs no operator decision; refusing turned every deploy into a reconcile sweep across worktrees (#339).
Fixing it in the shared loader covers every strict module (`fabric`, `board`, `loom`, `landing`) and the degrading ones, which hit the same check whenever their file is present.

### D2 — `reed add --unless-name <name>` skips the add when that strand is live

Decision: `lyx reed add` gains `--unless-name <name>`.
When the worktree's session holds a strand matching `<name>` (matched as role segment or full name, the same rule as `--name`) whose pane is alive, the add is a successful no-op: it creates no strand, changes no focus and applies no layout, and its JSON envelope reports the skip.
A recorded strand whose pane is dead does not count, so the add proceeds.
Bound: the flag can only suppress an add; it never removes, resizes or refocuses an existing strand.
Reed stays role-agnostic: the name is the caller's.
Rationale: the 3-row collapse is reed's stacking rule, not the window size.
The folder-open `reed add --if-absent --name claude` adds a strand below orch, orch collapses to `collapsed_rows`, and the orch watcher's idle probe, which reads the captured pane, can no longer see Claude's idle prompt and fails closed, so every cycle stalls.
Not adding the strand removes the cause.

### D3 — The interactive folder-open chain passes `--unless-name orch`

Decision: `vscode.WriteConfig`'s interactive chain adds `--unless-name` with `agentname.RoleOrch` to its `reed add claude` row.
`reed up` and `reed attach` stay in the chain unchanged, so opening the prime attaches VS Code to the existing session with the orch strand as the only, full-height strand.
Rationale: the folder-open task's intent is to give the operator an agent in the worktree; a live orchestrator already is one.

### D4 — The interactive `tasks.json` is lyx-owned and rewritten on every `lyx ide spawn`

Decision: `Spawn` writes the interactive `tasks.json` on every call, overwriting an untracked one, as `SpawnDriven` already does for the attach-only chain.
`settings.json` keeps its write-only-when-absent rule.
A `tasks.json` tracked in the repo is left alone with one `logger.Warn`, as in `SpawnDriven`.
Bound: it overwrites only the untracked `.vscode/tasks.json` at the spawned worktree's anchor; operator edits to that file are lost on the next spawn, the same contract the hub workspace file already has.
Rationale: an existing prime keeps its old chain forever under write-when-absent, so D3 would never reach it; the overview's documented "delete tasks.json and re-run spawn" upgrade is the manual step this task removes.

### D5 — `lyx ide spawn` keeps `.vscode/` out of git through `info/exclude`

Decision: `Spawn` uses the same path `SpawnDriven` uses: `fabricengine.PathTracked` on the anchor's `.vscode/tasks.json`, then `fabricengine.ExcludeAnchoredDir(worktreeDir, AnchorRel, ".vscode")`, which writes nothing when `git check-ignore` already reports `.vscode/` ignored.
`vscode.WriteConfig` no longer touches `.gitignore` in either chain.
`internal/gitignore` is deleted with its tests, since nothing else calls it.
Rationale: a tracked `.gitignore` edit dirties the pair and rides into its commits (#334); `info/exclude` is untracked and already the mechanism for junctions and driven pairs.

### D6 — New invariant: lyx never writes a `.gitignore` in a code worktree

Decision: `CONSTRAINTS.md` gains the rule that lyx's own code never writes a `.gitignore` in a warp or prime worktree; every ignore lyx needs there goes to `.git/info/exclude` through `fabricengine`.
The board repo, which lyx owns, is exempt.
It sits beside the Fabric Git Invariant's existing junction-exclusion bullet, which it generalizes.
Rationale: #334 is the second instance of the rule the junction bullet states for junctions alone; stating it once for every writer stops the next one.

### D7 — Config Strictness Invariant records D1

Decision: the Config Strictness Invariant in `CONSTRAINTS.md` gains a bullet: a key missing from a present config file resolves to its template default and is logged; `Load` and `LoadOrTemplate` differ only on an absent `_lyx/` or file.
Rationale: the invariant decides which loader a caller adopts and must not imply a missing key still refuses.

### D8 — Reconcile hints describe the new behavior

Decision: every template comment, error string and doc sentence that says an existing hub must run `lyx config reconcile` to adopt a new key is rewritten to say the key takes its default until reconcile writes it.
The absent-file error under `Load` keeps its "run `lyx config reconcile`" hint, since reconcile still materializes absent files.
Rationale: leaving the old wording keeps operators running the sweep D1 removes.

### D9 — The repo `.gitignore`'s mill block becomes a plain section

Decision: the `# === mill-managed (regenerated by mill-setup) ===` and `# === end mill-managed ===` markers are replaced by one plain comment heading the same entries; the entries stay.
Rationale: mill is gone and nothing regenerates the block, so the markers lie; keeping the entries keeps leftover mill-era directories in existing worktrees out of `git status`.

### D10 — CLAUDE.md TEMPORARY items go

Decision: the TEMPORARY items for #339, #340 and #334 are deleted from CLAUDE.md in the same commit; the items for other issues stay.

## Constraints

- `CONSTRAINTS.md` is authoritative; D6 and D7 land in it in the same commit as the code.
- Config Strictness Invariant: each caller keeps the loader it adopts today; D1 changes the shared body, not the split.
- Fabric Git Invariant: the `info/exclude` write goes through `fabricengine` (`ExcludeAnchoredDir`, which holds `mutateGitExclude`'s lock), never raw file I/O from `ideengine` or `vscode`.
- `internal/fslink` and Windows: nothing here creates links; the reed flag must behave the same on the Windows reed backend.
- Docs land in the same commit: `docs/overview.md` (ide bullet: regenerated `tasks.json`, `--unless-name orch`, the removed manual-upgrade sentence; reed's `add` flags if listed), `docs/shared-libs/configengine.md` (flow step 3 and Key properties), the reed package doc for `--unless-name`.
- Semantic line breaks in every `.md` file touched.
- No `claude -p`; nothing here spawns agents.
- `lyx` builds with cgo; tests run under `go test` as usual.

## Auto-mode assumptions

- No template key is meant as "the operator must set this": every template value, empty string included, is a usable default. Checked against every module template; none uses a bare `${env:NAME}` marker today.
- `logger.Info` (durable trace) is the right level for a filled key; `lyx config reconcile` (dry run) remains the operator-facing report of pending keys. A Warn would repeat on every lyx call in every worktree until reconcile runs.
- `window-size latest` is already pinned on every reed session (since August), so the board's "reed sizes to the latest client" half of #340 needs no code; the collapse comes from strand stacking (D2).
- The prime is opened through `lyx ide spawn <prime>`, which is what regenerates `tasks.json` under D4.
- The orch strand's reed name resolves from `agentname.RoleOrch` through the same role-segment matching `--name` uses.
- Mill-era entries in the repo `.gitignore` are kept, not pruned (D9).

## Open risks

- A prime reopened from VS Code's recent list after a deploy, without a `lyx ide spawn`, runs whatever chain its `tasks.json` held at the last spawn; worktrees spawned before this task land need one more spawn to pick up D3.
- Any other strand added to the prime while orch runs (an operator's own `reed add`) still collapses the orch pane and stalls its idle probe; this task removes only the folder-open trigger.
- A config whose author deliberately removed a key to mean "unset" now gets the template default instead of a refusal; no module documents removal as meaningful today.

## Acceptance criteria

- A `loom.yaml` (and a `fabric.yaml`, `board.yaml`, `landing.yaml`) missing one template key loads; the loaded value equals the template default; the file on disk is byte-identical afterwards; the trace carries one Info line naming the module and key-path.
- Nested keys: a `reed.yaml` missing `selvage.height_rows` loads with the default; a missing whole mapping (`status_line`) loads with its defaults.
- A present list keeps its length (an emptied `require_pr_to_base: []` stays empty); a present empty string keeps its empty value.
- An empty but present config file loads as the template.
- Still refused: absent file under `Load`, absent `_lyx/` under `Load`, unparseable YAML, a filled key whose template value is `${env:NAME}` with `NAME` unset (unit test with a fixture template).
- `lyx reed add --unless-name orch --if-absent --name claude --cmd <x>` in a worktree with a live `orch` strand: exit 0, envelope reports the skip, strand list unchanged, orch pane height and focused pane unchanged (real-tmux integration test).
- Same command with no `orch` strand, or with an `orch` strand whose pane is dead: the `claude` strand is added as today.
- `vscode.WriteConfig` interactive output: the `reed add claude` row carries `--unless-name orch`; no `.gitignore` is created or modified in either chain.
- `ideengine.Spawn` on a task pair in a git repo with a tracked `.gitignore`: `git status --porcelain` is empty afterwards; `.vscode/` is ignored through `info/exclude`, or nothing is written when the repo already ignores it.
- `Spawn` overwrites an untracked `tasks.json` holding an old chain; leaves a tracked `tasks.json` untouched; leaves an existing `settings.json` untouched.
- `internal/gitignore` no longer exists and `go build ./...` and `go vet ./...` pass.
- The repo `.gitignore` has no `mill-managed` marker; `git status` in this worktree is unchanged by the edit.
- CLAUDE.md has no TEMPORARY item for #339, #340 or #334; `CONSTRAINTS.md` carries D6 and D7.
- `go test ./...` passes.

## Notes for the plan writer

- Loader: `internal/configengine/config.go`'s `load`; `yamlengine.Reconcile` already merges template and existing (returning `added`), which may serve the in-memory fill; check that its list handling keeps a present list whole before reusing it.
- Templates with reconcile-adoption wording: `internal/reedengine/template_posix.yaml` and `template_windows.yaml` (`debug_log`, `collapsed_rows`); search for "config reconcile" across `internal/**/template*.yaml`, `docs/` and `contracts/` for more.
- Reed add flags live in `internal/reedcli/add.go`; `--if-absent`'s strand classification is in `internal/reedengine/strand.go` (`classifyIfAbsent`), a natural neighbour for the liveness check.
- `ideengine.SpawnDriven` already holds the tracked-check + `ExcludeAnchoredDir` sequence D5 reuses.
- The orch idle probe that stalls is `shuttleengine.Runner.SessionIdle`, which captures the pane; it explains the stall and is not changed.
