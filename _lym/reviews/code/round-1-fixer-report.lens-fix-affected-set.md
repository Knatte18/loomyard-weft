No findings.

Searches rerun:
- F5: every other statement of `RedFirstUnjudgedExit` already names the non-go scoper alone (`gateslot/doc.go`, `gatecli/redfirst.go`, `gatecli/doc.go`, `docs/overview.md`); the remaining generic phrasings (`loomshed/redfirstgate.go:33`, `shedrecipe/doc.go:209`, `contracts/stencils/darn/darn-template-write.md:111-112`) claim no diff-based cause.
- F2: the removed sentence "An embedded file lies outside `testdata`" has no twin; the `gatecli/doc.go` section matches the new behaviour; the callers of `isModulePackage` keep their behaviour.
- F1 and F3: `embedPatterns` and `embedPatternMatches` have only `embeddedAtBase` and the tests as callers; `goEmbedDirective` has no other user.

Unchecked: whether the repository's real `//go:embed` lines all parse the same under the new reader.

Tool calls: 3 (grep for stale phrases; grep for `isModulePackage(` and `go:embed` callers; read of `gatecli/doc.go` lines 862-900 and grep for `RedFirstUnjudgedExit`).
