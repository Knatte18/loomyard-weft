---
round: 2
focus:
  - "internal/gatecli/redfirst.go, embedPatterns and embedPatternMatches: settle whether a directory pattern without the `all:` prefix matches a descendant whose path elements below the named directory begin with `.` or `_`, as Go's embed excludes, and whether a changed test-only fixture of that kind is deleted on the base so the test's build failure counts as red."
  - "internal/gatecli/redfirst.go, baseOverlay and embeddedAtBase: settle whether a changed non-Go file under `testdata`, or a dot- or underscore-prefixed directory, that a base production file embeds is restored, and whether internal/gatecli/doc.go claims anything about such files that Go does not require."
  - "internal/gatecli/redfirst.go, embedPatterns: settle whether `//go:embed` directives separated by a tab, double-quoted names holding a space, and backtick-quoted patterns are read as Go reads them, and whether the tests exercise each form."
  - "internal/gatecli/redfirst.go, embeddedAtBase and baseOverlay: settle whether the failure of a base Go file listing or read is returned by baseOverlay and whether a test reaches each of the two failures."
  - "internal/loomshed/doc.go and internal/gatecli/redfirst.go: settle whether the package doc and the `redFirstCall.run` doc comment state the causes of the unjudged exit and the files the overlay restores as the code now behaves."
---

Round 1 raised a MEDIUM `design` finding on directory embed patterns, so round 2 looks first at what the fixes to the embed scan changed.
