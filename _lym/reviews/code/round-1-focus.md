---
round: 1
focus:
  - "contracts/stencils/gatefix/gatefix-body-card.md, the single-test re-run `lym gate test --tier <tier> <package> -- -run ...`, which carries no --gate key: does it run the same test, the same way, as the red card gate call did for every kind of test the card key can run (//lym:only and //lym:watch tests, tags, nested modules, a tier covered by another), and can it pass while running nothing?"
  - "The same prompt for failures that are not one named test (build or compile failure, package timeout or panic, TestMain failure, vet, lint, comment or pattern step): does the failure report name a runnable command and tier for each, so the fixer can re-run that check alone?"
  - "internal/gatecli: statements about when a guard step runs or what a --card scope holds, read against the new card-key behaviour, including the command's long help in cli.go (near the --guards sentence) and the `scope.guardStep` field comment in scope.go, not only the lines the diff touched."
  - "internal/gatecli/scope.go computeScope under `ownOnly`: how --guards, --main-packages, the vet step, watched-test skips, tag tiers, nested modules and a card tier above tier0 behave with `guardStep` false; with --guards forced, does the guard step cover what the listed runs leave out?"
  - "Hubs whose scoper is dotnet or none, where the rule keyed in computeScope's go path does not apply: do the spec, template.yaml comment, --card help, stencils and PATTERN-test-speed bullet claim an own-packages-only or no-guard-step card gate that those scopers do not give?"
  - "Coverage moved to the batch gate: for each flow that runs `--gate card` (Webster fork, recovery fork, Weaver batch, any path that resumes or stops before the batch gate), does a `--gate batch` call over the same cards always follow, and does it hold every dependent package and guard test the card calls used to run, including under a hub's own tier lists for the two keys and in nested modules?"
  - "contracts/stencils/webster/webster-body-implementer.md, the guard-test fix rule moved to the batch gate section: read it against the sanctioned-deviation sentence in step 2, the commit step's ` (fix)` form, the card-verify calls, the self-fix bound and the report status rules; is each still consistent for a fix made after the last card's commit, and does the Weaver batch template give its implementer the matching rule it needs?"
  - "_lym/landing/summary.md: check the guard-step trade figures and their stated basis against docs/research/webster/gate-time-2026-11-28.md and the current code (the amended entry conditions the move on the trade still holding, with the why in the change description), and check the Merge danger door and blast radius against what the diff changes."
---

Seed focus for round 1, set from the board entry, the diff over the review range and a few searches of the surrounding code.
It does not limit the review.
