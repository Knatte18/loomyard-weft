## Goal

```markdown
Imported from [issue #123](https://github.com/Knatte18/loomyard/issues/123).

## Seen

`deadcode ./cmd/lym` (golang.org/x/tools/cmd/deadcode) on `main` at `35f5f8670` reports 156 functions in production code that the `lym` binary never reaches from `main`.
54 of them are `RunCLI` or `RunCLIIn` in the `*cli` packages.
Each is a thin wrapper, `clihelp.Execute(Command(), out, args)` or `clihelp.ExecuteIn(...)`, that only tests call, often from other packages' tests.
Others, such as `batcher.ProfileWeights` and `batcher.PeakContext`, are logic only tests call.
`cmd/lym/main.go` has `run`, which repeats the body of `main` and is called only by tests.

## What to do (operator, 2026-11-15)

1. **`RunCLI` and `RunCLIIn`:** replace every call with a direct `clihelp.Execute(<pkg>.Command(), ...)` or `clihelp.ExecuteIn(...)`, then delete the wrappers.
   The tests then run the same code they ran before.
2. **Functions with no production caller:** delete each, together with the tests that test only it.
   Keep one, and move it into a `_test.go` file, when it is a reference or oracle that tests of live code compare against; say which in the change description.
3. **`cmd/lym`'s `run`:** fold it and `main` into one function both call.
4. **The rule:** add a PATTERN entry: a function only tests call lives in a `_test.go` file or in `internal/testkit`, never in a production file.
   Enforce it with a pattern test; a `deadcode` run that fails on a new such function is one way, and the Discussion of the run settles the way.

## Safety conditions (operator, 2026-11-15)

A function is deleted only when every one of these holds:

- `deadcode` roots every `main` package of the module, `cmd/...` and `tools/...` both, not `cmd/lym` alone.
- It is unreachable under `GOOS=linux`, `GOOS=windows` and `GOOS=darwin` alike, since build-tagged files such as `internal/fslink`'s Windows junctions call code a Linux analysis cannot see.
- No reflection, `go:linkname`, template function map or lookup by name reaches it; grep its name before deleting.
- The gate is green up to tier 2 after the change.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-15): Test-support packages and the allowlist (code review F2)

Decision: internal/gitkit and internal/hubforge count as test-support packages beside internal/testkit, since PATTERN-test-isolation's import rules bar what they do from a kit. A function only other packages' tests call may stay in its production file under an entry of the guard's allowlist, naming the tests it serves, when a _test.go file cannot serve those tests and a kit cannot hold it because it reads its package's own state or writes a record its package alone writes. The entry's rule of '_test.go or internal/testkit' is read with these two widenings; the code stays as it is.

Rationale: A _test.go file cannot be imported by another package's tests, so a cross-package oracle or record writer has no home under the entry's literal rule; the board entry already keeps such oracles. The guard names each exception with its reason and fails on a stale entry, which keeps the exceptions visible and bounded.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-2-review.md; fixer report: _lym/reviews/code/round-2-fixer-report.md.
  - allowlist-keeps-pure-shedcheck-in-production (class design, severity BLOCKING, open in rounds 1, 2)
  - summary-describes-round-1-tree (class consistency, severity MEDIUM, open in rounds 2)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
