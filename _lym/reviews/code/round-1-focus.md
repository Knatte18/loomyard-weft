---
round: 1
focus:
  - "internal/landingshed/fixerrun.go `judge` and `haltReason`: does every path that ends `fixerEndDirty` set `outcome.dirty`? `s.halt(fixerEndDirty, ...)` is reached in one place; check that no other caller or resume path builds a dirty outcome with an empty list, which would print an empty path list."
  - "The board entry's second bullet asks for the cause (paths or state) in the halt reason instead of the generic wording. Does the new `fixerEndDirty` reason name the paths in the form the entry asks for, and does `listShown` cap them the way the package's other dirty-tree reasons do?"
  - "The board entry's first bullet asks to treat a passed gate plus a finished fixer turn as a clean end. The diff leaves the classification alone and the summary says commit b5e0ed26 already covers it. Does the code on this branch bear that claim out, and does leaving the dirty end a halt contradict the entry?"
  - "`internal/landingshed/fixerrun_test.go`: the `outcome != (fixerOutcome{})` comparison became `reflect.DeepEqual` because the struct now holds a slice. Is that the only comparison the new field breaks, and is the dirty row's `avoidReason` field an existing field of the table? Check the dirty row asserts the paths, not just the way forward."
  - "`internal/landingshed/doc.go`: the revised halt sentence. Does it read correctly against the code, given that the dirty halt applies to Parent-Align and Finalize alike, and does it still say that every halt opens with `LandingStopPrefix`?"
  - "The change description's claims about the timeout and no-report ends keeping the old wording, and about the dirty row asserting the old generic wording is absent. Check each against the diff."
---

The diff is small: three files in `internal/landingshed`, one commit, no plan.
The measuring stick is the board entry under `## Goal` in the decision record; its other sections are empty.
