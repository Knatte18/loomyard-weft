# Fixer report — discussion round 1

All eight findings are fixed in `_lyx/discussion/decision-record.md`, with two consistency edits in `support-log.md`.
`lyx loom validate-discussion` exits ok after the edits.

## Fixed

- **F1 (MEDIUM), D1 shape mismatch:** D1's bound now fills a key-path only where the file lacks it and every ancestor the file holds is a mapping.
  A shape mismatch in either direction still refuses, naming the key-path.
  A present value, null included, keeps its value.
  Acceptance criteria gain a fixture-test row covering both directions.
- **F2 (LOW), list-of-mappings element:** D1 says a present list is carried whole and nothing is filled inside its elements; a template key missing inside an element still refuses.
  The same refusal row in the acceptance criteria covers it.
- **F3 (LOW), Reconcile pointer:** D1 says keys the template lacks stay as the file has them, and a new criterion checks that an extra key survives.
  The Notes bullet now names the two `Reconcile` behaviours a reuse must avoid: its merge is built on the template, and its `added` list over-reports list-element keys.
- **F4 (MEDIUM), Goal vs residual spawn:** the Goal now allows one `lyx ide spawn` of each worktree spawned before landing.
  The Auto-mode assumption now says only a spawn regenerates `tasks.json`, so a later open from VS Code, recents included, runs the last-spawned chain.
  The Open risk adds that the prime is covered too, since D10 removes #340's workaround.
- **F5 (LOW), D2 liveness:** D2 defines "alive" as `--if-absent`'s set: a visible strand whose pane is present and not `pane_dead`, never a hidden one.
  The check sits in the shared engine with no backend-specific code.
  A new cross-platform unit criterion covers the skip decision.
- **F6 (LOW), tracked branch:** D4 says `PathTracked` runs before any write.
  A tracked `tasks.json` means `Spawn` writes neither file and no exclude line, logs one Warn and still launches.
  The acceptance criterion is split to cover the absent-`settings.json` tracked case.
- **F7 (LOW), D6 exemption:** D6 now binds warp worktrees, the prime included.
  Its new Bound names `boardengine`'s writer in `_board`, the weft's `main` worktree, as the one weft-side writer.
  It says it extends the junction bullet rather than generalizing it.
  The D6 heading, the Scope Out bullet, support-log Interview item 8, item 10 and Q10 are aligned.
- **F8 (LOW), D8 criterion:** a new acceptance criterion requires that no template comment, Go string or doc under `internal/`, `docs/` or `contracts/` tells an existing hub to run reconcile to adopt a new key.
  It also requires that `Load`'s absent-file error still names `lyx config reconcile`.

## Deferred

None.
