# Support log — lenses-module

## Interview

### Turn 1 — exploration

Autonomous run, no operator turns.
Read the board entry and the related entries `writer-reviewer-lens-align`, `review-setup-audit`, `twill-writer-quality-audit` and `plan-writer-finding-classes`; none plans a change to where lenses live, and `writer-reviewer-lens-align` records the seven-lens trial this task makes the default.
Read burler's config, fan key grammar and template, configreg's registration and key moves, configsync's reconcile and key-move code, yamlengine's `Reconcile`, hubreconcile's package doc, the loom twill loader and the hub's `burler.yaml` and `loom.yaml`.
Findings that shaped the design: burler is seed-only, so reconcile never touches a present file; the declared key move already migrates a key between modules at every layer; `LoadOrTemplate` fills missing keys from the template, which replaces burler's per-name fallback; `hubreconcile.Ensure` walks the hub after a binary change, so the migration needs no new route.

### Turn 2 — decisions

Settled D1 to D6 as auto-picks; see the question ledger.
The first draft of D2 made `lenses` a plain reconciled module, pruning a lens or fan the template lacks.

### Turn 3 — lens review

Seven forks ran; every one returned a report.

- interactions F1 (BLOCKING), run settings naming `burler.lenses` cannot resolve once burler is deregistered and guard keys were never valid run settings: fixed, the run-settings claim and criterion are gone and D1 states the bound.
- interactions F2 (MEDIUM), pruning leaves a tuned fan naming a pruned lens: fixed by D2's keep-and-report.
- interactions F3, walkthrough 1, bound-breaker 1, existing-paths 1 (MEDIUM, merged), the dry run cannot report the apply pass's additions: fixed, D4 states the bound and the criterion names only the moves.
- board-fit (MEDIUM), pruning removes the config-only trial route other entries plan on: fixed, D2 now keeps and reports hub-only entries and `--set` adds them.
- goal-scope and leanness (MEDIUM, merged), the PATTERN bullets carried background text: fixed, the bullets keep the one-liners and the note names the background edits.
- goal-scope (MEDIUM), D6 claimed no code while the deletion commit was unverified: fixed, D6 says the plan makes the commit stage the deletion if it does not.
- existing-paths 2 (MEDIUM), `moveRunSettings` leaves an empty `burler` object: fixed with interactions F1.
- bound-breaker 2 (MEDIUM), an old binary re-seeds `burler.yaml` and the two walks alternate: fixed, named in Open risks with its bound.
- walkthrough 2 and bound-breaker 3 (LOW, merged), `LoadOrTemplate` expands environment references in prose: fixed, D3 states the bound.
- walkthrough 3 and bound-breaker 3 (LOW), the load logs filled keys: rejected, a reconciled hub file lacks no key and a standalone run logs once per load.
- bound-breaker 4 and existing-paths 3 (LOW, merged), the template header says seed-only: fixed, Scope rewrites it.
- bound-breaker 5 and existing-paths 4 (LOW), the deletion path does reach the commit and legacy removals are precedent: fixed, Open risks narrowed.
- goal-scope (LOW), the CLI change had no criterion: fixed, a criterion and a Scope line added.
- goal-scope (LOW), pruning is a reading the entry does not state outright: superseded by D2's keep-and-report.
- goal-scope (LOW), the leaf listing is unrequested: rejected, it is the cycle guard for the three importers and costs one background row.
- board-fit (LOW), `writer-reviewer-lens-align` left half-done: fixed, Scope's Out names what stays open there.
- board-fit (LOW), no dependency gap: no change.
- leanness (LOW), constraints restating decisions, the tolerant-records bullet, the pinned cap, the auto-mode list, D1's symbol list, the second open risk and the first note: fixed, each cut or reduced.
- leanness (LOW), criteria pinning existing refusals: fixed, reworded to the moved tests.

## Rejected alternatives

- Keep `lenses` seed-only like burler: reconcile would still never carry a new lens into a hub file, which is the defect the task fixes.
- Declare `lenses` and `fans` open maps: `Reconcile` carries an open map whole and reports nothing under it, so new template lenses would never be added.
- Prune a lens or fan the template lacks, as other modules prune keys: removes the config-only trial route and breaks a fan key naming a pruned lens at the next config load.
- A per-name merge for the burler-to-lenses migration: the whole-key move plus the same pass's reconcile gives the same result for every file the hub has, with no new merge code.
- A dedicated migration command or a `Migrate` hook on `lenses`: a `Migrate` hook sees only its own module's file, and a new command duplicates the key move and Ensure.
- Keep `burler` registered as an empty module: it would hold no key, and `lyx config burler` would show nothing.
- Strict `configengine.Load`: refuses a hub without a `lenses.yaml` and a standalone `lyx burler run` with no file, both of which work today.
- Make `lenses` hub-wide: burler is per-worktree today and nothing asks for the change.
- Package name `internal/lensconfig`: avoids shadowing local `lenses` variables but departs from the module name the operator chose.
- Delete the `premises` lens once no fan names it: deleting lens prose is outside this task.
- A dry run that reconciles the moved target in memory: new code for a report the apply pass already gives.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): where do lenses and fans live in code? Picked a new leaf package `internal/lenses` with the template, load, resolution and fan key grammar moved from `burlerengine`, because burler, loom and darn all import it and a leaf avoids cycles.
- Q2 (auto-pick, revised at lens review): how does reconcile treat the module? First picked plain reconcile with pruning; revised to per-name maps that add template entries and keep and report hub-only ones, because pruning removed the config-only trial route and could break fan keys.
- Q3 (auto-pick): which load stance? Picked `LoadOrTemplate`, whose template fill replaces the per-name fallback and keeps an absent file working.
- Q4 (auto-pick): how does a hub's `burler.yaml` migrate? Picked two declared key moves, so every layer migrates through the existing route, and the emptied `burler.yaml` is deleted.
- Q5 (auto-pick): does `burler` stay registered? Picked deregistering it, since its file holds nothing else.
- Q6 (auto-pick): what happens to `premises`, which no fan names after the `twill-discussion` change? Picked keeping it defined.
- Q7 (auto-pick): does the post-landing hub migration need its own route? Picked none: `hubreconcile.Ensure` runs the walk at the first start verb after deploy, and the orch's reconcile confirms it.
- Q8 (auto-pick): hub-wide or per-worktree? Picked per-worktree, as burler is today.
- Q9 (auto-pick): must a dry run report the apply pass's additions? Picked no; it reports the moves, and the bound is in D4.
