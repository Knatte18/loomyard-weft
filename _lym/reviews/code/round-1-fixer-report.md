---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed-differently
    reason: The proposed first line of the run doc comment is about 140 characters, past the 120-character limit, so it is rewritten as two sentences on two lines.
---

## Changes

- F1, F2, F3, F4 and F6 share `internal/gatecli/redfirst.go` and `redfirst_test.go`, and go in one commit.
- F1: `embedPatterns` returns an `embedPattern` holding the glob and whether it carried `all:`.
  `embedPatternMatches` rejects an ancestor-directory match when a path element below the matched directory opens with `.` or `_`, unless the pattern carries `all:`.
  New rows cover a plain directory with dot-named, underscore-named and nested dot-directory files, an ordinary file under it, and an underscore file under `all:embedded`.
- F2: `baseOverlay` tests a non-Go file with the new `inModule` and lets `embeddedAtBase` decide; `.go` files keep `isModulePackage`, which now calls `inModule`.
  The line "An embedded file lies outside `testdata`." is gone from `internal/gatecli/doc.go`.
  The testdata row is renamed to `NonGoFileUnderTestdataNoGoFileEmbedsIsLeftAlone`, and a row for an embedded `testdata/golden.json` is added.
- F3: `embedPatterns` takes `//go:embed` followed by a space or a tab and reads arguments with `strconv.QuotedPrefix` and `strconv.Unquote` for quoted ones.
  Rows cover a backtick-quoted glob, a double-quoted name holding a space and a tab-separated directive.
- F4: `failingTree` in the test file fails `goFiles` or `file` on demand; `TestBaseOverlay_ReturnsTheErrorOfAFailedBaseReadWhileFindingEmbeddingFiles` asserts the wrapped error, its text and a nil map.
- F5: `internal/loomshed/doc.go` names "a hub whose scoper is not go", in its own commit.
- F6: the `run` doc comment names the embedded non-Go files, as two sentences on two lines.
- Gate repair: the `run` doc comment's clause-wrapped break and the test's three-line wrapped comment on the base's embeds are rewritten one sentence per line.
- Self-check follow-ups, same commit as the F6 line break: two sentences in `internal/gatecli/doc.go` state the dot, underscore and `all:` rule and that a `testdata` file counts when a directive names it.

`lym gate test --tier 0 ./internal/gatecli` passed before the self-check follow-ups; `go vet ./internal/gatecli` passed after.

- Reach gate: it names `redFirstCall.run` and `redFirstFailure` as reached by no mapped test.
  This round changed only the `run` doc comment, and did not touch `redFirstFailure`, so no test was added and the code stays unchanged.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW, `gatecli/doc.go` omits the dot, underscore, `all:` and `testdata` embed rules: fixed as proposed.
- fix-consistency, LOW, `loomshed/redfirstgate.go:33` generic wording: disputed as no finding; the text is true and the fork called it a mention only, so it is left as is.
- fix-proposal, MEDIUM, F6's proposed line passes 120 characters: fixed by rewriting it as two sentences.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
