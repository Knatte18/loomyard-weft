# Drift accept refuses unknown test names and keeps the output it accepts

Two interim Drift repairs ahead of the Drift rework that `docs/research/drift-whole-review-2026-11-04.md` proposes (faults 1 and 3).

## Accept checks its test name

`lym drift accept <test>` refused nothing about the name, and an accepted name joins the compare's skip list (`SkipList`), so a typo or a test that never drifted hid that test from every later compare.
`Accept` now refuses a test with no profile file under the run's baseline directory, naming the test and the directory.
A profile file belongs to a test when its path relative to the profile directory, with or without its extension, is the test's name, so `parser_roundtrip` matches the C# harness's `parser_roundtrip.txt` and a nested `group/case.txt` is named `group/case`.
`CheckAccept` runs the same checks without writing, so the verb refuses before it spends a capture.

## The accepted profile holds the output each acceptance saw

Drift-Verify used to capture the accepted profile at the end of its pass (`keepAcceptedProfile`), after the agent's fixes, while the agent's later compares skipped accepted tests.
An agent that accepted test X and then moved X again left a profile that certified the moved output.

Now `lym drift accept` captures the current code's output into `.lym/drift/accepting/` (`RunAcceptanceCapture`, one gate slot, outside the session's run budget) and `Accept` moves the test's profile files to `.lym/drift/accepted-tests/`, replacing any an earlier acceptance of that test kept.
`Withdraw` drops them.
At the end of a passing pass, Drift-Verify builds `.lym/drift/accepted/` as the baseline with each accepted test's files replaced by its kept output, compares HEAD against it with only the scope's skipped tests left out, and records `accepted_profile_head` only when that compare passes.
A failing compare means an accepted test's output changed after its acceptance: the row halts naming `lym drift accept` again, `lym drift withdraw` or a fix, then `lym loom resume`.
An acceptance with no kept output, such as one a binary from before this change recorded or one on a machine that adopted the run, halts naming a new `lym drift accept`.
Drift-Recheck is unchanged: it already compares against the accepted profile, which now holds the accepted outputs.

The `CaptureAccepted` seam and `RunAcceptedCapture` are gone; Drift-Verify now requires the `CompareAccepted` seam Drift-Recheck already used, and the shed registry, the wiring in `loomcli` and the test kits follow.
The verify agent's judging text tells it how to name a test and that an acceptance keeps the output as it is then.

## Design decisions

- `lym drift scope --skip` stays unchecked, by the parent's decision recorded in the decision record: it runs at Drift-Baseline before any capture, and a capture writes no file for a skipped test, so a check against the baseline directory would refuse every partial scope.
- The acceptance capture runs the whole suite rather than the one test, since a skip list built from file names need not match the harness's case names.

## Tests

`TestAcceptWithdraw` covers the name forms, the refusals naming the test and directory, the kept and replaced outputs, `CheckAccept` and the withdrawal's drop.
`TestDriftVerify` covers the built profile, the halt on a moved accepted output and on a missing kept output, and an agent's acceptance compared at its later commit against the output it accepted.
`TestRunCapture` covers `RunAcceptanceCapture`'s pass and failures and its missing-dir way forward.
