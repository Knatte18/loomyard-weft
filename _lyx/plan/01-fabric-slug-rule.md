# Card 1 — fabric-slug-rule

**Create:**
- `internal/fabricengine#ValidatePairSlug`

**Edit:**
- `internal/fabricengine#TestValidateWorktreeSlug`

**Extends:**
- `internal/fabricengine/doc.go`

**Intent:**
Export fabric's pair slug rule (D14) so the board can apply the same rule fabric applies at `Add` and `Remove`.
`ValidatePairSlug` is a thin exported wrapper over `validateWorktreeSlug`, returning its refusal unchanged (still matching `ErrInvalidSlug`), so the rule keeps one owner and its reason text.
`TestValidateWorktreeSlug` gains rows that call `ValidatePairSlug` for a slug ending in the records suffix (`weftname.Suffix`) and for a configured junction name, asserting the same reason text and `errors.Is(err, fabricengine.ErrInvalidSlug)`, and one row that a plain slug passes.
The package doc names `ValidatePairSlug` as the one slug rule the board applies through its seam.

**ImpactSummary:** One new exported wrapper and test rows in fabricengine; no existing caller changes.
