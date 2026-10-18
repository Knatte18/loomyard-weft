# Decision record — hub-workspaces

## Goal

The operator opens a hub through one VS Code workspace file that sits in the hub directory, lists the prime first, Bolt second and then every live task pair at its anchor directory, and drops a pair again once the pair is removed.
The `_portals` container, whose only consumer is today's workspace, is retired.

## Scope

In:

- The root umbrella's workspace file, `<hub>/<RepoName>.code-workspace`, replacing `<hub>/_launchers/<AnchorRel>/<prime>.code-workspace`.
- Its folder list, display names and order.
- Regenerating the file from fabric whenever a hub is cloned or a warp task worktree is created or removed, from `lyx fabric reconcile`, and from `lyx ide spawn <prime>` as today.
- Retiring `_portals`: no verb creates, restores or removes a portal link any more, and `lyx fabric reconcile` sweeps the retired portal links and the retired workspace file from an existing hub.
- The docs and help text that describe the workspace or `_portals`.

Out:

- Workspace files for umbrellas (now called beams, see `orch-any-worktree` and `umbrella-daemon`) other than the prime.
  No other umbrella exists yet; the task that builds them adds their files, under the naming the operator already decided (`<RepoName>-<umbrella-slug>.code-workspace`), reusing D3's function, and partitions pairs among them.
- Batch worktrees planned by `parallel-batch-parts` and `weaver`: none exist, and whether the workspace lists them is those tasks' call.
- A task pair's own `lyx ide spawn <slug>` and batten's VS Code seed and open rows: they keep opening the pair's bare folder.
- `_launchers` and its launcher scripts.

## Decisions

### D1 — File location and name

The workspace file is `<hub>/<RepoName>.code-workspace`, with `RepoName` from the resolved `lyxcwd.Location`, so the loomyard hub's file is `loomyard.code-workspace`.
`fabricengine` alone declares its path, as it declares `HubWorkspacePath` today.

Rationale: the operator decided the files live directly in the hub directory, ready to open, and carry the repository name because VS Code's recent list shows workspaces by file name.
The hub directory is no git repository, so the file is untracked by construction, and every scan of the hub's children already skips regular files.

### D2 — Folders, names and order

The folders are, in order:

1. The prime's anchor directory, named after the prime's checked-out branch (`main`); a prime on a detached HEAD is named after its directory.
2. `_board`, named `Bolt`.
3. Every live task pair's anchor directory, named after its slug, sorted by name in byte order.

A live task pair is a code worktree of the hub other than the prime whose anchor directory exists; weft worktrees are never listed.
A worktree entry the listing cannot resolve is skipped, never fatal, so one half-created pair never drops the others.
Every task pair of the hub is listed, including a pair forked from another pair and a pair created with `fabric add --local`.
Folder paths stay relative to the file's directory and slash-separated, and the `settings` key still splices the prime's `.vscode/settings.json` verbatim.
A folder whose directory does not exist is left out, as today.
`fabricengine`'s doc states that the file lists every task pair of the hub and that no per-umbrella partition exists.

Rationale: the operator's decisions name the prime first, Bolt next and the rest alphabetically by display name, each pointing straight at its anchor.
The prime is the only umbrella today, so every pair of the hub belongs to it.
`fabric-remote-async` turned local pairs into an explicit operator choice for local work, so the operator wants to see one.
Two folders sharing a display name (a task slugged `main`) are left to VS Code, which shows both.

### D3 — One regeneration function, owned by fabric

`fabricengine` gains one function that resolves the prime, lists the folders, reads the prime's settings, builds the file through `vscode.BuildWorkspace` and writes it if its bytes changed, appending `KindFileWritten` to the `*Mutations` it is given when it wrote.
It returns every failure: an unresolvable prime, unreadable prime settings, a lock timeout or a failed write; on any of them it writes nothing.
`fabricengine` imports `internal/vscode` for this, retiring the "imports no vscode package" comment; `internal/vscode` imports no package that imports `fabricengine`, so no cycle forms.
`ideengine`'s prime spawn calls the same function with a recorder of its own that it discards, and keeps returning its errors as today.
The relative-settings warning moves with it and names the hub directory as what relative settings now resolve against.

