# Fixer report — plan, round 1

## Changes

- **F1** (fixed as proposed): in card 3's Intent, `buildConfigLong` gains only the `machine` token in its two lists, as `--layer default|worktree|machine|local` and `layer: template, default, worktree, machine or local`, and the Intent names the 2000-byte `maxLongBytes` cap.
  The gloss goes into the `--layer` flag help in `Command` and into `configcli/doc.go`, never into Long.
  `00-overview.md`'s verify gate line now ends `./internal/configcli ./internal/hubreconcile ./cmd/lyx`.
- **F2** (fixed as proposed): card 1's `ValidateMachine` list gains a `CategoryNone` bullet, `key %q is not a machine key`, naming both the hub file's `lyx config <module>` and `lyx config <module> --layer local`, and the other-category bullet keeps the category text.
  A new sentence says that "remove it from the machine override" applies to a file that already holds the key, while a `--set` refusal writes nothing.
  The `TestValidateLocal` row for `name` asserts `not a machine key`, `lyx config mod` and `--layer local`.
- **F3** (fixed as proposed): card 1's Intent gains a Bound paragraph after the `ValidateLocal` sentence.
  It covers the prime-only write guard of card 3, the direct-write bypass, that the file never reaches Bolt or any record, and that the loader floor of 1 applies with no ceiling.
- **F4** (fixed differently): the review proposed that `ConfigFiles` also return the local file.
  I did not make that change, because card 2's `GateSlots` wrapper appends ` and <file>` for every `ConfigFiles` entry after the first, so the local file would leak into that wrapper's text.
  Instead, `withConfigWayForward` gains a separate `local string` argument (Edit signature arrow updated, and `configengine#LayerLocal` added to Uses).
  `LoadConfig` passes the local file only when `LayerFile` applies, the anchor holds `_lyx/` and `os.Stat` succeeds.
  With that argument set, both forms insert `; remove <local>, which takes no gate key` ahead of the role clause.
  The two-file form now reads "the hub file <hub>, the machine file <machine> or the file the error names holds the value".
  The `TestLoadConfig` local-layer row asserts the removal step.
- **F5** (fixed as proposed): card 1's `PATTERN-config-layers.md` edit rewords the "one of four categories" and "none of the four" sentences without a count.
- **F6** (fixed as proposed, with one wording change): card 3 states that `setMachine` and `editMachine` report a load-check refusal through `newLoadFailure`, and that its trailing `way forward: --set the key to a value it takes` is the operative step.
  The `TestDispatchLayers` rows for `--set go_parallel=0` and the `go_parallel: 0` editor save assert that this clause comes after the loader's text.
  They do not assert it as the message's end, because `setChecked` wraps the error as `"%w; %s unchanged"` (`internal/configcli/configcli.go:259`).

## Self-check

- fix-consistency, MEDIUM: the new card 1 `--set` sentence claimed a set step that the remove-only refusals lack. Fixed as proposed: the sentence is now limited to covered units, and every other refused `--set` is resolved by not setting the key there.
- fix-consistency, LOW: card 3 left the `--layer` flag help's own layer list unstated. Fixed as proposed: the list becomes `default, worktree, machine or local`.
- fix-claims, LOW: card 2 could name a local file that the load skips for an anchor with no `_lyx/`. Fixed as proposed: the file is named only when the anchor holds `_lyx/`.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
