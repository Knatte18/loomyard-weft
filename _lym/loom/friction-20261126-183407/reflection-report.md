# Reflection report: audit-flags-master-friction

## Filed

- https://github.com/Knatte18/loomyard/issues/260: flaky `TestGateTestMap/NamedPackageIsRebuiltWhenFresh` (`internal/gatecli`), from `burler-code-r1.md`.

## Dropped

All from `burler-code-r1.md`, as being about the task's own code:

- F1: the proposed rev-parse could not go through the existing Git seam, because `HeadSHA` reads through go-git, which cannot open the records subdirectory.
- F12: the proposed wording is refused by `TestRenderPrompts_FrictionDirective` with friction off.
- F2: the proposed fix would have created a terminal dead end after a hand merge between runs.
