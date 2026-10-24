```yaml
lens: fix-way-forward
outcome: reported
```

Task: check, through the fix-way-forward lens, the ways forward this round's fix added.

Ways forward introduced:
1. D10 step 4: when uncommitted weft content sits outside the record pathspec, the step is refused, naming the paths and "commit or discard in the weft worktree".
2. D13: a ready task whose beam orch is stopped or never started gets no notice and waits until the operator, or the beam's orch once started, reads the board.
3. D13 acceptance criterion: a ready task of a beam whose orch is stopped is named to the prime's orch not at all.

Findings:

- MEDIUM. D10 step 4: "refuses the step without force". `lyx beam promote` has no force flag anywhere in the record, so "without force" points to a way past the refusal that the verb doesn't offer. The record mentions force only at lines 169 and 170, the two lines this fix added. The phrasing is copied from `Remove`, whose refusal names `--force` (`internal/fabricengine/remove.go:939`: "commit or remove them, or use --force"). Either drop "without force" or give `promote` a `--force` and name it in the refusal.
- LOW. D10 step 4: the refusal's way forward is only "commit or discard in the weft worktree". It never names re-running `lyx beam promote <slug>`, which is the step that actually moves the promote forward. Steps 1–3 have already run when this refusal fires, so the entry is an open, unclaimed task that the start guard refuses, naming `lyx beam promote`. The re-run is reachable, but the refusal itself doesn't name it.
- LOW. D10 acceptance criterion: the new step-4 refusal for uncommitted content outside the pathspec has no criterion that reaches it. The Constraints bullet PATTERN-refusal-way-forward requires that "every new refusal names its way forward, and a test reaches it". The D10 criterion adds only the case where the uncommitted file is a record file and stays reachable.

Tool calls:
- One grep of `internal/fabricengine/remove.go:939` for `Remove`'s sibling-dirt refusal text, plus the record's mentions of "force". My context showed only the doc comment for `ErrPairSiblingDirty`, not the text the refusal prints.
