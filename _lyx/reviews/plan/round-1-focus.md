---
round: 1
focus:
  - "Card 1 (vscode-attach-chain): its Intent declares both `TaskChainInteractive` and `TaskChainAttachOnly`, and its own `ideengine.Spawn` edit passes `TaskChainInteractive`, but its Create list names only `TaskChain` and `TaskChainAttachOnly`. Settle whether the card's target list is complete for what it creates."
  - "Card 2 (fabric-anchored-exclude): `ExcludeAnchoredDir` and `PathTracked` are two exported functions, each with its own tests and neither a helper of the other. Settle whether the granularity rule, under which an independently testable symbol gets its own card, calls for a split."
  - "Card 2: settle whether the `git check-ignore -q -- <anchorRel>/<dirName>/` short-circuit can be implemented as written. Does it give the stated exit codes on a directory that does not exist yet? Does it report a path with tracked content under `.vscode/` as not ignored? And is treating an operator's global excludes file as 'already ignored' consistent with the decision record's shared-exclude constraint?"
  - "Cards 2 and 3 against the decision record's acceptance criterion that the exclude write holds `fabricengine`'s exclude lock: settle whether any card's intent or tests carry that criterion, beyond routing through `mutateGitExclude` and asserting exact file content."
  - "Cards 1 and 3 both edit `internal/ideengine#Spawn`: card 1 changes its `WriteConfig` call and card 3 extracts a shared worktree/color resolution helper. `Spawn`'s prime-workspace branch needs `primeName` and `primeErr`. Settle whether the two edits compose, and whether card 3's claim that `Spawn`'s observable behaviour is unchanged holds for the helper as specified."
  - "Card 5 (batten-wire-open): the lifecycle and records integration tests will now run the real `SpawnDriven` against real pairs, writing `.vscode/` and a shared `info/exclude` line. Settle whether the card states what those tests must still assert (a clean tree at teardown, a successful worktree removal), or leaves it as a conditional 'fix it if it breaks'."
  - "Card 4 (innerrun-open-once) against the decision record's Run-Shed acceptance criteria: settle whether 'outcome, reason and bounce accounting identical' is fully carried by the card's `Reason`/`BudgetExempt` assertion, and whether the once-marker clear on `found == false` is placed consistently with the existing done-seen clear in `Call`."
---

Every entry was checked against the rubric's Do-not-flag list and against the plan-writer symmetry note.
None re-derives a mechanical check, asks for `DependsOn`/`Produces`, or asks for an `ImpactSummary` on a card type that does not require one.

Checked and left out:
verify coverage names all five targeted packages under both the untagged and the `integration` runs, and also runs `battenrecipe` and `loomcli` for their guards;
no `manifest/designs/` doc covers batten or ide, so card 5's docs scope matches the decision record's scope;
card 3's exclude-before-tracked-check order, which writes the entry even when the repo tracks `tasks.json`, falls under the repo-wide exclude risk the decision record already accepts.
