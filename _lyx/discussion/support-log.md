# Support log — operator-surface-fixes

## Interview

Autonomous run: no operator answered; every answer below is an auto-pick grounded in the code, recorded in the Question ledger.

1. Scope of #339's fill.
   The board says "loom fills a missing key"; the refusal lives in `configengine`'s shared `load`, which every strict module and every degrading module with a present file goes through.
   Picked the shared loader, so `fabric`, `board` and `landing` stop refusing too.
2. Which keys "have a default".
   Read every module template: each key carries a value, empty string included, and none uses a bare `${env:NAME}` marker.
   So every template key has a default; a bare env marker keeps refusing through `Resolve` when its variable is unset.
3. Write the fill to disk or not.
   `_lyx/config/` is weft content; a loader write would dirty the weft in every worktree on every call.
   Picked in-memory only, with reconcile still the way to persist.
4. Log level for a filled key.
   Picked Info: a Warn would repeat on every lyx call in every worktree until reconcile.
5. #340's cause.
   `window-size latest` has been pinned since 2026-08 (`8b14f32ba`), so the board's first fix half already holds.
   The 3-row figure is `collapsed_rows`: the folder-open `reed add --name claude` stacks a strand under orch.
   The stall follows from `shuttleengine.Runner.SessionIdle`, which captures the pane; a 3-row pane hides Claude's idle prompt and the probe fails closed.
6. Where the "skip when orch is live" check lives.
   Picked a generic `reed add --unless-name <name>` flag, with the folder-open chain passing `orch`; reed stays role-agnostic.
7. How an existing prime gets the new chain.
   Interactive `tasks.json` is written only when absent today, and the overview documents a manual delete-and-respawn upgrade.
   Picked: `tasks.json` is lyx-owned and rewritten on every spawn, matching the attach-only chain and the hub workspace file.
8. #334 mechanism.
   `SpawnDriven` already does tracked-check plus `ExcludeAnchoredDir`; `Spawn` adopts the same path and `WriteConfig` drops its `gitignore.Ensure`.
   `internal/gitignore` then has no caller and is deleted.
   `boardengine`'s `.gitignore` writer works in `_board`, the weft's `main` worktree, not a warp worktree, and stays.
9. Repo `.gitignore` mill block.
   Picked renaming the markers to one plain comment and keeping the entries.
10. Invariants.
    The junction-only exclusion bullet extends to "lyx never writes a `.gitignore` in a warp worktree"; the Config Strictness Invariant records the fill.

## Rejected alternatives

- #339: deploy (`update-plugins.sh`) runs `lyx config reconcile --apply` across the hub's worktrees.
  Rejected: it writes weft config in pairs with runs in flight, misses worktrees deploy does not know about, and leaves the loader's refusal in place for the next path that skips deploy.
- #339: loader writes the filled keys to disk.
  Rejected: dirties weft `_lyx/config/` in every worktree from a read path, and races concurrent lyx processes.
- #339: fill only in `loomengine`.
  Rejected: `fabric`, `board` and `landing` hit the same refusal through the same body; a loom-only fix leaves #339 open for them.
- #339: Warn instead of Info on a filled key.
  Rejected: repeats on every invocation everywhere until reconcile; the dry-run reconcile already reports pending keys.
- #340: change `window-size` or condition it on client count.
  Rejected: already `latest`, and the reed package doc records why the other policies are worse; the collapse is not a window-size effect.
- #340: hard-code an orch check inside `reed add --if-absent`.
  Rejected: couples reed to a role name and silently changes `--if-absent`'s meaning for every caller.
- #340: add the folder-open `claude` strand with `--anchor hidden` when orch is live.
  Rejected: still creates an unwanted session in the prime; skipping is simpler.
- #340: a hidden `lyx ide` verb that the chain calls, so future chain logic needs no regeneration.
  Rejected: a new verb plus runtime `claude` resolution for a one-flag change; D4's regenerate-on-spawn already carries future chain changes.
- #340: make the orch idle probe work in a collapsed pane.
  Rejected: out of the issue's scope and the probe's fail-closed behaviour is deliberate.
- #334: skip the `.gitignore` write when `.vscode/` is already ignored, keep writing otherwise.
  Rejected: still writes a tracked file in repos that do not ignore `.vscode/`.
- `.gitignore`: prune the mill-era entries while renaming.
  Rejected: leftover mill directories in existing worktrees would surface as untracked.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): fill at `configengine`'s shared loader or in loom only? → shared loader (D1). Covers every module that hits the same check.
- Q2 (resolved, auto-pick): persist filled keys? → no, in memory only (D1). Avoids weft writes from a read path.
- Q3 (resolved, auto-pick): log level? → Info (D1). Avoids per-call noise; reconcile dry-run reports pending keys.
- Q4 (resolved, auto-pick): does an empty present file load as the template? → yes (acceptance criteria). It is a present file with every key missing.
- Q5 (resolved, auto-pick): is the board's `window-size latest` fix needed? → no code; already pinned since `8b14f32ba` (assumptions).
- Q6 (resolved, auto-pick): where the orch skip lives? → generic `reed add --unless-name` (D2) passed `orch` by the chain (D3).
- Q7 (resolved, auto-pick): does a dead orch strand suppress the add? → no, only a live pane does (D2).
- Q8 (resolved, auto-pick): how existing worktrees get the new chain? → rewrite untracked `tasks.json` on every spawn (D4).
- Q9 (resolved, auto-pick): #334 mechanism? → reuse `SpawnDriven`'s `ExcludeAnchoredDir` path; delete `internal/gitignore` (D5).
- Q10 (resolved, auto-pick): new invariant? → lyx never writes a `.gitignore` in a warp worktree; `boardengine`'s writer in `_board` is named as the one weft-side writer (D6).
- Q11 (resolved, auto-pick): mill block entries? → keep entries, rename markers (D9).
- Q12 (open, residual): a prime reopened from VS Code's recents without a spawn keeps its old chain until the next spawn; recorded under Open risks.
- Environment: no tool failed during this session.
