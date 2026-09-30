---
verdict: APPROVED
rationale: "All eight round-1 findings (F1-F8) are fixed in the current cards 1-6, and every Decision, Constraint and acceptance criterion in the decision record maps to a card and a test; the plan introduces no unlicensed scope"
---

The round-1 report raised one BLOCKING and seven lesser findings against the pre-fix plan.
I judged the plan as it stands after the fixer pass and checked each fix against the cards and the code they name.

- **F1:** card 2 now runs `git check-ignore` through the checked `gitexec.Run` and tells exit 1 apart via `*gitexec.GitError.ExitCode`.
  `internal/gitexec/gitexec.go:125-131` confirms `Run` returns that error type with the exit code.
  `cmd/lyx/checkedcall_test.go:87` pins `fabricengine` at 2 raw sites, and the plan adds none.
- **F2:** card 6 now targets `records_integration_test.go`, which asserts the attach-only `tasks.json` under `childLocation.AnchorPath()` and a clean `git status --porcelain`.
  `childLocation` already exists in that test (line 38).
- **F3:** `PathTracked` is its own card 3, in its own file with its own integration test file.
- **F4:** card 2 adds a lock-held test on `gitExcludeLockFileName`, which exists at `internal/fabricengine/gitexclude.go:25`.
- **F5:** card 4's helper now returns the prime's name and a resolved flag, and logs the warning once.
- **F6:** card 1 creates `TaskChainInteractive`, and card 4 lists it in `Uses`.
- **F7:** card 4 checks tracked status before writing the exclude entry, and its tracked-case test asserts `info/exclude` is byte-for-byte unchanged.
- **F8:** `## verify:` now runs `./cmd/lyx/...`.

The report's one focus departure was raising the exclude-before-tracked-check order as F7, although the round-1 focus had set it aside as an accepted risk.
I ratify that departure.
The decision record's accepted risk is an exclude line that hides a `.vscode/` the driven path created.
In the tracked case the path creates nothing, so the line serves no purpose, and swapping the steps costs nothing.

Beyond the fixes, fidelity holds.
Cards 5 and 4 carry each Run-Shed and driven-path acceptance criterion as a named test.
Card 6's integration `TestMain` stub, together with card 5's `OpenIDE` stubs, keeps any real `code` process out of the tests.
No test outside `internal/battencli` reaches batten's wiring.
Verify runs every targeted package both untagged and under `-tags integration`, plus `battenrecipe` and `loomcli` for their guards.
No `manifest/designs/` doc covers batten or ide, so card 6's docs scope, `docs/overview.md` only, matches the decision record.
