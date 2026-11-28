---
round: 2
focus:
  - "The batch gate key's name: look at gateslot.BatchGateKey in internal/gateslot/tiers.go (including goReadGates), internal/runplan/cardgate.go, internal/weaverengine/strand.go and internal/webstercli/beginbatch.go, and search for other sites. Is the key name declared once and referenced everywhere it means the gate key, as PATTERN-code-conventions asks?"
  - "Narrowed versus whole-running batch tiers: look at ownPackageTiers, computeScope and the listed-packages and guard-step handling in internal/gatecli/scope.go, with tier handling in internal/gateslot/tiers.go. For a tier set to carry false, run with --no-carry, or whose command holds no target placeholder, do the printed notice, the recorded packages and the guard step match what runs?"
  - "Coverage of a narrowed batch tier: look at internal/gatecli/scopes_integration_test.go. Would a row fail if the main packages added by --main-packages, or the //lym:watch skip taken over the cards' own packages, were dropped from or swapped in the narrowed tier, and does each changed row check what its name states?"
  - "Scope wording against the code: look at the verify-model section of contracts/specs/loom-plan-spec.md, the --card usage string in internal/gatecli/cli.go, internal/gatecli's doc.go and _lym/landing/summary.md. Does each state the scope each batch tier gets, agree with the others, and, for repository documents and the change description, follow the rubric's Names of the tooling bullet?"
---
