# Fill missing config keys, keep orch pane, stop .gitignore writes

A deploy no longer leaves worktrees refusing to run, opening the prime in VS Code no longer collapses the orch pane, and `lyx ide spawn` no longer dirties a pair's tracked `.gitignore`.

## What changes

- A config file that lacks template keys now loads.
  `configengine`'s shared load body fills each missing key from the module's embedded template in memory and logs one Info line naming the module, file and key-paths.
  It never writes the file, so `lyx config reconcile` still writes new keys, prunes removed ones and materializes absent files.
  `Load` and `LoadOrTemplate` both get this, so every strict and degrading module is covered.
- `lyx reed add` gains `--unless-name <name>`.
  When a visible strand matching `<name>` has a live pane, the add is a successful no-op that reports the skip in its JSON envelope.
  A dead or hidden strand does not count.
  The check lives in the shared `reedengine`, so the tmux and psmux backends decide alike.
- The interactive folder-open chain in `internal/vscode` now runs `reed add --unless-name orch` for its `claude` row.
  A prime hosting `lyx orch` no longer gets a second strand stacked under orch, which had collapsed orch to `collapsed_rows` and stalled the idle probe.
- `lyx ide spawn` rewrites the interactive `tasks.json` on every call and keeps `.vscode/` out of git through `.git/info/exclude` (`fabricengine.PathTracked` and `ExcludeAnchoredDir`, as `SpawnDriven` already does).
  `settings.json` is still written only when absent.
  A tracked `tasks.json` is left alone: no file or exclude line is written, one warning is logged, and the spawn still launches.
- `internal/gitignore` is deleted; its only caller was the removed `.gitignore` write.
- The repo `.gitignore`'s `mill-managed` markers become a plain section heading; the entries stay.
- Template comments, `reed up` help text and docs that told operators to run `lyx config reconcile` to adopt a new key now say the key takes its default until reconcile writes it.
- CLAUDE.md's TEMPORARY items for #339, #340 and #334 are removed.
- `CONSTRAINTS.md` gains the missing-key rule in the Config Strictness Invariant and a new invariant that lyx never writes a `.gitignore` in a warp worktree.
  `docs/overview.md`, `docs/shared-libs/configengine.md`, `docs/shared-libs/yamlengine.md` and the reed package doc are updated.

Closes #339
Closes #340
Closes #334

## Design decisions

- The fill is `yamlengine.FillMissing`, not `Reconcile`.
  `Reconcile` builds its result on the template, so it drops file keys the template lacks and replaces mismatched shapes.
  `FillMissing` keeps every file key and fills only key-paths the file lacks under mapping ancestors.
- Present values are never replaced: empty strings, nulls and emptied lists keep their value, and a present list is carried whole.
- Still refused: absent `_lyx/`, absent file under `Load`, read errors, unparseable YAML, an unset `${env:NAME}` without a default, and a shape mismatch (named by key-path).
  A key missing inside an element of a present list also still refuses.
- `--unless-name` can only suppress an add; it never removes, resizes or refocuses a strand.
- The invariant covers the warp side only.
  `boardengine`'s `.gitignore` in `_board`, the weft's `main` worktree, is named as the one exception.

## Check by hand

- Worktrees spawned before this lands, the prime included, keep the old chain in `tasks.json` until `lyx ide spawn` runs once more there; until then the prime's workspace still collapses the orch pane.
  Open the prime in VS Code after that spawn with `lyx orch` running and confirm orch keeps full height.
- Any other strand added to the prime while orch runs still collapses the orch pane; this change removes only the folder-open trigger.
- A config whose author deliberately removed a key to mean "unset" now gets the template default instead of a refusal; no module documents removal as meaningful today.
- The real-tmux `reed add --unless-name` test runs only where tmux is available; the psmux backend is covered by the engine's fixture-table unit tests, not run live.
