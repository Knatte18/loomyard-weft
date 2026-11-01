# Round 1 fixer-report

## Changes

Each finding is one commit, `d9e8aad2` to `06707ebe`, and the self-check fixes are the last commit.

- **F1** fixed as proposed.
  `Set` and `Remove` act on a slug the store holds and refuse a shipped slug only for a new entry.
  `refuseShipped` and the verb help and package docs say so.
  Rows added: store collision test, render collision row, CLI collision test.
- **F2** fixed as proposed.
  `ReadLayers` falls back to no overrides and sets `Layers.OverrideWarning`; `Render` warns and `list` prints `warnings`.
  `Layers.Ordered` ignores a blank override text.
  Rows added: malformed and blank override render rows, a CLI list row.
- **F3** fixed as proposed, through a non-parallel test that replaces `shippedStore` with a shipped layer holding a `claude-prompt` entry and restores it by `t.Cleanup`.
  It pins Show and Passages with the override on, off and absent, and the shipped-first order.
- **F4** fixed as proposed, by the stated option: the pattern entry, `doc.go` and `Claims` state that the override file is replayed by plain cherry-pick, and a `Claims` row pins it unclaimed.
  I did not make `Apply` merge the override store: override entries carry no order or targets, so `ValidateEntry` would refuse them and the merge would need its own validation.
- **F5** fixed as proposed: `shipped_test.go` covers the three `decodeShipped` panics, and two render rows cover the off default with no `heddle.yaml` and a stale override slug.
- **F6** fixed as proposed: `ShippedBatchingEntryIsPinned` in `store_test.go`.
- **F7** fixed as proposed in `claudeengine/doc.go`.
- **F8** reverted first, then restored after the gate: `TestCLIForm_EveryCommand` caps the `lym config` Long at 2000 bytes, and the old wording ("JSON success output's") made it 2007.
  The shorter wording in `buildConfigLong` is required by that test, which is the exception the finding's `fix` key names, so the line keeps the change from `e0aa42dc`.
  Commit `fix F8 gate` restores it.

## Self-check

All four forks returned a report.

- fix-code-correctness: no findings.
- fix-consistency, MEDIUM: `PATTERN.md` index line for `PATTERN-heddle-instructions` still said the replayer writes the Bolt files. Fixed as proposed.
- fix-consistency, LOW: `heddlecli/doc.go` `list` paragraph lacked the warnings sentence. Fixed as proposed.
- fix-affected-set, LOW: `shuttleengine/doc.go` and `shuttleengine/passages.go` repeated F7's stale "Bolt checkout holds" claim. Fixed as proposed.
- fix-affected-set and fix-proposal, LOW (one merged finding): `get` returned the shipped entry only, so a colliding repository entry could not be read.
  Fixed with my own change, not a change to `Set` or `Remove`: `get` prints the entry the renders use, plus the other entries under the slug in `shadowed`; the CLI collision test pins it.
- fix-proposal and fix-affected-set, LOW (one merged finding): `get` (fixed), `Show` and `Passages` drop `OverrideWarning`.
  `get` now prints `warnings` like `list`.
  `Show` and `Passages` are disputed below.

## Disputed

- Self-check LOW, `Show` and `Passages` drop `OverrideWarning`: ground `every fix worse`.
  `Show` writes only the rendered text to stdout (`verbs.go` `showCmd`, `io.WriteString(c.out, text)`), so a warning would corrupt the text an operator pipes.
  `Passages` returns `[]string` that `claudeengine` appends to a session's system prompt, so a warning placed there would become instruction text; a changed return type would touch every `PassageReader` caller outside this diff.
  `Render` and `list` carry the warning, and a hub with a bad override file sees it on the next render.

## Deferred

Nothing deferred.
