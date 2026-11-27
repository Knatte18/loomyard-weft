---
round: 1
focus:
  - "internal/planparser/latercaller.go, checkCreateCallerLater and its call in validate.go: Validate also runs on plans under execution. Settle whether a card already carried out, or a handle already resolved or rewritten, can raise or hide create-caller-later, judged against how the neighbouring checks treat done and pending cards."
  - "latercaller.go, refCanHoldProductionCaller and cardCanHoldProductionCaller: walk every ref shape the plan format allows against the real classification helpers (IsHandleRef, diskPathForRef, ParseRef, handleUnit). Shapes include a file path, an extensionless path, a directory, a self glyph, a member glyph, a bare symbol, a Move or Rename form, and a handle created on this or another card. Settle whether any is classed wrongly, so the check refuses a valid plan or passes one where the creating card cannot hold a caller."
  - "latercaller.go, isProductionFunctionDeclaration, declarationKeyword and cardNamesRef: settle whether the declaration-head and handle parsing holds for methods, generic functions, test-prefixed names, a head that mentions `testing.`, and a later card that names the function by a spelling other than the exact handle."
  - "Scope: the board entry under Goal reports a plan that could not pass its own gates and offers routes to handle it. Settle whether the diff's check handles the reported case, including its guard-file and later-batch shape, and whether anything else that has to know of a new check id or plan rule (registries, specs, stencils, the docs the constraints document names) was left behind."
  - "internal/planglyph/glyphchain_integration_test.go: the redundant-package fixture changed from a func to a type. Settle whether the test still checks what its case name and doc comment state, and whether other fixtures or plans in the tree are affected by the new check."
  - "internal/planparser/latercaller_test.go: settle which behaviors the diff adds have no coverage (each skip condition, each group type, each ref shape the code branches on), and whether any case passes for a reason other than the one its name states."
  - "Claims: check _lym/landing/summary.md, the new spec text in loom-plan-checks-spec.md and loom-plan-spec.md, the doc comments and the Root-Cause trailer against the code, including the list of refs that do or do not count and the Merge danger door and blast radius."
---
The range is a new plan-format check, its tests, two spec edits and one changed integration fixture.
The answer key is a decision record alone, holding the board entry with no added entries.
