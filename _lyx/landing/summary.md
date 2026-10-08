# Plan cards name members as glyphs, with new plan-gate checks

Plan cards now name the members they change as glyphs instead of file paths, so plan validation and the guards downstream work at member granularity.
Before the stencil pushes plans that way, loomyard's own use of quarry glyphs is covered by a committed test, and the cross-card checks that forced awkward plans are retired or replaced.

## What changes

- `internal/planglyph` gains a table-driven glyph-chain test over a committed fixture module in `testdata/glyphchain`.
  Each row is one member shape (package-level func, type, var and const, value and pointer methods, interface method, generic type, `_test.go` and build-tagged members, a directory name that differs from its package name, a `main` package under `cmd/`, var and const blocks, and a `plan:` handle created on one card and listed by later cards).
  Rows run through parse, validation, handle canonicalization, binding over a real commit's delta, done-checks and ref rewrite.
  The integration legs sit in the `integration` tier.
- Retired: `containment-unit-overlap`, `containment-file-overlap` and `handle-unreferenced`, with their spec rows, docs and tests.
  Two cards may now target the same file, or a member and its file.
  A declared handle that no later card uses is legal; it is still bound at record-batch and done-checked.
- New same-card redundancy findings, `redundant-package-target` and `redundant-file-target`: a card listing a package self glyph with a member or file of that package, or a file with a member that resolves into it.
- New re-sign arrow on `**Edit:**` member bullets, `` `<glyph>` -> `<new declaration head>` ``, parsed by `planparser`.
  The head must name the same member as the glyph (`resign-head-mismatch`); the arrow on anything that is not a member glyph is `resign-not-member`; an interface method or struct field takes no arrow (`resign-interface-method`).
- New caller coverage, `caller-uncovered`: for each `Delete` member and re-signed `Edit` member, every reference outside its own declaration must be covered by a target on the card.
  A `Delete` also accepts coverage by an earlier card; a re-sign needs the same card.
- The plan stencil (`loom-template-plan.md`) shows member-glyph `Edit:` targets, handles for every new member, the arrow and caller coverage.
  The Plan-Review rubric gains two findings: an `Edit` card with only non-test Go file paths while its Intent names symbols, and a signature change without the arrow.
- `loom-plan-spec.md`, `internal/planglyph/doc.go` and the `planparser` docs are updated to match.
- `CanonicalizeHandles` now derives the declaration for an interface method Rename and for a one-name var or const Rename, so both canonicalize.

## Design decisions

- The new redundancy file-versus-member check and `caller-uncovered` run only at the plan gates (`ValidateFormat` and `Validate`, hence `lyx loom validate-plan`), never in `ValidateDispatch`.
  Their verdicts depend on the tree, which earlier cards change, so a dispatch-time run could differ from the plan-gate verdict.
  The package-level redundancy pairings and the re-sign head match read only plan text and run at every gate.
- Caller coverage reads references with a `go/scanner` token walk over every `.go` file (tests and build-tagged files included, `testdata` and `vendor` excluded), so comments and strings never match; `deleteorder.go`'s line patterns are not reused.
  For a method or field the match is name-based and its finding is informational; for a package-level member it is blocking.
  A caller that imports the package under an alias or a dot import, or sits in a file an earlier card creates, is let through; webster's per-card build and test gate catches it.
- `Rename` Old sides and `Delete` by file or package are not caller-checked: the Rename mechanic rewrites references itself, and the build gate bounds the rest.
- A member's own package is the files of its directory with the same package clause; an external `_test` package reaches the member qualified.
- The arrow's head is parse-checked and member-matched only; nothing compares it with the built declaration.

## To check

- Known gap in quarry, not fixed here: it indexes no struct fields, so `shapes#Struct.Field` resolves `not_found`.
  The glyph-chain test asserts that current answer with a comment naming the gap.
- `cmd/lyx/testdata/test-budget.yaml` changed (the `internal/planglyph` row was raised) because the test-budget guard failed otherwise.
- `internal/planglyph/handle.go` carries a `resignHeadMismatch` helper and a `resign` field on `declSource`, beyond `CanonicalizeHandles` itself.
- Not run as given: the card gates ran `go test -tags integration` on the fixture packages inside the nested fixture module, which Go refuses with `[setup failed]`; the equivalent ran inside the fixture module instead.
  Run `go test -tags integration ./internal/planglyph/...` and the full `go test ./...` by hand.
