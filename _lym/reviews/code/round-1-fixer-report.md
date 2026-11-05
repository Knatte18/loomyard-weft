## Fixer report, round 1

All six findings are addressed and the package gate is green (`lym gate test ./internal/driftengine ./internal/driftcli`, tier 0: 423 passed, 0 failed).
Commits, oldest first: `a1a744ee` (F1), `39455c0a` (F6), `fc3ef6b9` (F3, F4, F5 together, since `doc.go` carries hunks of all three), `c0ed73ff` (F1's CLI twin), `ab52042e` (F4 and F5 follow-ups from the self-check).

## Changes

- **F1, fixed as proposed.**
  `NewDriftGate` in `internal/driftengine/check.go` now narrows `clearable` to `Skip == "" && BaselineFault == ""`, so an unexpected cell or a failed current capture fails the gate beside a records or seal fault, and the failing findings carry that fault's text.
  Rows added to `TestCheck` in `check_test.go` for a records mismatch and for a seal fault, each beside an unexpected cell, with `gatePasses` false.
  The `NewDriftGate` doc comment and `doc.go` agree.
  Self-check follow-up: `checkResult` in `internal/driftcli/cli.go` held the same `clearable` rule for `lym drift check`'s `would_pass`; it is narrowed the same way, with a new tier 0 test `TestCheckResultWouldPass` in `internal/driftcli/checkresult_test.go`.
- **F2, no code change, as the fix key says.**
  The commits `e2dcd747` and `b7f922f6` cannot be rewritten inside the range.
  Their Pattern-Departure trailers name `PATTERN-fabric-vocabulary`, which is not an entry in `PATTERN.md` or `pattern/`.
  The departures (`CodeHead`, `RecordsChangedSince`, `LoomRecordsDirs` and `RecordsPathsChangedSince` in place of the card-named `WarpSHA` and Weft spellings) rest on `PATTERN-fabric-boundary`, whose rule at `pattern/PATTERN-fabric-boundary.md:24` the side-named identifiers would break.
- **F3, fixed as proposed.**
  `listLines` in `internal/driftengine/diff.go` treats a match whose `key` group is empty as no match, falling through to the next pattern or the unkeyed branch.
  Row `EmptyKeyGroupMakesTheLineUnkeyed` added to `TestDiffCaptures`; the `ListCaptures` doc comment and `doc.go` state the rule.
- **F4, fixed differently for Drift-Baseline and by an own variant for the rest.**
  - Drift-Expect: a records mismatch is now a Stuck for the parent (`haltForParent(recordsMismatchReason)`, the reason constant Drift-Verify already holds) instead of an error.
    The halt sits where a write would follow: after the sealed, scope-off and deferred cases, and inside `doneUnsealed` only when it must write `Unsealed`, so a run that needs no write is not stopped.
    Rows added to `TestDriftExpect`: a mismatch Stuck, a mismatch Stuck under scope off, and Done for an already unsealed run over edited records.
  - `Amend` admits Drift-Expect, for `--restore` alone (`admitAmend` takes the request), because `--discard` before the seal would make the check ignore the expect agent's file.
    `TestAmend` covers `--restore` at Drift-Expect (the restore test runs at Drift-Verify and at Drift-Expect) and the refusal of `--discard` there.
  - Scope off: `Amend` admits `--restore` under scope off, since the Drift-Expect halt under scope off would otherwise name a verb `Amend` refuses; a `TestAmend` test covers it.
  - Drift-Baseline: no change.
    `recordScope` in `baseline.go` writes only when no scope is recorded, and every records commit holds `scope.json`, since `WriteRecords` always writes both files, so a mismatch there always has a committed scope and no write is refused.
    This differs from the review's fix key, which names Drift-Baseline: the premise (a refused write naming `--restore`) does not hold there.
- **F5, fixed as proposed, with one narrowing.**
  `runAgent` closes the expect session mark as `SessionMark{OpenedAt: openedAt}`, and `pass` seals an existing valid file without a spawn only for a closed mark with `OpenedAt` set.
  Narrowing from the self-check: the mark closes as ended only when the agent's call returned no error; after an error it closes to the zero mark, so a spawn that never started the agent does not count as an ended session.
  Rows and tests: `AClosedMarkSealsOrSpawnsByTheFileAndTheSession` (a valid file with no session ever opened spawns), `AnErroredSpawnIsNotAnEndedSession`, and the `AgentWritesAndTheFileIsSealed` assertion that the mark keeps `OpenedAt`.
  The `SessionMark` and `OpenedAt` comments, the `Call` doc and `doc.go` state the rule for the expect mark only; the judge mark still closes to the zero value.
- **F6, fixed as proposed.** `TestDeferWaive` renamed `TestDefer`, with its comment.

## Self-check

Four forks returned; their reports are saved as `round-1-fixer-report.lens-<lens>.md`.

- fix-affected-set, MEDIUM: `lym drift check` kept the old `clearable` rule — fixed as proposed (`cli.go`, with a new test).
- fix-affected-set, LOW: `recordsMismatchHalt` duplicated `recordsMismatchReason` — fixed as proposed (the duplicate is removed).
- fix-affected-set, LOW: `await.go:99` (Drift-Await) can still hit the plain `WriteRecords` refusal, and `Amend` refuses at that row — not fixed; see Deferred.
- fix-proposal, MEDIUM: the mismatch halt fired where no write follows, and under scope off led to a refused way forward — fixed with an own change (halt only where a write follows; `Amend` admits `--restore` under scope off).
- fix-proposal, LOW: admitting Drift-Expect widened `Amend` to `--discard` and `--entries` before the seal — fixed with an own change (`--restore` alone at Drift-Expect).
- fix-proposal, LOW: a closed mark after an errored spawn counted as an ended session — fixed with an own change (the ended mark needs a call without error).
- fix-code-correctness, LOW: same as the fix-proposal MEDIUM above — merged, fixed.
- fix-consistency, LOW: the `lym drift amend` help text omitted Drift-Expect — fixed as proposed (`internal/driftcli/amend.go`).
- fix-consistency, LOW: the `SessionMark` doc said a zero mark means never opened, but the judge mark closes to zero — fixed as proposed (the comment is narrowed to the expect mark).

## Disputed

None.

## Deferred

- **Drift-Await and a records mismatch (fix-affected-set LOW).**
  `internal/driftengine/await.go:99` writes the awaited_at record through `WriteRecords`, which refuses over a records mismatch with an error naming `lym drift amend <slug> --restore`, and `Amend` is not admitted at Drift-Await.
  Left open: the review scoped F4 to Drift-Expect and Drift-Baseline, and a fix means a new halt in Drift-Await plus a new row in `amendRows` and its refusal tests (`ARowBeforeDriftExpect` pins Drift-Await as refused), a behaviour change of its own.
  The reason is scope, not an inability: it needs the parent's or operator's decision whether Drift-Await joins the halting rows.
