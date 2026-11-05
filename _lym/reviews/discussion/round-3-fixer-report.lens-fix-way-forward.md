```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked that every way forward the fix added can be carried out from the state it names and leads to progress.

Ways forward checked:
1. A records mismatch at Drift-Verify or Drift-Recheck is a Stuck naming `lym drift amend --restore`. D11 admits the verb at those rows, and the restore amendment lets the check pass.
2. A records write interrupted before its commit shows as a mismatch, and `--restore` drops it.
3. The drift gate entry and `lym drift check` report a records mismatch without failing and check against the commit's copies.
4. The judge-tree Stuck lists its restores: a moved branch, a changed tracked file, an untracked file from the snapshot's copy, an added untracked path and a changed scratch capture directory, then `lym loom resume`.
5. A baseline digest mismatch after the judge returns is a Stuck naming `lym drift defer`. D12 admits `defer` at a Stuck Drift-Verify or Drift-Recheck whose digest mismatches, so this one is reachable.
6. A lost snapshot leads to a new snapshot and the earlier session's Stuck is dropped. This is reachable.
7. A run with no `Drift-Records` commit reads its files as they are. This is reachable.

Findings:

- **MEDIUM** — D9: "deleting a changed scratch capture directory, then `lym loom resume`" together with "A resumed row compares against the recorded snapshot, never a new one, until the trees match it."
  - Summary: this restore can never satisfy the snapshot compare, so the resumed row stays Stuck for good.
  - Evidence: the snapshot records a list of the scratch directory's paths and content hashes. Once the directory is deleted, it can never match that list again. The compare only passes "until the trees match it", and no text says a deleted scratch directory counts as matching or is left out of the compare.

- **MEDIUM** — D6: "a row that writes the records starts from those copies", together with D11's amend and D12's defer, which also write the records.
  - Summary: a non-restore `amend` or a `defer` run at a records-mismatch Stuck may commit the forged files under a new `Drift-Records` trailer, so the forgery passes.
  - Evidence: only rows are told to start from the commit's copies. `lym drift amend` (adding entries or `--discard`) and `lym drift defer` are verbs, not rows, and both are admitted at a Stuck Drift-Verify. If either writes onto the current, hand-edited `scope.json` and then commits, the edit is sealed into the newest `Drift-Records` commit. Only `--restore` is named as the way forward, but nothing stops the parent from choosing another verb at that Stuck.

- **MEDIUM** — D11: "`--restore` writes the seal commit's copy of `expected.yaml` back over the current file, and, when the records check failed, writes `scope.json` and `judged.json` back".
  - Summary: on a run with no seal commit, `--restore` has no seal copy to write, so the records-mismatch way forward may not be carried out.
  - Evidence: a run marked `unsealed`, a run after a `--discard`, or a run past Drift-Expect with no seal commit has no seal copy. The text makes the seal write unconditional, and it never says `--restore` skips it there and restores only the records.

- **LOW** — D9: "restoring a modified or deleted untracked path from the snapshot's copy".
  - Summary: the Stuck names the changed paths but not where the snapshot's copies are, so the person restoring has to work out the directory under the hub's board-level `.lym`.
  - Evidence: the copies' directory is only said to be under a name `internal/lymdirs` declares. `PATTERN-refusal-way-forward` asks for a way forward the reader can act on.

I made no further tool calls.
