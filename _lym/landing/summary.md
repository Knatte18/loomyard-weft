# Red-first gate: restore embedded non-Go files instead of refusing

`lym gate red-first` refused to judge any diff that changed a non-Go file in a Go package.
A bug fix with a Go seam that also touched a spec or doc, such as `contracts/specs/raddle-proposal-spec.md`, therefore needed a "No test seam" waiver.

The base overlay now restores the changed non-Go files that a production Go file at the base embeds, with their base text.
Every other changed non-Go file keeps its working-tree text, as an input of the tests or nothing the build reads.
`gateslot.RedFirstUnjudgedExit` now means only a hub whose scoper is not `go`.

## Design decisions

- `baseOverlay` in `internal/gatecli/redfirst.go` no longer returns an unrestorable list.
  It decides per changed file whether the base's build reads it: a production Go file of a module package, or a non-Go file some production Go file embeds.
- `embeddedAtBase` reads the `//go:embed` patterns of the base's non-test Go files in the changed file's directory and in each directory above it, up to the module's root.
  `embedPatterns` reads the arguments as Go does: double- or backtick-quoted arguments run to the closing quote, and a malformed quote ends the directive's patterns.
- `embedPatternMatches` matches a pattern against the path below the Go file's directory, or against a directory that path lies under.
  A directory's subtree leaves out names that open with `.` or `_` unless the pattern carries `all:`.
  A pattern that names a file under `testdata` or a dot- or underscore-prefixed directory still counts.
- A changed non-Go file the code reads at run time, rather than embeds, keeps its working-tree text.
  This is a documented bound of the overlay in `internal/gatecli/doc.go`.
- The darn write stencil, `docs/overview.md`, and the godoc of `internal/gatecli`, `internal/gateslot` and `internal/loomshed` drop the non-Go-file cause of the unjudged exit.
  The `loomshed` red-first gate failure text now names only the scoper.

## To check by hand

- `embed-pattern-dir-matches-excluded-descendants` (design, medium), in `embedPatternMatches` in `internal/gatecli/redfirst.go`:
  the review left open whether a pattern that names a directory matches a changed file under it when an excluded dot- or underscore-prefixed name lies between.
  Check the match against the go command's own embed rules.

## Merge danger

**Door:** two-way
**Blast radius:** gatecli
