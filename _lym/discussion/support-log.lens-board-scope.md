```yaml
lens: board-scope
outcome: reported
```

Lens board-scope: checking the decision record's fit with the board entry, its peers and the parent commit.

No findings.

Goal points of the entry:
- The checklist sits in a window of its own whenever a pawl run lives: met by D1, D3 and D4.
- That window is named for pawl: met by D3, which titles it with the recipe name, `pawl`.
- It sits beside the beam overview, with the beam overview first: met by D1's order key and D3's key after the runs strand's.
- The runs strand no longer takes the checklist's window: met by D4. The entry offered re-adding the checklist or reordering the starts; the record takes neither, because D1 makes both unnecessary. That fits the orch's recommendation in the same entry.
- Both overview windows get the plain style and the overview focus rules: met by D1, which keeps every overview property, and D2.
- The optional line in the beam overview naming pawl's current row: left out on purpose under Scope. The entry says "may", so this is no finding.
- The split of the one overview window into two panes: excluded, as the entry decided.

Entries, peers and the parent commit:
- `tapestry-motif-block`: no conflict. It removes the arras recipe, which this run itself runs under, but the record's design does not depend on arras. That is a fact about the run, not a contradiction in the record.
- `batten-remote-order`: no conflict, since it lists no changed paths.
- `reconcile-migrates-profiles`: no conflict. It changes config, runprofile and hubreconcile code, none of it in reed, orchcli or loomcli.
- `drift-scope-in-run-settings`: no conflict. It changes drift code and the orch role contract, not the overview or the strands.
- E8 (`orch-role-making-a-beam`) and E11 (`parent-merge-task`) carry the pawl label but concern making a beam and merging from the parent: no conflict.
- Parent commit `486bbf4` ("Preflight: stop refusing lym's own run records under _lym"): unrelated, no conflict.

Decision notes: no `Decision for pawl-checklist-hidden-in-beam:` note is on the board, so there was none to read and none I could not reach.

Tool calls:
- `lym loom peers`: my context lacked the peer list and the parent commits.
