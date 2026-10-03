# Webster round 1 — fixer report

Every finding in `round-1-review.md` is fixed, one commit per finding (F7 has two), none pushed.
`go build ./...`, `go vet ./...` (untagged, `integration`, `smoke`) and `go test ./...` pass after the last fix.

## Fixed

- **F1** (`ed620249c`): `scankit.AssertImportAllowlistNoStale` fails for every allowed import path no production file imports; `importViolations` now returns an `importReport` carrying the unused paths, covered in `scankit_test.go`.
  The seam import guards of cards 8 and 9 call it; card 7's leaf guards keep `AssertImportAllowlist` untightened, as card 7 requires.
  No stale entry surfaced.
- **F2** (`7236b9b2f`): the CLI walk's bogus check requires `unknown subcommand` in the envelope.
  `config bogus` then failed (it refuses with `not a git repository`) and joined `cliTreeFindings` as `config#bogus`.
- **F3** (`342e637b7`): `bareProblems`/`bogusProblems` collect each invocation's problems, and `checkInvocation` skips a known finding with its problems listed and fails it once it has none.
  `envelope.Parse` is `Decode` without a test.
  The `board notes#bare` reason now matches the binary's behaviour.
  Proved by a planted `board#bare` entry, which failed with "now meets the invariant".
- **F4** (`1b2d87fff`): a claim's `anyCase` flag folds case; `droppedConceptClaims` sets it for `oversized` and `chain` and leaves `## Scope` case-sensitive, as the moved test did.
  Proved by a planted `Chain of batches.` in `webster-prefix-recovery.md`.
- **F5** (`0ca520e96`): the tmux isolation scan checks every (tag set, platform) pair and folds `GOOS`/`GOARCH` file-name suffixes into a file's constraint (`fileNamePlatform`, go/build's rule).
  New fixture cases and `TestFileNamePlatform`; CONSTRAINTS.md's Tmux Test Isolation Invariant states the per-platform rule.
- **F6** (`0e32adeef`): `declaresTestMain` resolves the name bound to the kit's import path and grants the unqualified `Main` call only to files in `internal/testkit/tmuxkit`; fixture cases cover an aliased import, another package imported as `tmuxkit`, and an unqualified call inside and outside the kit.
- **F7** (`7e5678496`): every comment block the branch added or rewrote that wrapped mid-sentence now runs one sentence per line, breaking only at clause boundaries; untouched lines of partly changed legacy comments are left alone.
- **F7, found during the fix** (`4167b6abf`): card 29 inserted each file's `StatusRel` helper between an existing doc comment and the function it documents, so the comments of `TestNewCommitStatusSeam_RepeatedPairCommitsOnce` (battencli), `TestNewCommitStatusSeam_OrdinaryPath`, `registerBootstrapTeardown` and `writeStatusFile` (loomcli) had attached to the helpers.
  This defect was not in the review; I found it while rewrapping F7's comments and fixed it in its own commit.
- **F8**: see below.
- **F9** (`cb0cee7ed`): the two `shedverbs` step-envelope test doc comments are gone.
- **F10** (`270cde1c6`): `TestSectionOf` covers the trailing-clause heading match and that `## One` does not take `## Onerous`.

## F8 — leaf test counts

PLACEHOLDER

## Deferred

Nothing.
