# Red-first: put embedded non-Go files back instead of waiving the gate

`lym gate red-first` refused to judge any diff that also changed a non-Go file under a Go package.
It exited with the unjudged code, so a bug fix with a real Go seam fell back to a "No test seam" waiver whenever a spec or doc changed beside it.
In `raddle-slug-length-check`, a change to `contracts/specs/raddle-proposal-spec.md` did this, although its new tests failed on the merge base.

A Go build overlay also replaces the files a package embeds: the go command resolves `//go:embed` files through the overlay.
So the base overlay can put such a file back, and the refusal had no reason to exist.

## What changes

- `baseOverlay` in `internal/gatecli/redfirst.go` now maps each changed non-Go file that a production Go file at the base embeds to its base text.
  A file the base lacks maps to the deleted path.
  The new `embeddedAtBase` reads the `//go:embed` patterns of the base's production Go files in the file's directory and each directory above it, up to the module root.
  A pattern counts when it matches the file, a directory the file lies under, or a glob matching either; quotes and an `all:` prefix are stripped.
- Every other changed non-Go file keeps its working-tree text.
  This includes a file that only a test embeds, testdata, and docs nothing embeds.
  These are the tests' own inputs, or files the build never reads.
- The "cannot put back" refusal is gone, and so is `holdsGoFileAtOrAbove`.
  `gateslot.RedFirstUnjudgedExit` now means only a hub whose scoper is not `go`.
  The verb's help, the package docs of `gatecli` and `gateslot`, the loomshed gate's failure text, `docs/overview.md` and the darn writer stencil say so.
- `gatecli/doc.go` names the new bound: a changed non-Go file that the code reads at run time, rather than embeds, keeps its working-tree text.

## Tests

- `TestBaseOverlay_ReplacesTheFilesOfTheModuleTheBasesBuildReads` replaces the old overlay table.
  Its rows cover a file embedded by name, under an embedded directory, through a quoted glob and from a package above.
  They also cover an embedded file the base lacks, and files left alone: one only a test embeds, one nothing embeds, and testdata.
- `TestGateRedFirst` drives the real go command over a hub fixture whose base package `spec` embeds `text.txt`.
  A changed non-Go file nothing embeds now leaves the verdict to the tests, which is green.
  A changed `text.txt` with a new test reading the embedded text exits 0, with that test red on the base's text.
  On the merge base, both rows get the unjudged exit instead.
- `TestRedFirstGate_VerdictPerLabelRecordAndExitCode` pins the exit-5 failure text to the scoper alone.

## Design decisions

- The precise rule restores only embedded files; overlaying every changed tracked file would not be safe.
  Deleting a new fixture that only a test embeds would break the test's build on the base.
  That would count as red and pass the gate with no proof.
- Embed patterns are read with a line scan of `//go:embed` directives, not `go list`.
  This keeps the verb free of another child process, and it reads the base's text, which is what the overlay builds.
- This run's own red-first gate is waived by the parent's recorded "No test seam" decision.
  The installed `lym` still holds the bug this change fixes, so it refused the stencil edit as a non-Go file it cannot put back.
  The new tests above fail on the merge base.

## Merge danger

**Door:** two-way
**Blast radius:** gate
