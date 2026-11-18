---
round: 2
focus:
  - "internal/editdirective/editdirective_test.go, TestDirective_ReadsTheEnginesCopyWithoutBanner: does any assertion still check the literal wording of a shipped edit-directive stencil, beyond the engine tool-wording, marker, leading-comment and trailing-newline checks Go chooses or fills?"
---

The round-1 fix removes the scratch-ban assertion from the test; round 2 checks that no stencil-prose assertion remains.
