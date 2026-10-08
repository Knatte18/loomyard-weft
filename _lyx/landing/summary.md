# A hub survives a lyx binary change; spawned agents get the edit rule

A `lyx` binary change no longer needs a manual `lyx config reconcile --apply` in the prime and every pair.
A pair behind main no longer asks for a production deploy.
Every spawned agent other than the orch is now told the no-script edit rule.

## Config reconciles itself after a binary change

The hub keeps one never-tracked build stamp beside a hub lock, under the board dir's `.lyx` mirror of the hub-wide config dir.
The stamp key is the binary's VCS revision, its modified flag and a fingerprint of the config registry (module names, flags and embedded templates).
The fingerprint catches a dirty dev rebuild whose revision did not move but whose templates did.
An absent stamp counts as stale.

The new `internal/hubreconcile` package owns the stamp, the lock and the walk, through one entry point, `hubreconcile.Ensure`.
On a stale stamp it takes the hub lock with a bounded wait, re-reads the stamp, and reconciles only if it is still stale, so two start verbs running together produce one reconcile.
The walk reconciles the hub-wide modules, then the prime, then every pair, and re-enumerates pairs until none is new before it writes the stamp.
The new `PATTERN-binary-change-reconcile` entry pins `Ensure` as the only route.

The entry point runs before their own work in `lyx orch start`, `lyx batten run`, `lyx loom start` (and its root `lyx start` alias) and `lyx loom resume`, and only inside a hub.
No other command reconciles, and `lyx config reconcile` keeps its per-worktree behavior and does not write the stamp.
The loom recipe's `Preflight` row calls the same entry point as a safety net, before its worktree-clean check.

- **Committed result:** hub-wide files are committed in `_board` and pushed, where a push failure is logged and never fatal.
  Each worktree's written config files are committed through `Fabric.Commit` with a file list holding only those files, so the commit serializes with a running driver's own records commits.
  A reconcile therefore no longer leaves uncommitted `_lyx/config` changes to block `Preflight` or a Finalize merge.
- **New pairs:** `lyx batten run` reconciles before it creates or adopts its pair, so the pair forks already-reconciled config.
  It then reconciles that pair once more, whatever the stamp says.
- **Mid-merge worktrees:** a worktree mid-merge is skipped without being written, the stamp stays stale and the next start verb retries it.
  If a merge starts between the write and the commit, the written files are restored to their prior bytes.
- **Failures:** a failing reconcile or commit stops the start verb with a JSON error naming the worktree and file and the way forward.
  The `Preflight` row goes `Stuck` instead.
  A hub-lock timeout is handled the same way, with a way forward to re-run.
  The four new refusals have rows in `contracts/specs/refusal-spec.md`, each with a test that reaches it.
- **Logging:** each changed module per worktree and a summary line naming the previous and new revision go through `internal/logger`, and nothing goes to stdout.
- **Build identity:** the VCS reader in `internal/shedverbs` moved to a shared home that `shedverbs` and `hubreconcile` both import; step records carry the same `vcs_revision` and `vcs_modified` fields.

Bounds a reviewer should know:

- A dirty dev rebuild that changes only a `Migrate` hook, with no template change, does not trigger; `lyx config reconcile --apply` remains the manual route.
- Running an older binary after a newer one reconciles back to the older templates, and the newer binary later re-adds a removed key with its template default.
- A `lyx fabric add` whose fork read the prime before a walk committed it, and whose pair becomes enumerable only after that walk's last enumeration, escapes by design; Loom-Preflight's retired-key refusal is the backstop.
- While a skipped mid-merge worktree or a failing file keeps the stamp stale, every start verb and every run's `Preflight` row repeats the walk or stops until the merge ends or the file is fixed.

## Stencil drift behind main

Closes #445

A drifted stencil whose board copy is untouched and whose worktree source differs from the embedded bytes is `source-ahead` only when the binary's VCS revision is an ancestor of the worktree's HEAD.
Otherwise the class is `behind`, logged at info level with no deploy advice.
A binary with no VCS revision keeps today's classification.
The ancestry check runs in `internal/fabricengine`, only when a drift comparison needs it, and reaches `internal/stencilstore` as an input, so a worktree with no drift runs no git and `stencilstore` runs none.
The Part 1 stamp is deliberately not used here, since it says the binary changed, not whether a given worktree has caught up.

## The no-script edit rule reaches spawned agents

One shared `edit-directive` stencil, rendered by the new `internal/editdirective`, states the rule: change files only with Edit or Write, never with a script that rewrites them; never use `sed` at all; search and read with `grep`, `awk` or `cat`.
It renders unconditionally, unlike the PATTERN directive, into every spawned role's opening stencil other than the orch's.
The rule never widens a role's write limits.
`PATTERN-edit-directive` pins this, and a stencil scan in `contracts/stencils` fails when a spawned role's opening stencil lacks the marker or another stencil restates the rule.

## To check by hand

- **Config read failure during the walk:** the walk's config snapshot treats an unreadable config file as absent, which would delete it on restore.
  Check in `internal/hubreconcile/walk.go` that a file that exists but cannot be read fails the worktree.
  Fixes for this and two related restore cases landed in the last commits, and no fresh review has seen them.
- **Pair that is incomplete when the walk reaches it:** check in `internal/hubreconcile/walk.go` that the walk does not write config into a pair that is not yet complete.
- **Not run:** the first start verb after a real production deploy of this binary against the live hub, including a `lyx batten run` that creates a pair.
- **Not covered by this change:** the `merriamFixedContext` calibrate step in `tools/tokencount` and extending the shuttle deny beyond python.
