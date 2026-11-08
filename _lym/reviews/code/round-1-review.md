---
verdict: APPROVED
---

## Result

No finding.
The diff adds a `//lym:watch` marker to 25 `//lym:guard` tests, adds one trigger sentence to each marked test's doc comment, and adds bullets to `pattern/PATTERN-test-speed.md`.
It changes no test body and no production code.

## Checked against the decision record

- **Every guard classified.**
  The module holds one `//lym:guard` test more than the 65 the change description lists, which is the string `TestPlanted` inside a test's planted source, not a test.
  The 25 marked and the unmarked tests together are exactly the module's guards.
  Each unmarked guard scans all Go code of the module, of `internal/` and `cmd/`, or reads inputs no pattern set bounds, so the goal's "leave unmarked" rule holds.
- **Patterns cover every input.**
  For each marked test I listed what it reads and matched it against its patterns.
  - Walk roots, pinned paths, allowlist keys and result-type tables sit in the scanned packages or in the test's own file.
  - The values the stencil test names are declared in packages the patterns list: `gatecli`, `shuttleengine`, `shedrun` and `websterengine`.
  - The registry, the embed declaration and the shared helper are under `contracts/stencils/*.go`, with `contracts/orchrole`, `internal/stencil` and `internal/stencilstore`.
  - The two directive tests also watch `internal/parentdirective` and `internal/editdirective`.
  - The retired-steps test watches the file holding the shared token matcher.
  - The denied-recovery test watches `internal/shellguard/*.go`; the `gateslot` and `agentname` constants that file imports feed a regex, not the forms the test compares.
  - The sandbox-coverage test gets module names from `newRoot()` in `cmd/lym/main.go`, which is the only non-test file building the root, and from each `internal/*cli` package.
- **Walk depth.**
  Each recursive walk's patterns reach one directory level below the deepest file scanned today.
  The only deeper files, `internal/planglyph/testdata/...`, are skipped by `scankit`.
- **Rename and delete.**
  `impactset.ChangedPaths` reports a renamed file by both paths, so moving a watched file out of its directory still runs the guard.
- **Comment conventions.**
  Each marker stack sits directly above `func Test…`, and each added sentence follows the repository's comment form.
- **`pattern/PATTERN-test-speed.md`.**
  The added bullets sit in "Narrow tests carry `//lym:watch`", state the rule the diff applies, and agree with the neighbouring guard section and `internal/gatecli/doc.go`.
- **Change description.**
  Its counts of marked and unmarked guards and its per-test tables match the diff and the module.
  I did not re-run its skip estimate.
- **Commit message.**
  It carries no `Pattern-Departure:` trailer, and the diff needs none.
