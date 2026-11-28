---
round: 4
focus:
  - "D5 in _lym/discussion/decision-record.md, wherever it treats a jacquard start after a done motif run: can the board hold that state? Check finishesAsSketch (internal/loomcli/wiring.go), checkDiscussedPromotion and checkSketchLabel (internal/boardengine/sketch.go) and what sketchFinish leaves when the board refuses the Sketch. Then check that D5's Bound, the D5 acceptance criteria and the Constraints Facts about a motif run agree with what D5 now says."
  - "Every way forward D5 states for a done motif run or a pair it left standing: does each step succeed in the stated order? Follow the start guard (internal/battencli/startguard.go), cancel and childReachedParent (internal/battencli/cancel.go), cleanup's apply, the origin reconcile inside cancel (judgeOrigin's inputs once cleanup has removed the record) and Worktree-Create's leftover-branch refusal."
---
