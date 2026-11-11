---
round: 1
focus:
  - "internal/gatecli/tiers.go and run.go: settle whether every route that carries a requested target (explicit project, --card, batch gate, --gate card) is refused when the selected tier command holds neither {targets} nor {test_targets}, and whether any route still silently runs the unfiltered suite."
  - "internal/gatecli/tiers.go: settle whether the refusal applies only when a target was actually requested, so a plain tier run with no target and a placeholder-free command still passes."
  - "internal/gatecli/dotnetscope.go: settle whether the unreached-project warning correctly detects a card's added or edited test projects that no configured tier command reaches, including the case of an empty tier1, and whether it warns rather than refuses."
  - "internal/gatecli/dotnetscope.go: check failure paths such as unreadable project files, projects with no tier match, path normalisation and non-dotnet targets, and whether they are handled without a false warning or a panic."
  - "internal/gatecli/test_integration_test.go and tiers_test.go: check whether changed assertions still pin the earlier behaviour they replaced, and whether new tests duplicate each other's coverage."
  - "internal/gatecli/doc.go: settle whether the documented behaviour matches what tiers.go and dotnetscope.go do after this diff."
---

Seed from the board entry in the decision record's `## Goal`, which is the measuring stick.
Its two asks are a refusal or warning when a requested target cannot reach the tier command, and a warning for test projects no tier reaches.
The record's other sections are empty.
