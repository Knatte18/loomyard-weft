# Test-writing rules that stop the suite regrowing

`test-writing-rules` · Task · enhancement, loom, stencil

Depends on: [`test-suite-prune`](design-test-suite-prune.md)

Stage 2 of getting the test suite under control: stop it growing back after `test-suite-prune`.

Today's contracts drive one test per symbol: `contracts/specs/loom-plan-spec.md` and `contracts/stencils/loom/loom-template-plan.md` say every card that introduces behavior bundles its own test, and every independently testable symbol gets its own card.
Nothing asks whether a new test covers anything not already covered.

- Tests target behavior and contract at a module's public surface, not each helper.
- A new test must cover something no existing test covers; otherwise it extends an existing table test.
- A review fix adds a test only when the finding is a coverage gap; otherwise it corrects an existing test.
- The Bouncer and Burler rubrics treat a redundant test as a finding, like a missing one.
- A guard test pins a per-package budget for test count and wall time; exceeding it fails until the budget is raised deliberately in the same commit.
- Check whether `scribe:testing` needs the same change.
