# Report Code Mode on a darn writer repeating one edit across packages

Adds the report of a bench of Claude Code's Code Mode on a darn writer doing a repeated edit, and the data it rests on.
The change is documents only: `docs/research/claude/code-mode-darn-bench-2026-11-17/report.md`, its `data/` folder and a line in `docs/research/README.md`.
No code changes.

The bench replayed the landed task `test-binary-fork-guard` (`888969e7`, base `112bb821`) with one control writer and one Code Mode writer, each once, on Opus at `medium`.
One Code Mode session checked first that lym's shell guard still refuses a guarded command inside a script.

## What the report finds

- The guard held: `sed -i` was refused through every route a script has, with the direct call's refusal text.
  A redirect into a non-source file ran, as the guard's rule allows.
- Code Mode cost $5.89 against the control's $7.17 with its lookup subagents, used 36% fewer cache-read tokens and 17 fewer turns, and took 24m12s against 18m46s.
- Every `REPL` eval settled; one inner `go build` failed on an unused import.
- Both clones compile and vet on the Go directories they changed.
  No test ran and no diff was reviewed, so the report draws no quality conclusion and does not support adopting Code Mode for the darn writer.

## Design decisions

- The input is `888969e7`, chosen by the parent over `bolt-dir-rename`, which is mostly docs.
  The decision record carries that choice, the clone-based method and the $25 cap.
- The writers ran in local clones outside the hub, as the 2026-11-12 advisor bench did, so no gate, cold fixer or test ran.
  The prompt is the darn template rewritten for a clone and points to `PATTERN.md` instead of inlining it; both arms got the same text.
- Code Mode was switched on through this worktree's `claude.yaml` `binary`, a wrapper that sets the variable; the file was deleted before the control ran and after the Code Mode arm.
- The control ran first and the Code Mode arm after it, so the arms did not share the account's rate.
- The report states one sample per arm and names the cause of the extra Code Mode wall time as not known.
- The bench's spend by list price is $13.25 against the $25 cap.

## To check by hand

- The change description each writer produced is in `data/` and was not validated against the final-summary spec.