Rationale: fabric verbs are called in-process from batten, the CLI and test fixtures, so a regeneration inside the verbs covers every path, and one function keeps the prime spawn and the verbs from writing different files.
Writing nothing on a failed prime resolution keeps the first folder stable, which VS Code needs to avoid restarting extensions.

### D4 — When fabric regenerates

These fabric verbs regenerate the file: `Clone`, `Add`, `AddLocal`, `Remove`, `Prune` with apply, and `Reconcile`.
`Cleanup` removes no worktree and does not regenerate.
A creating or removing verb regenerates on every exit past its pre-flight checks, whatever error it returns, so a refused-midway or partial `Remove`, or an `Add` rollback, still leaves the file matching the worktrees on disk; a pre-flight refusal regenerates nothing.
`Reconcile` regenerates as the repair verb that a failed regeneration's warning names.
In a verb, a regeneration failure never fails or rolls back the verb: it is logged at Warn naming `lyx fabric reconcile` and carried as a detail in the verb's result where the result type has one.

Bound: the only thing a failure lets through is a stale folder list, which the next regenerating verb or prime spawn rewrites.

Rationale: the operator decided a finished pair drops out of the workspace, and batten's teardown already goes through `Remove`.
A freshly cloned hub holds its file at once, ready to open.
VS Code picks up a changed folder list in an open window and restarts extensions only when the first folder changes, which never happens since the prime stays first.

### D5 — Concurrent writers

The listing and the write run under one hub-level file lock in the board's never-tracked state dir.
It is a leaf lock: taken after any lock the verb already holds, with no lock taken under it.
Its wait is bounded to a few seconds; a timeout is a regeneration failure under D3 and D4.
The file is written in place through the hub `os.Root`, with no temp file.

Rationale: batten runs pairs in parallel, and two unlocked regenerations can interleave so that the later write lists fewer pairs than exist.
A bounded wait keeps a stuck holder from hanging `Add` or batten's teardown.
VS Code rereads the file on the write's last change event, so a reader that meets a partial write recovers without an atomic replace, and no temp file has to be placed or cleaned up.

### D6 — Retire `_portals`

Fabric no longer creates, restores or removes portal links: the portal helpers in `portals.go`, Reconcile's portal restore and its `portal_restored` action, Remove's portal step and its portal presence probe, Prune's portal teardown and the `hubforge` portal accessor go, with their tests.
The workspace write no longer creates `_portals/<AnchorRel>`, and `ideengine` no longer writes twice.
`_portals` stays in the hub-reserved name set, and its name constant stays in `internal/fabricengine`, the token's registered owner, for that set and the D7 sweep.

Rationale: the operator decided `_portals` is not needed once every pair is a folder of its own, and the workspace is its only consumer.
Existing hubs still hold the directory until a sweep.

### D7 — Sweep the retired artifacts

`lyx fabric reconcile`, from whichever worktree it runs, removes through `destroy.go`:

- each link under `<hub>/_portals/<AnchorRel>/` whose target path is `<hub>/<slug>/<AnchorRel>/_lyx` for the link's own name, matched by the target's path whether or not the target still exists, then the emptied `_portals` chain;
- the file at `<hub>/_launchers/<AnchorRel>/<prime>.code-workspace`, removed as a link if one is planted there and never followed; this removal is skipped when the prime cannot be resolved.

Anything else under `_portals` is kept, with a detail naming it, and keeps its directory.
Each removal is recorded in Reconcile's `*Mutations`; an absent artifact is a no-op.
While either retired artifact exists, every regeneration logs one Warn naming `lyx fabric reconcile`.

Bound: the sweep deletes only fabric-shaped portal links, judged by target path, and the one entry at the retired workspace path.

Rationale: the operator's hubs already hold portals and the old file; a portal of a pair removed after deploy dangles, since `Remove` no longer tears it down, and the old file stays in VS Code's recent list.
Reconcile is the verb that already repairs hub-level entries, and the Warn tells the operator to run it.

