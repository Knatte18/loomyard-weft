```yaml
lens: fix-proposal
outcome: reported
```

Findings:

1. **MEDIUM.** The F2 fix contradicts the D4 family-project exemption and its criterion.
   - **Changed text:** the new D4 bullet "It is refused while the slug's weft branch is live by the test `fabricengine.Topology.Add` adopts by: a local branch, …".
   - **Summary:** that refusal now rejects the case the D4 criterion admits: "`--record-file` on a family project whose raddle run has filed and keeps its worktree replaces the project's stored record".
   - **Evidence:** a kept task worktree means the pair stands. Its weft branch therefore exists locally, which is live under the test `internal/fabricengine/add.go:150` documents ("A pair is live when its weft branch exists locally…").
   - **Gap in the fix:** D4 exempts that project from the worktree refusal only. The new weft refusal has no matching exemption, so the criterion can no longer pass.
   - **Proposed change:** extend the exemption to the weft refusal for a filed family project. Its run reads no record again, so adopting its weft is harmless.

2. **LOW.** The F7 fix names a seam whose input D3 never supplies.
   - **Changed text:** the D3 lines "The filing counts as done when … or when the board already holds the filing as `boardengine.Board.HoldsRaddleFiling` judges it."
   - **Summary:** `HoldsRaddleFiling` needs the filing's part slugs, and with none it reports false. D3 never says where `add`, `replace` and `store` get those slugs, so as written that clause never refuses.
   - **Evidence:** `internal/boardengine/raddlefile.go:407-413` ("HoldsRaddleFiling reports whether the board holds every slug of slugs… With no slugs it reports false").
   - **What still works:** the "entry is no longer a task" clause covers the family case on its own.
   - **Proposed change:** name the slug source, such as the run's raddle proposal, or drop the clause.

The other proposals I tested held. The F5 cut handles the X, Y, X sweep case. The F1 split reaches every caller, because each history reader in loomcli goes through `recordHistory`. The F3, F4, F8, F11 and F12 fixes break nothing elsewhere in the artifact.

Tool calls:
- One grep for callers of `CommittedAnchoredFileHistory` and `recordHistory`, plus a read of `raddlefile.go:400-425`. My context showed neither the full set of callers nor the contract of `HoldsRaddleFiling`.
