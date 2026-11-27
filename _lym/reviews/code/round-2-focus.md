---
round: 2
focus:
  - "internal/gatecli/scope.go, scanGuards, computeScope and the tier set they share: for a --card call whose scope includes packages that depend on the card's packages, does every //lym:guard test run somewhere, in a listed tier's run or in the guard step under its own tier? Settle it for each combination of listed, covered and dropped tiers, including a narrowed scope, and for diff scopes that use the same code. Check that internal/gatecli/doc.go, the command help and _lym/landing/summary.md state the rule the code applies, and that a test would fail if it were broken."
  - "internal/gatecli/scope.go, cardDependents: when go list cannot list a module, is the caller told that the gate covers the card's packages alone, and do the doc comment, internal/gatecli/doc.go and the test agree with what happens, including a call over more than one module?"
  - "contracts/stencils/webster/webster-body-implementer.md, the sanctioned deviation for failures in files a card does not list: does its wording match which runs of a card gate can now fail outside the card's targets, and does it keep every limit the stencil set before this branch that the board entry did not ask to change?"
  - "A card whose targets are non-Go files that a package embeds or reads, in a directory with or without a .go file of its own: trace planparser.CardTargetDirs and CardPackages against the diff scope's nearest-enclosing-package and graph rules, and settle whether the card scope reaches the packages the board entry expects."
  - "internal/impactset, DependentPackages, with the listed tier commands and vet runs in internal/gatecli: can every package it adds be handed to each of them without error, including a package whose files are all tier-tagged, a test-only package and a platform-constrained one?"
---

The first two entries and the third cover the places where the widened card scope changes what a gate runs or tells its caller.
The last two are cases the earlier review did not drive.
