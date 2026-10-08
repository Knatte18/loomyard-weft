# Card 11 — caller-coverage

**Create:**
- `internal/planglyph/callercoverage.go`
- `internal/planglyph/testdata/glyphchain/callees/callees.go`
- `internal/planglyph/testdata/glyphchain/callees/local.go`
- `internal/planglyph/testdata/glyphchain/callees/callees_external_test.go`
- `internal/planglyph/testdata/glyphchain/callers/callers.go`
- `internal/planglyph/testdata/glyphchain/other/holder.go`

**Edit:**
- `plan:internal/planglyph#planGatePass`
- `internal/planglyph/deleteorder.go`
- `internal/planglyph/doc.go`
- `internal/planglyph/glyphchain_test.go`

**Prosa:**
- `contracts/specs/loom-plan-spec.md`

**Uses:**
- `plan:internal/planparser#CardResign`

**Intent:**
Add D5's caller coverage, the check `caller-uncovered`, to `planGatePass`, so it runs at the plan gates and never in `ValidateDispatch`.

Subjects: every `Delete` group ref, and every `Resigns` target, that is a member glyph whose answer (through `answerSymbols`, from `planGatePass`'s one batched resolve) holds its declaration.
A `Rename` Old side, and a `Delete` of a file path, file self glyph or package self glyph, are not subjects.

`callercoverage.go` holds the reference walk, run only when a subject exists, over every `.go` file under the worktree root, `_test.go` and build-tagged files included.
It skips directories named `testdata` or `vendor` and directories whose name starts with `.` or `_`, which the go tool ignores as well, so no compiled caller lives there; non-Go files are not searched.
Each file is tokenized with `go/scanner`, so comments and string literals never match, and its package clause is the identifier after `package`.
- A package-level subject named `N` (no owner): its own package is the files of its declaring file's directory whose package clause equals the declaring file's; there, a reference is an identifier token `N` not preceded by `.`.
  Everywhere else, an external test package of that directory included, a reference is the token sequence `<package clause>` `.` `N`.
- A method subject (an owner chain): a reference is `.` followed by `N`, anywhere.
- An occurrence inside one of the subject's own resolved spans is its declaration, not a reference.

A reference at a file and line is covered by a target of an admissible card: a file self glyph whose `UnitPath` is that file, a package self glyph whose `UnitPath` is the file's directory, or a member glyph whose resolved span holds the line.
A re-signed member admits only its own card, since no earlier card can call a signature that does not exist yet; a deleted member admits its own card and every earlier card.
A deleted member's reference inside a later card's Edit region stays `delete-before-reference`'s alone: extract `LaterDeleteReferences`' Edit-region construction in `deleteorder.go` into a helper both checks call, so the region means the same in both.
A re-signed member's reference there is a `caller-uncovered` finding.

One finding per subject and file, attributed to the subject's card, with `Ref` the subject, naming the file and its lines.
Its way forward: list the file, or the member glyph whose body holds the reference, on that card, or for a deleted member on an earlier card.
It is blocking for a package-level member and informational for a method, since a name-based scan cannot tell receivers apart.
A file that cannot be read is an `ErrQuarryUnavailable`-wrapped error, as `deleteorder.go`'s reads are.

What this lets through, bounded:
- Dispatch never refuses on `caller-uncovered`, so an uncovered caller that first appears after the plan gate passes there; the check stays out of `ValidateDispatch` because the reference set shrinks as earlier cards land, which would change a verdict mid-run.
  Webster's per-card build and test gate fails the card that breaks such a caller.
- A cross-package caller importing the package under an alias or with a dot import, and a caller in a file an earlier card creates, are not seen.
  Webster's per-card build and test gate fails the card that breaks such a caller, and a planner who knows of one covers it like any other caller.
- A method's callers are reported only informationally; the same gate bounds them.
- A `Rename` card's callers are rewritten by the spec's Rename mechanic and checked by its grep verify and the build gate.
  A file or package `Delete` stays with `delete-before-reference`'s import match against later cards' Edit regions and the build gate.
- In its own package the scan also matches a local identifier, a struct-literal key or a method declared under the same name; the card clears it by listing the member glyph whose span holds the occurrence.

Fixture: `callees/callees.go` (`package callees`) declares `Target`, `Helper`, `UsesHelper` (calling `Helper` in the same file), `Thing` with `Method`, and `NotACall(h other.Holder)`, which names `Target` only in a comment, a string literal and the selector `h.Target`.
`other/holder.go` (`package other`) declares `type Holder struct{ Target int }`.
`callees/local.go` calls `Target()`; `callees/callees_external_test.go` (`package callees_test`) calls `callees.Target()`; `callers/callers.go` (`package callers`) declares `UseTarget`, calling `callees.Target()`, and `UseMethod`, calling `t.Method()`.

Rows in `glyphchain_test.go`:
- Deleting `callees#Target` alone: blocking findings for `callees/local.go`, `callees/callees_external_test.go` and `callers/callers.go`, nothing for `callees/callees.go` or `other/holder.go`; `ValidateDispatch` on the same plan reports none.
- The same card also listing `callees/local.go`, `callees/callees_external_test.go` and `callers#UseTarget`: clean.
- Those covers on an earlier card (with `callers#` in place of `callers#UseTarget`) and the delete on a later one: clean.
- Re-signing `callees#Target` with the covers only on an earlier card: blocking; with the covers on its own card: clean.
- Deleting `callees#Helper`: blocking for `callees/callees.go`; with `callees#UsesHelper` on the same card: clean.
- Deleting `callees#Thing.Method`: informational for `callers/callers.go`.
- The shape rows' `shapes#Generic` Delete and arrow legs stay clean unchanged: `GenericMethod`'s receiver is a reference to `Generic`, covered by the `shapes#Generic.GenericMethod` glyph cards 1 and 10 list beside it.
- Renaming `callees#Target`, deleting `callees/callees.go`, and deleting `callees#`: no `caller-uncovered`.
- Deleting `callees#Target` with a later card editing `callers/callers.go`: `delete-before-reference` for that file and no `caller-uncovered` for it; re-signing it instead: `caller-uncovered` for that file.

`doc.go` gains the bullet; the spec gains the row (emitted at `ValidateFormat`, `Validate` and `ValidateRework` only), renumbered with every `row N` cross-reference fixed, and the Card types table's Delete and Edit mechanical checks name it.

**ImpactSummary:** Adds one plan-gate check that walks the tree only when a plan deletes or re-signs a member; delete-before-reference keeps its behavior through a shared region helper, and dispatch validation is unchanged.
