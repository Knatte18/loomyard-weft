# Support log — batten-records-leak-into-pairs

## Interview

Autonomous session: no operator answered; each turn below is a question the agent put to itself, answered from the codebase.

1. **What leaks, and through what?**
   Batten's prime-side run directory is `_lyx/shed/<slug>/` (via `shedrun.RunDir`), committed onto prime's weft pair by batten's `CommitStatus` seam.
   `fabricengine.Add` forks every new weft branch from the parent's weft branch, so every pair inherits every committed `_lyx/shed/*`.
   The child's own loom run resolves `self` to the same slug, finds batten's seed, and `WriteSeed` refuses as a disagreeing seed.
   This worktree's own `_lyx/shed/` shows the inherited copies.
2. **Do inherited records ever flow back?**
   No: `fabricengine`'s merge verbs merge the warp checkout alone and never make the weft a merge participant.
   So removing records on a child's weft branch cannot delete anything on the parent.
3. **Namespace for batten vs. stop the inheritance?**
   The brief suggests both.
   A batten namespace fixes only the collision; the clutter remains, and it splits the flat run-id namespace `lyx shed seed|step|status` and ly-drive address, and needs a migration of in-flight batten runs.
   Stopping the inheritance fixes both, for every creator of pairs.
   Pick: stop inheritance only.
4. **Where to stop it: batten's `Worktree-Create` or `fabricengine.Add`?**
   `Add` has two callers (`lyx fabric add`, batten); dropping in `Add` covers hand-made pairs too.
   Pick: `Add`, on the freshly forked branch only, not on the adopt path.
5. **How does `fabricengine` name `_lyx/shed` without breaking the Shed Run-Directory Invariant?**
   `shedrun` exports an anchor-relative constructor; `fabricengine` already imports domain packages (`pattern`, `stencilstore`), and `shedrun` imports only `lyxcwd`/`lyxdirs`, so no cycle.
   Pick: `Add` calls the constructor itself rather than taking a caller-told option.
6. **Which destruction rules apply?**
   Deleting files in the new weft checkout is a destructive primitive under the Fabric Destruction Chokepoint Invariant; an index-only removal avoids it.
   Left to the plan, stated as a constraint.
7. **What is `batten-prime-lock-wait`?**
   Not on the board.
   `battenshed`'s create and teardown producers call a non-blocking `TryAcquireWriteLock` and return `Stuck` on contention, so a second `lyx batten run` started while the first creates its pair halts `blocked`.
   Pick: a bounded, context-cancellable retry loop in the producers, 2 s poll, 10 min bound, sleep seam for tests.
8. **Legacy pairs?**
   Existing pairs keep their inert copies; the hub already unblocked the affected ones by hand, and `Seed-Child` runs right after create, so no pair created before the fix is still waiting to seed.
   No special handling.

## Rejected alternatives

- **Batten-owned namespace in prime** (e.g. `_lyx/batten/<slug>/`, or a `batten-<slug>` run-id).
  Fixes the collision but not the clutter; needs a read-fallback or migration for in-flight runs; makes `shedcli`'s seed lookup (which reads the seed to learn the recipe) probe two roots or learn a recipe-to-root mapping.
- **Drop in batten's `Worktree-Create` after `Add`.**
  Leaves pairs from `lyx fabric add` leaking, and adds a second weft commit and push per create.
- **Caller-told `AddOptions` field listing paths to drop.**
  A future caller of `Add` can forget it; `Add` deriving the path from `shedrun` cannot.
- **Fabric config key listing non-inherited directories.**
  Configurability nobody asked for.
- **Drop all per-run directories (`_lyx/loom/`, `_lyx/discussion/`, `_lyx/plan/`, …) too.**
  No reported problem; YAGNI.
- **Stop committing batten records to prime's weft.**
  They are committed so a batten run resumes on another machine; losing that is a regression.
- **Seed-Child treats an inherited batten seed as foreign and overwrites it.**
  Treats the symptom, needs a recorded-driver/recipe read the Driver Choice Single-Site Invariant scrutinises, and leaves the clutter.
- **Blocking `lock.AcquireWriteLock` for the prime lock.**
  Not cancellable and unbounded; a stuck holder would hang the batten step forever.
- **Prime-lock wait as a config key.**
  No tuning need exists; constants suffice.

## Review rounds

_No rounds yet._

## Question ledger

| # | Question | Status | Answer |
|---|----------|--------|--------|
| 1 | Add a batten-specific namespace as the brief suggests? | Resolved (auto-pick) | No; stop inheritance in `fabricengine.Add` instead, which fixes collision and clutter both. |
| 2 | Drop in `fabricengine.Add` or in batten's `Worktree-Create`? | Resolved (auto-pick) | `fabricengine.Add`, fresh-fork path only. |
| 3 | How does `Add` know the path? | Resolved (auto-pick) | New `shedrun` anchor-relative constructor, called by `Add` itself. |
| 4 | Drop only `_lyx/shed/` or all per-run directories? | Resolved (auto-pick) | Only `_lyx/shed/`. |
| 5 | One commit or two on the new weft branch? | Resolved (auto-pick) | One, together with the origin record; drop included only when the fork point tracks something there. |
| 6 | Mechanism for the drop (working-tree delete vs index-only)? | Open for Plan | Either, provided it satisfies the Fabric Destruction Chokepoint Invariant; plan states which. |
| 7 | Scope of `batten-prime-lock-wait` with no board entry? | Resolved (auto-pick) | Contention on the prime lock halts a second create `blocked`; fix with a bounded wait in both lock-holding producers. |
| 8 | Wait bound and poll interval? | Resolved (auto-pick) | 10 min bound, 2 s poll, package constants. |
| 9 | Handle pairs created before the fix? | Resolved (auto-pick) | No; inherited copies are inert and blocked runs were already cleared by hand. |
| 10 | Environment faults observed? | Resolved | None: `lyx board get` worked; `lyx board get` for `batten-prime-lock-wait` returned `task: null`, which is a missing board entry, not a broken tool. |
