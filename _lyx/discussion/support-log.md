# Support log — board-config-per-worktree

## Interview

Autonomous mode: no operator answered, so each turn below is an exploration finding and the pick it led to.

1. Board entry read: `board.yaml` is per worktree while the board is per hub; the proposal is to move it beside `fabric.yaml` in `_board/_lyx/config/`, and to check which other config files are repo-wide before generalizing.
   Related notes read: `config-overlay` (general two-layer default), `config-reaches-parent` (task config changes lost at landing), `board-viewer` (README rendering; unaffected).
2. Code survey: board config is loaded by `boardcli`'s pre-run, `loomcli`'s `openHubBoard` and `ideengine.Menu`, each from the worktree anchor, while the board data path is already the hub's `_board`.
   `fabric.yaml` is the precedent: reconciled at `BoardDir` by `fabric clone` and `fabric reconcile`, skipped by name in `configsync.ReconcileAll`.
   `_board` is the `weft:main` checkout, committed through `fabricengine.Bolt` by board sync.
3. Other modules surveyed (`orch`, `reed`, `landing`, `logger`, `loom`, `shuttle`, plus `models`, `burler`, `batcher`, `webster`): each tunes the run in its own worktree or is read only from the prime, so only `board` joins `fabric` (D1).
4. Found that `lyx config fabric` edits a per-worktree `fabric.yaml` nothing reads; folding hub-wide handling into the registry fixes it alongside `board` (D2, D5).
5. Migration: an absent hub `board.yaml` needs a source; the prime's copy holds the live vocabulary, a pair's copy is the stale one this task removes (D4).
6. Dead per-worktree copies: left in place they invite edits that do nothing, the `board-model` D13 failure; `lyx config reconcile --apply` deletes them once the hub copy exists (D6).

## Rejected alternatives

- **Keep per-worktree `board.yaml` and sync copies from the prime at pair creation or landing.** Copies still drift between creation and teardown, which is the reported bug.
- **Build the general overlay (`config-overlay`) now, with board as its first user.** The board has no legitimate per-worktree override; an overlay would let a pair shadow the hub vocabulary again. The overlay stays its own note.
- **Move other modules (`orch`, `reed`, `landing`) hub-wide as well.** Their values are per-worktree run tuning or prime-only; making them hub-wide removes per-task tuning nobody asked to remove.
- **Hard-code `"board"` beside `"fabric"` in `configsync` and `configcli`.** Spreads the hub-wide list across packages; a registry field declares it once.
- **Seed the hub file from the running worktree's copy.** A pair running reconcile first would install the stale vocabulary.
- **Leave hub-wide edits uncommitted for the next board sync to sweep up.** The commit would be mislabelled "board sync" and the edit lost if no board write follows before a re-clone.
- **Re-render README on every `lyx config board` edit.** Not asked for; the next board write renders with the hub config.
- **`LoadOrTemplate` for board config, so a missing hub file degrades to the template.** Silently drops the hub's custom types and labels, the same symptom as the bug.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Which config modules are hub-wide? Auto-pick: `fabric` and `board` only (D1). Resolved.
- Q2 Where is the hub-wide set declared? Auto-pick: a field on `configreg.Module` (D2). Resolved.
- Q3 Strict or degrading load of the hub `board.yaml`? Auto-pick: strict, refusal naming `lyx fabric reconcile` (D3). Resolved.
- Q4 Source for the first hub `board.yaml`? Auto-pick: the prime's per-worktree copy, else the template, only when the hub file is absent (D4). Resolved.
- Q5 How does a `lyx config` edit of a hub-wide module reach git? Auto-pick: commit and push in `_board` through `Bolt` under the board write lock (D5). Resolved.
- Q6 What happens to per-worktree copies? Auto-pick: `lyx config reconcile --apply` deletes them once the hub copy exists (D6). Resolved.
- Q7 Does this need a PATTERN entry? Auto-pick: yes, `PATTERN-hub-wide-config` (D7). Resolved.
- Q8 Re-render README on config edit? Auto-pick: no, out of scope. Resolved.
- Q9 How does the seeded hub file reach git, and what if there is no prime copy? Auto-pick: committed through `Bolt` under the board write lock like D5; template seed with a warning in reconcile, silent in clone (D4). Resolved.
- Q10 What bounds a bad `lyx config board` edit and a failed commit? Auto-pick: strict-load validation before commit, write and commit under the write lock, a failed commit leaves the file on disk with an error saying so (D5). Resolved.
- Q11 Can `--apply` drop an unmerged per-worktree edit? Auto-pick: no, a copy with content the hub file lacks is reported divergent and kept (D6). Resolved.