### D8 — Docs in the same commit

Every doc comment, help text and doc that names the hub workspace file or `_portals` is updated to the new file and to `_portals` as retired.

Rationale: CLAUDE.md's docs-in-the-same-commit rule.

## Constraints

- `PATTERN-fabric-write-containment`: the workspace write goes through an `os.Root` at the hub.
- `PATTERN-fabric-destruction-chokepoint`: the D7 sweep removes only through `destroy.go`.
- `PATTERN-mutation-record`: every write and removal D3, D4 and D7 make is appended after it happened.
- `PATTERN-hub-containment`: the file references hub-inward folders only and is never written into a worktree.
- `PATTERN-fabric-git`: reading the prime's branch goes through fabric's own git helpers.

## Auto-mode assumptions

- D2: local pairs and pairs forked from pairs are listed; a detached prime is named after its directory.
- D3: the regeneration lives in `fabricengine` and imports `internal/vscode`.
- D5: in-place write, no atomic replace.
- D7: the sweep runs in `lyx fabric reconcile` only, prompted by the regeneration's Warn.

## Open risks

- An open window that has a pair folder whose directory vanished before the rewrite shows a missing folder until the next regeneration.
- A hub where the operator ignores the Warn keeps its `_portals` directory and old workspace file.

## Acceptance criteria

- On a hub with a prime on `main` and pairs `b-task` and `a-task`, the file `<hub>/<RepoName>.code-workspace` lists `main`, `Bolt`, `a-task`, `b-task` in that order, each path the folder's anchor directory relative to the hub.
- A freshly cloned hub holds the file listing `main` and `Bolt`; a hub with no `_board` omits `Bolt`.
- A prime on a detached HEAD is listed under its directory name.
- `fabric add` adds the new pair to the file; `fabric remove` and `fabric prune --apply` drop it; an `Add` rollback and a `Remove` that fails after taking the task worktree leave the file without it.
- A `fabric add --local` pair is listed.
- A worktree entry the listing cannot resolve is skipped and the other pairs are still listed.
- A regeneration that fails (unresolvable prime, unreadable settings, lock timeout, failed write) writes nothing, leaves the verb's own outcome unchanged and logs a Warn naming `lyx fabric reconcile`; the prime spawn returns the same failure as an error.
- An unchanged folder list rewrites nothing and records no mutation; a changed one records `KindFileWritten`.
- Two concurrent regenerations on the same hub end with a file listing every pair that exists after both; a regeneration that meets a held lock past its bound gives up without hanging the verb.
- `lyx ide spawn <prime>` launches `<hub>/<RepoName>.code-workspace`, and no verb creates anything under `_portals`.
- `lyx fabric reconcile` on a hub holding fabric-shaped portal links, including one whose pair is gone, and the old workspace file removes them all, keeps a foreign entry under `_portals` with a detail, and records each removal; a second run changes nothing.
- While a retired artifact exists, a regeneration logs the Warn; after the sweep it does not.
- A symlink planted at the workspace path or a component of the sweep path that escapes the hub is refused; a symlink planted at the retired workspace file is removed as a link, its target untouched.

## PATTERN changes

- `PATTERN-fabric-write-containment` — changes: Writing under `_launchers`, or the hub workspace file, from `fabricengine`: via an `os.Root` at the hub, never raw `os.MkdirAll`, `os.WriteFile` or `fslink`.
- `PATTERN-hub-containment` — changes: Linking hub-level containers or writing the hub workspace file: from the hub only, never into a worktree.

## Notes for the plan writer

- `PATTERN-hub-containment`'s background becomes: `_board` and `_launchers` are reachable from the hub only; `_launchers` links point hub-inward only; the hub workspace file sits directly in the hub dir, references hub-inward folders only and is never written into a worktree; `_portals` is retired, still reserved, and swept by `lyx fabric reconcile`.
- The comment in `add.go` saying no hub workspace lists a local pair goes with D2.
