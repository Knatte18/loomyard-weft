# Card 17 — burler-review-parse-gate

**Create:**
- `plan:internal/burlerengine#CheckReviewFile` -> `func CheckReviewFile(path string) error`
- `plan:internal/burlerengine#ReviewGate` -> `func ReviewGate(reviewPath string) shuttleengine.Gate`

**Edit:**
- `internal/burlerengine#Engine.Run`
- `internal/burlerengine/engine_test.go`

**Intent:** D12, gate half: a Burler round whose review file has invalid frontmatter is re-prompted to the reviewer in its own session with the parse error and its quoting hint, instead of failing the run (#386).
The error text already states the fix, and the session that wrote the file is the one that can repair it.

In `internal/burlerengine/verdict.go`, beside `ParseReview`:

- `CheckReviewFile(path)` reads the file and returns `ParseReview`'s error, or nil when it parses; a read failure is returned wrapped with the path.
- `ReviewGate(reviewPath)` returns a `shuttleengine.Gate` closure that calls `CheckReviewFile` and answers passed on nil, or failed with findings carrying the error text verbatim, which already includes `quotedSummaryHint`'s hint, and a closing line naming the review file to rewrite; a missing file is a failed result with the same shape, never a gate error.

In `Engine.Run`, append one entry to every round's gate spec, after the wrapped `opts.Gate` entries, so the Webster segment's otherwise ungated round gets it too: `{Name: "review", Gate: ReviewGate(p.ReviewPath), Attempts: reviewGateAttempts, PassOnCap: true}`.
It is not wrapped in `repairReportBeforeGate`, since its own findings already name the file.
`reviewGateAttempts` is an engine-owned constant, 3, the re-prompt budget the recipe's artifact gates use; no config key is added (A4).
`PassOnCap` is what keeps the strict post-gate parse in place: once the budget is spent the entry lets the run through, and the existing `ParseReview` call after the gate fails the round with today's error, so a file still invalid after the budget fails as today.
Update `Engine.Run`'s doc comment for the new entry and its bound.

Bound: the entry re-prompts only on a parse failure of the round's own review file, never touches another file, and adds at most `reviewGateAttempts` re-prompts per round; `ParseReview` stays the strict reader, per `PATTERN-sole-parsers`.
The gate entry, the next card's verb and the post-gate parse all reach `ParseReview`, per `PATTERN-gate-self-check-parity`.

Tests in `engine_test.go`, through `Engine.Run` with the fake shuttle extended to evaluate the gate spec it receives like the shuttle wait loop (run each entry's closure in order, rewrite the review file between attempts as a reviewer would, honour `Attempts` and `PassOnCap`):

- a round whose first review fails `ParseReview` sees the `review` entry fail with the parse error and the quoting hint in its findings, the fake rewrites a valid file, the entry passes, and Run returns the verdict;
- a file still invalid after the budget lets the entry through and Run returns today's `round reached done but its review file is invalid` error;
- `TestEngine_Run_ZeroGateSpec` now asserts the spec holds exactly the `review` entry, and `TestEngine_Run_SpecConstruction` (or the gate-spec tests beside it) asserts it sits last after the told entries, unwrapped.

**ImpactSummary:** Every burler round's gate spec gains a trailing review-parse entry; producers calling Engine.Run need no change, and a malformed review now costs re-prompts before it fails.
