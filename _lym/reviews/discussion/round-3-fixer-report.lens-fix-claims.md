```yaml
lens: fix-claims
outcome: reported
```

Task: check that the claims the fix added to D5 hold against the code at HEAD.

**Findings**

1. LOW. Changed text: D5, "A finished motif run turns its entry into a Sketch (`finishesAsSketch` in `internal/loomcli/wiring.go`)."
   - **Summary:** the sentence states as unconditional something the code can refuse.
   - **Evidence:** `sketchFinish` (`internal/loomcli/wiring.go:293-312`) catches `ErrSketchRefused` from `MakeSketch` and leaves the entry a draft task over its done motif run. `MakeSketch` (`internal/boardengine/sketch.go:85-99`) refuses through `validateWrite`. The kept batten-refusal clause ("when the slug holds a done motif run … `lym batten cleanup --apply`") depends on exactly that draft-task case, and the new sentence never names it.
   - **Suggested change:** add "unless the board refuses the Sketch, which leaves it a draft task (`sketchFinish`)".

**Claims that hold**
- Motif has no Finalize row and no Raddle-File row. Its engines are DiscussionWrite, Bouncer, BurlerReview and BurlerFix (`contracts/recipes/motif-recipe.yaml`), so `finishesAsSketch` is true for motif.
- "Only a raddle run takes a Sketch": `checkDiscussedPromotion` lets a discussed note with a stored record become a task only with recipe raddle, the discussed flag and the sketch label (`internal/boardengine/sketch.go:125-128`).
- "batten gains no motif-run clear for it": with the clear removed, `startClearsMotifRun` stays raddle-only (`internal/battencli/startguard.go:178`).
- The shortened Bound, "reads the board once at start": nothing added contradicts it.

**Tool calls**
- One grep of `contracts/recipes/motif-recipe.yaml` for its engines and `internal/boardengine/sketch.go` for the sketch label rule. My context had not shown motif's rows.
