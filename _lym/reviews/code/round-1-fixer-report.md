# Fixer-report, code review round 1

Every finding is fixed.
The gate is green: `lym gate test ./internal/driftengine ./internal/driftcli` at tier 0 and `lym gate test --tier 1 ./internal/driftcli`.

Commits, grouped by files because a hunk-level commit per finding is not possible without interactive staging:

- `fb9e3ded` F6
- `9a17e9ac` F2, F4, F5
- `9778e34a` F1, F3
- `998e8b89` the self-check's consistency findings

## Changes

- **F1, fixed as proposed.**
  `testFiles` matches a file when `rel == test`, or when the last path element has an extension that is not its whole name and `rel` minus that extension equals `test`.
  So `.gitkeep` and `group/.gitkeep` never yield an empty stem or a `group/` stem.
  `requireBaselineTest` refuses an empty `test` first, showing it as `""` in the same refusal.
  Rows added to the name table of `TestAcceptWithdraw`: the empty name against `.gitkeep`, `group/` against `group/.gitkeep`, `t1.txt` against a baseline holding only `t1.txt.bak`, and `t1` against the same baseline.
  The decision on the second shape: `t1.txt` is admitted for `t1.txt.bak`, since the documented rule names a file by its path with or without its last extension, and `t1` is refused.
  The module and verb docs now say "last extension" and that a dotfile has none.
- **F2, fixed as proposed.**
  `internal/driftcli/accept_integration_test.go` holds one table-driven tier 1 test over the built `lym drift accept`.
  Its rows: a passing capture keeps the output, records the acceptance and returns `log`; a capture exiting non-zero refuses and records nothing; a refused check at the wrong row runs no capture; a held accept lock refuses before any capture.
  Every row that ran the capture asserts it read the scope's skip list.
- **F3, fixed as proposed.**
  `keepAcceptedOutput` moves the new files first, then `dropAcceptedOutput(p, test, except)` removes only earlier kept files the new set did not replace.
  `Withdraw` writes the scope record first and drops the kept output after it.
  No test is added, since F3 is not a coverage gap and the existing replacement and withdrawal tests pin the observable result.
- **F4, fixed as proposed.**
  `Paths.AcceptLock` (`.lym/drift/accept.lock`) is declared.
  `acceptTest` takes it through `holdAcceptLock` (non-blocking, refusing with a way forward) before anything else and releases it when it returns.
  The lock row of the F2 test covers it.
- **F5, fixed as proposed.**
  `acceptTest` calls `w.verbStatus()` again after the capture and passes that status to `driftengine.Accept`.
- **F6, fixed as proposed.**
  The wording names the compare, not a capture, in `recheck.go` (doc comment and Stuck text), `doc.go`, and the helper comment and two subtest names in `recheck_test.go`.
  The test assertion on the Stuck text is in step with it.
  The self-check also found `docs/overview.md`, which is reworded the same way.

## Self-check

Four forks returned a report; none was lost.

- fix-code-correctness: no findings.
- fix-affected-set: no findings.
- fix-consistency, MEDIUM: `internal/driftcli/doc.go` and the `accept` help text did not describe the accept lock, the order and the second status read. Fixed as proposed.
- fix-consistency, LOW: `docs/overview.md:615` still said the profile was captured. Fixed as proposed.
- fix-consistency, LOW: `internal/driftengine/doc.go` did not say the verb, not the engine, holds the accept lock. Fixed as proposed.
- fix-consistency, LOW: the name-matching prose did not carry F1's dotfile and last-extension rule (`accepted.go` header, `internal/driftengine/doc.go`, the accept help text). Fixed as proposed.
- fix-proposal, LOW: a failed `os.Rename` after the first move in `keepAcceptedOutput` leaves new files beside old ones. Deferred, see below.
- fix-proposal, LOW: a failed scope write after `keepAcceptedOutput` leaves the new kept output under the old record and its `AcceptedProfileHead`. Deferred, see below.
- fix-proposal, LOW: after `Withdraw` records the withdrawal, a failed drop cannot be retried through the verb, since the test is no longer accepted. Deferred, see below.

## Disputed

None.

## Deferred

The three fix-proposal findings above are the residue of the reviewer's own F3 proposal (move first, drop after the scope write), which the fix carries out.
Closing them takes a transactional keep (stage the new set and swap it in, or record the acceptance and keep the output as two steps with a repair path), and that changes what `Accept` and `Withdraw` do when I/O fails after part of their work: an error alongside a recorded acceptance or withdrawal, or a rollback of kept files.
That is a tradeoff the operator decides, and only an I/O failure triggers any of the three; nothing reads a withdrawn test's kept output, and a re-accept replaces it.
