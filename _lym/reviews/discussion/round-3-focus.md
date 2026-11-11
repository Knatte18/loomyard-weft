---
round: 3
focus:
  - "D2 and D5 file sets: for a covered package, does the recorded and compared file set equal every tracked file impactset maps to that package (every file under its directory outside nested packages, go:embed targets in subdirectories included), so an edit to an embedded non-Go file in a subdirectory, such as under contracts/stencils/loom/, cannot narrow importers to nothing selected?"
  - "D2, D4 and D5 compared packages: is every in-module package the test binary links recorded and compared, including a package with no covered block such as one holding only constants or types (internal/lymdirs declares no function), so a const or type change there is a miss and not none selected?"
  - "D5 non-body miss: does the record state, with a source, how often landings on main change widely linked packages outside function bodies, and does the rule leave a plain added declaration no unchanged code can reach out of the miss list, as a plain added function already is?"
  - "D4 changed-function rule against D5: is a changed function defined by its whole declaration text (receiver, signature, body), so a receiver or signature change with an identical body selects tests or misses?"
  - "D5 comment exception: can a changed //go:embed, //go:build or //go:linkname line pass as a comment and select nothing?"
  - "D3 union and D6 selected invocation: can a package's union hold tests outside the tier or tests the diff's watch skip removes, so the selected invocation runs zero tests and the existing empty-tier check fails the gate?"
  - "D2 staleness and the Open risks line about note testmap-upkeep: is the question for the parent worded so the writer can act on the parent's recorded answer either way?"
  - "D2 recorded hashes at the map's Commit: can an edit to a covered package made during testmap's walk and reverted later enter the map under Commit's hashes, given testmap reads the prime's dirty files once before the walk?"
---

Focus covers the round-2 gating design gaps and the open parent question.
