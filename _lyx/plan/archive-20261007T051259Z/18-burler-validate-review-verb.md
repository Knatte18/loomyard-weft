# Card 18 — burler-validate-review-verb

**Create:**
- `internal/burlercli/validate.go`

**Edit:**
- `internal/burlercli/cli.go`
- `internal/burlercli/cli_test.go`
- `internal/burlerengine/doc.go`
- `pattern/PATTERN-gate-self-check-parity.md`
- `docs/overview.md`

**Uses:**
- `plan:internal/burlerengine#CheckReviewFile`
- `plan:internal/burlerengine#ReviewGate`

**Intent:** D12, verb half: `PATTERN-gate-self-check-parity` binds the new review gate to a self-check verb, so `lyx burler validate-review <review-file>` lands in the same task.
This is the one new verb the decision record admits.

`validate.go` builds the cobra subcommand: `Use: "validate-review <review-file>"`, a `Short`, a `Long` naming the review gate it mirrors, `cobra.ExactArgs(1)`.
It resolves a relative path against the seam cwd (`lyxcwd.CwdFrom`), calls `burlerengine.CheckReviewFile`, and prints `output.Ok` with the absolute `review_file` on success, or `output.Err` carrying the error verbatim with exit 1 on failure.
It is read-only, writes nothing and has no flag that skips the parse.

`cli.go`: register the verb, and make `resolvePersistentPreRun` skip resolution for it as it already does for the group command, since a read-only check of a told file needs no hub, mode or engine wiring.

`internal/burlerengine/doc.go` describes the review gate and the verb; `pattern/PATTERN-gate-self-check-parity.md` gains the pair line: every burler round's `review` gate and `lyx burler validate-review`: `burlerengine.CheckReviewFile`, over `ParseReview`; `docs/overview.md`'s burler entry names the verb.

Tests in `cli_test.go`, through `RunCLIIn` from a temp directory outside any git repository (the verb must not need one):
one table-driven parity test runs the verb and `burlerengine.ReviewGate` over the same table of fixtures (valid, invalid YAML, malformed quoted summary, missing delimiter, missing file).
Each row asserts the verb's exit code and envelope (exit 0 with an `ok` envelope on the valid file, exit 1 with `CheckReviewFile`'s error on every other row: `ParseReview`'s error, quoting hint included on the malformed quoted summary, or the wrapped read failure on the missing file), and that the gate's passed/failed agrees with the verb's ok/error, with the gate's findings containing the verb's error text.
The verb's ok and error behavior has no test beside this table.

**ImpactSummary:** Adds one read-only burler verb and its pre-run skip; the run verb's wiring and every other module are untouched.
