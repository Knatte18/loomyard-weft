# Support log — tuned-defaults-template

## Interview

### Turn 1 — exploration (autonomous, no operator)

Read the board entry, the related entries `lenses-language-sets`, `kickstart-pack-remove` and `test-tiers`, and the config stack: `configengine`'s layers and fill, `configsync`'s hub-wide reconcile and worktree strip, `hubreconcile`'s walk and stamp, and `configreg`'s module flags.
Diffed loomyard's `_board/_lyx/config/` against every embedded template.
Findings: the board entry's loom, shed, darn, orch, webster and batcher differences match the hub files;
`claude`'s difference is `prompt_cache_ttl_roles`;
`models` differs only in indentation;
`board` differs in kinds as well as labels, and `language.skills` is Go's, both loomyard's own.
`ReconcileHubWideAt` adds every template key to a base file, which is why a seeded hub pins its seed-time values.

### Turn 2 — design picks (auto)

Settled D1 to D6 as auto-picks, recorded in the question ledger.
The board entry's "after the change, loomyard's default layer holds only the kept entries" decided a sparse base over a value migration that keeps it full.

### Turn 3 — lens review

Every lens returned a report.

- bound-breaker, interactions, existing-paths: the marker is written by any `Apply` caller but committed by none, since callers commit only module results, and a failed walk cannot restore a file that was new — fixed: the strip and the marker are walk-only, committed in the walk's board commit, removed on failure; clone writes the marker only on a board with no base file.
- interactions, walkthrough, existing-paths, bound-breaker: an unmarked worktree keeps old neutral values as overrides once the base is stripped — fixed: the walk's pending worktree strip also drops former values (D3).
- board-fit, existing-paths: `moveKeys` starts an absent base from the template and overwrites a template-default target — fixed: key moves added to D4, with a plan note.
- bound-breaker: a `--set` of a former value between clone and the first walk is lost — fixed for new hubs by clone's marker; for existing hubs the window between deploy and the first walk is stated as an auto-mode assumption.
- interactions, walkthrough: D5's "base is sparse" had no test — fixed: Board told and module not seed-only.
- interactions: `lenses` drops template-equal entries on every reconcile, against D2's pin rule — fixed: named as the exception in D2.
- interactions: the resolved-stack comparison for `Retired`, `Divergent` and `Inherited` lived only in criteria and notes — fixed: moved into D2.
- interactions: the editor's empty-save abort — fixed: one clause in D4.
- bound-breaker: a comment-only base does not load as empty today — fixed: kept as D2's requirement, with a plan note on `readLayers`.
- bound-breaker: running pairs see new defaults mid-run — fixed: open risk added.
- board-fit: name `machine-handover` for the older-binary risk — fixed; treating a full-key-set base as unmigrated — rejected: a heuristic over file shape, and the way forward is deploying on every machine.
- board-fit: any task retiring or moving a D1 key stales the table, and `test-tiers` touches `configsync` — fixed: risk generalised, `test-tiers` risk added.
- board-fit, goal-scope: later tunings and `review-setup-audit` would re-diverge loomyard — fixed: stated out of scope in Scope.
- board-fit: `profiles`' run profiles hold loom bounce values — rejected: they equal the new template and seed-only stays out.
- goal-scope: log the units the strip kept — rejected: the strip's log names the dropped units, and what is left is visible in the file.
- leanness: the `models` no-op sentence in Scope — fixed: cut, kept here in Turn 1; D3's "no hand edit" repeat — fixed: cut from D3's rationale; D1's lens sentence — fixed: restated as a requirement; the tests-and-fixtures note — rejected: a useful hint to the plan writer.

## Rejected alternatives

- **Keep the base full and rewrite former template values to the new ones on reconcile.**
  Smaller change, but every future template change would need its own former-values entry, and loomyard's layer would keep every key, against the board entry.
- **Drop base units equal to the template on every reconcile, as `lenses` does.**
  No pin would be possible, and a former-values table applied on every walk would strip a deliberate later value such as `driver_kind: llm`.
- **Hand-edit loomyard's, quarry's and NORCE Models' hub files.**
  The NORCE Models hub is not reachable from this repo, and the walk reaches every hub on its next start verb anyway.
- **Run the base strip in every `Apply` caller of `ReconcileHubWideAt`.**
  Those callers commit only module files, so the marker would stay uncommitted on Bolt and another machine would strip again.
- **Treat a base holding every template key as unmigrated, to undo an older binary's refill.**
  A heuristic over file shape; deploying the binary on every machine prevents a refill, and removing the board marker by hand repairs one (D3).
- **Strip seed-only modules too.**
  Their file is the operator's registry, read with no template fill by `runprofile`; a strip would change what they resolve to.
- **Make the standalone base sparse too.**
  No hub reads it, and it would need a marker in the worktree's config dir beside the worktree `sparse` marker; nothing on the board asks for it.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Sparse base or full base with value migration? — auto-pick: sparse base (D2), since the board entry wants loomyard's layer to hold only the kept entries and future template changes then need no migration.
- Q2 How do existing hubs drop old neutral values? — auto-pick: a one-time, marker-gated strip against the current template and a former-values table (D3).
- Q3 Strip once or on every reconcile? — auto-pick: once, so a later deliberate value is a pin.
- Q4 Which paths must write the base sparse? — auto-pick: template seed, prime seed, `--set` and the editor (D4).
- Q5 Seed-only modules (`models`, `profiles`)? — auto-pick: out of scope; `models` has no value difference in loomyard.
- Q6 Standalone base? — auto-pick: unchanged.
- Q7 Fill log noise? — auto-pick: no fill log for a sparse base (D5).
- Q8 Which loomyard values are its own? — resolved by the board entry's split; `board` kinds and `language.skills` added as loomyard's own, since both name Go or lyx.
- Q9 PATTERN entry changes? — auto-pick: no existing entry line states the base is full, and the `PATTERN-config-strictness` background is updated with the docs (D6);
  round 1 added `PATTERN-sparse-hub-base`, since D4's rule binds every writer of a hub base, a later one included.
