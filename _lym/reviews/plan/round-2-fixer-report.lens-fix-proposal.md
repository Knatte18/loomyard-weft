```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged the F1, F2 and F3 proposals, which the fixer carried out as proposed.

no findings

- **F1 (open maps `buttons.task`, `buttons.beam`, `buttons.prime`):** I looked for a case where the proposal breaks something and found none.
  - `yamlengine.Merge` carries an open-map key whole, so a layer's group replaces the default whole, as the card says.
  - `yamlengine/doc.go` already supports an open map holding a list: `--set buttons.task=[...]` replaces the list whole, and `--set buttons.task.<name>` is refused because the value at that path is a list. That refusal is existing designed behaviour, not something the fix breaks.
  - Prune and Reconcile carry open maps whole, so a worktree's own groups are never pruned against the template's list elements.
  - Desktop gets no open maps and keeps its intended refusal of a group written as a mapping.
  - The `configsync.go:914` branch already applies to reed because reed is not hub-wide, so the open maps change no seeding path.
- **F2 (Extends moved to Edit for `docs/overview.md` and `internal/toolfloor/doc.go`):** both cards change an existing line, so Edit is the correct kind. `lym loom validate-plan` passes.
- **F3 (one Intent sentence in card 5 for the bar_integration_test.go header):** this does not add a file target, which keeps `redundant-file-target` satisfied. Card 5 already edits test members in that file, so whoever implements it will see the sentence.

Tool calls:
1. Grepped yamlengine for open-map handling. My context did not have the merge or fill semantics for open maps.
2. Read the open-map branch of `SetSparse` in `yamlengine/layers.go`. I needed to know how `--set` behaves on a list-valued open map.
3. Read `yamlengine/doc.go` lines 40–50 and grepped for where `Known` is used. I needed the documented contract for open maps holding lists and where the known keys are shown.
