---
round: 2
focus:
  - "internal/boardengine/holdwrite.go, `FixHold`: when the hold is lost or no longer stands as written, does the refusal's way forward name every subject of the hold (via `holdSubjectNames`) rather than only the failing subject, given that `planAdoptHold` refuses a re-run over any other set than the hold's? Does the FixHold test cover a two-subject hold whose second subject fails?"
  - "internal/fabricengine/takeoverwrite.go, `rewriteOverOrigin`: does a branch with no upstream return an error that says so, with no `%w` over a nil error, and does the drop-entry join still apply on both paths?"
  - "internal/adoptcli/cli.go, `Command`'s `Long` text: does it describe a plain adopt as refusing while another machine holds the hub, matching what `WriteAdopt` does?"
---

Round 1 raised these three sites; the fixes since then are new text to read against the code.
