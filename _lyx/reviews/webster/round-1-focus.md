---
round: 1
focus:
  - "internal/loomengine/template.yaml: settle whether `review` is `opus[medium]` and `webster_review` is `sonnet[high]`, with the other per-segment keys still `\"\"` and `fix`/`judge` unchanged, and whether each changed line's key comment still describes the new value."
  - "internal/loomcli/wiring_test.go `TestWire_ReviewKeysReachTheEnv`: settle whether the Webster-Burler row is asserted as one sonnet entry of effort high with `Fix` equal to the run-wide fix list, whether the run-wide loop still covers the right rows, and whether the doc comment states both."
  - "internal/loomengine/config_test.go `templateConfig`: settle whether it pins `Review` and `WebsterReview` to the new values and the other five per-segment lists stay `ModelSpecList{\"\"}`."
  - "Card 10's closing instruction: settle whether any other test or fixture in the tree still pins the old template `review: sonnet[medium]` or an empty `webster_review`."
  - "Test economy: settle whether the new wiring assertion duplicates coverage an existing config test already gives, or leaves the template-to-env path for `webster_review` uncovered."
---

Round 1 seed. The diff is one commit touching two template values and the two tests that pin them.
