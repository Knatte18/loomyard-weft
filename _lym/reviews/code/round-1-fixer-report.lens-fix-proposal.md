**Findings**

- **LOW**, `_lym/landing/summary.md`, last bullet of "Behaviour notes".
  - The F2 fix made `landUnderLock` read `parentHoldsWork` twice, so "now happens once" is false and disagrees with the "Helpers merged" bullet.
  - The second read's failure halts with `OwnerHaltPrefix` after the owner checks passed, so "halts as an owner check, which already ran first" no longer describes that path.
  - Fix: reword the bullet to say the land check's own halt is replaced by the owner-prefixed halt of the second read.

- **LOW**, `internal/landingshed/doc.go:209` ("Finalize reads Deps.ParentHoldsTaskWork first.").
  - The package doc stays correct for the first read but does not mention the second read after the owner checks.

Proposals with no case against them:
- F1: the shuttleparity gate passed, so the listed-holder scan is satisfied by the constant.
- F3: the split into a per-caller `Parse` wrap and `completeSummary` restores both original texts.
- F4: `Finalize.readAttempts` keeps its move-then-entry order; `ParentAlign.readAttempts` reads the entry first again.
- F5: the new row leaves each optional field unset by reflection and requires `Call` to succeed; a dropped or renamed name makes it fail.

Tool calls: none.
