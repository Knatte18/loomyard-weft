# Support log — scratch-defeats-test-cache

## Interview

Autonomous run: no operator answers; every pick below is an auto-pick recorded in the question ledger.

### Turn 1 — the mechanism

Read the board entry (issue #771) and the Go toolchain's `hashOpen` in `cmd/go/internal/test/test.go` (go1.26.9).
Opening a directory hashes the `Lstat` of every entry, modification time included.
`scankit.collect` walks from the module root with `filepath.WalkDir`, which opens the root, and skips `.scratch` only as a descent target.
So the root's `.scratch` entry's modification time, which moves on every create or remove directly under it, is in the cache key of every scan that walks the root.
The entry's wording "changes the worktree root listing" is right in effect: the listing's hash changes, not its names.

### Turn 2 — what else churns in the root

In a task worktree, `.git` is a file and `.lyx` and `_lyx` are links, so their hashed `Lstat` is stable.
The prime holds a real `.scratch` with live content cited by board entries, so any fix must keep `.scratch/<name>` paths valid.

### Turn 3 — where the fix goes

Options weighed: change the scratch path agents use; stop the scan kit from opening the root; make `.scratch` a link.
Picked the link (D1): agents keep their path, the root listing stays a hashed input, and only the churning entry becomes stable.
Checked the gates that read junction health (`PairComplete`, `preflight`'s `Healthy`) and kept scratch out of them (D3).
Checked git's handling of a link against a trailing-slash exclude pattern, which led to D4.
Checked `lyx cleanup`'s scratch segment: `os.ReadDir` on the link follows it, so only its doc changes (D5).

Lens review, all seven forks returned; findings judged together after the last one:

- interactions F1, walkthrough 1, bound-breaker 4 (BLOCKING, merged): materialising the target first made D2's migration unreachable. Fixed: D1 now classifies `.scratch` before touching the target, with an ordered case list.
- interactions F2, existing-paths 1, bound-breaker 6 (merged): re-pointing a wrong link removes a link, which goes through `destroy.go`, contradicting "nothing is deleted". Fixed: the step never re-points; a wrong link is a leave case (D1, D2).
- interactions F3, bound-breaker 1 and 2, walkthrough 3 (merged): reconcile running the step on every visited worktree would plant a weft tree for unmanaged or vanished pairs and break the unmanaged no-touch contract. Fixed: D3 runs it only from `repairPairWiring` on pairs whose weft root exists, and never creates anything without one.
- interactions F4: the leave detail had no home for add, checkout and clone. Fixed: D2 logs it through `internal/logger` there.
- existing-paths 2 (BLOCKING): reconcile on a healthy pair would add the link but keep the `/.scratch/` exclude, making the link untracked dirt. Fixed: the scratch step owns both link and exclude (D1, D4), and an acceptance criterion covers the healthy legacy pair.
- existing-paths 3: unwire's effect on the link was unstated. Fixed: D3 states unwire leaves link and exclude, and the weft stays.
- existing-paths 4: dangling link case missing. Fixed: D1 case 2 re-materialises the target.
- existing-paths 5: cleanup traced through the link. No change needed; it confirms D5.
- walkthrough 2, bound-breaker 3 (merged): `rm -rf .scratch` plus `mkdir` never converges, contradicting "until the next reconcile". Fixed: D2's bound states the state persists until the operator clears it, and the report names the way forward; automatic merging of children was not added.
- bound-breaker 5: rename-then-link race unnamed. Fixed: named as a leave case in D1.
- bound-breaker 7: "only `.scratch` churns" overstated, `.claude/` also churns. Fixed: the assumption is trimmed and Scope Out names `.claude/`.
- goal-scope 1: migration silently relied on the operator. Fixed: D3 and the auto-mode assumption state the one-time `lyx fabric reconcile` after landing, and the fabricengine doc says so.
- goal-scope 2: goal claimed more than D1 delivers. Fixed: the goal is limited to a wired link.
- goal-scope 3: plain-clone exclusion unexplained. Rejected: no gate the task cares about runs in an unwired checkout; the push gate runs in the wired prime.
- goal-scope 4: narrower fix by `git ls-files` top-level dirs not weighed. Rejected: listing through a spawned git hides the root from the cache like the bypass-`os` option, so a new top-level directory could serve a stale pass; added to rejected alternatives.
- board-fit 1: Windows risk unowned. Fixed: the open risk names `fabric-windows`.
- board-fit 2: overlap with `finalize-dirty-names-paths`. Rejected: D4's rationale already covers dirty-tree checks; no dependency either way.
- board-fit 3: text overlap with `origin-reconcile` in `reconcile.go`. Rejected: a merge concern for the plan, not a design dependency.
- board-fit 4: `run-process-ownership` fits. No change.
- leanness 1: Goal carried the investigation. Fixed: cut to the goal and one sentence of mechanism.
- leanness 2: rejected alternative argued in auto-mode assumptions. Fixed: bullet removed.
- leanness 3: auto-mode bullet 1 repeated Scope. Fixed: shortened to the bare assumption.
- leanness 4: D2 restated the chokepoint constraint. Fixed with the D2 rewrite.
- leanness 5: generic test-hygiene constraint. Fixed: removed.
- leanness 6: notes repeated D4. Fixed: clause cut.
- leanness 7: test-assertion risk was implementation guidance. Fixed: moved to notes for the plan writer.

## Rejected alternatives

- Document that scratch files must be created before the first gate run (the board entry's second option): every agent pays the cost, and any file created later still misses.
- Move agent scratch to a path outside the root, such as `.lyx/scratch/`: rewrites the edit directive, stencils, skills and cleanup, and breaks every `.scratch/<name>` citation on the board.
- Make `scankit` avoid opening the module root, by a fixed list of top-level directories or a listing that bypasses `os`: a fixed list misses a new top-level directory silently, and bypassing `os` hides a real input from the cache, which can then serve a stale pass.
- Nest scratch one level deeper inside a real `.scratch`: every agent would need a per-session subdirectory, and creating one still moves the hashed entry.
- Count `.scratch` in junction health so reconcile repairs it through the existing path: junction health gates runs through `PairComplete` and `preflight`, so a stray real `.scratch` would refuse a run.
- Remove an existing target directory to make room for migration: that is destruction outside `internal/fabricengine/destroy.go` and can lose scratch content.
- Make `scankit` walk only the top-level directories `git ls-files` reports: a listing read by a spawned process is no cache input, so a new top-level directory could be served a stale pass.
- Re-point a `.scratch` link to another target: removing the old link is destruction, which would need a scratch ownership rule in `destroy.go` for a case nothing produces.
- Merge a real `.scratch`'s children into an existing target when names do not collide: needs an empty-directory removal afterwards, and the case only arises after an agent removed the link.
- An automatic migration trigger on deploy or landing: a one-time operator `lyx fabric reconcile` covers it.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Fix in fabric wiring, the scan kit, or the agents' scratch path? Picked fabric wiring with a `.scratch` link (D1); it is the only option that leaves the agents' path and the cache's correctness alone.
- Q2 (auto-pick): Link target? Picked `scratch` under the weft-side `.lyx`: never tracked, already excluded in the weft, torn down with the pair.
- Q3 (auto-pick): Migrate existing real `.scratch` directories? Picked rename when the target is absent, else leave and report with the way forward (D1, D2); the prime's cited scratch content must survive, and nothing may be removed.
- Q8 (auto-pick, from the lens review): Which worktrees does reconcile's scratch step reach? Picked managed pairs with an existing weft root, from `repairPairWiring` (D3), so the unmanaged no-touch contract holds and no weft debris is created.
- Q4 (auto-pick): Should a scratch problem fail wiring or refuse a run? Picked no (D2, D3); scratch is convenience and must never block a task.
- Q5 (auto-pick): Automatic migration trigger for existing pairs? Picked none beyond `lyx fabric reconcile` and re-wiring; adding one is a requirement nobody asked for.
- Q6 (auto-pick): Exclude pattern? Picked `/.scratch` replacing `/.scratch/` (D4), since git does not match a link with a trailing-slash pattern.
- Q7 (auto-pick): PATTERN changes? None; the link is fabric's own wiring detail and the existing entries cover its constraints.
