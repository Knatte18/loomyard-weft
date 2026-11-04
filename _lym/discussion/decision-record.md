## Goal

```markdown
## Seen

On the `cs` hub, task `transitional-report-note` (Drift trial 2, 2026-11-03) ran its Finalize gate at 20:36:40.
`dotnet test LymCsharp.sln --no-build --filter 'Category!=Slow'` died in 10.7 s with `Fatal error. Internal CLR error. (0x80131506)` and `Aborted (core dumped)`; the gate record `_bolt/.lym/gate/runs/20261103T193653Z-tier0-*.json` shows `outcome: failed`, `passed: 0`, `failed: 0`, and the verify command exited 134.
Finalize read that as a failing gate: it freed the parent lock and spawned an agent fixer (`landingshed: fixer run ended … end=fixed spent=1 cap=2`).
The fixer changed no code; the gate ran again at 20:37:24, passed (138 tests), and the run landed.

The first trial saw the same CLR error in the adopt check of `root` and called it a one-off of the .NET runtime; it has now hit a second run.

A runner that crashes with no test result says nothing about the code under test.
Treating it as a code failure spends an agent session (and one of the fixer's two attempts) on a green tree, and a second crash in the same run would spend the other and stop the landing.

## Expected

- A tier that ends with no test result and a signal exit (exit 134, 139, `Aborted`) is reported as a runner crash, apart from a failing test.
- The gate runs a crashed tier again once before it reports a failure, and records both runs.
- Only a tier that reported failing tests goes to a fixer; a crash that repeats stops the run with the crash named, not a fixer.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - gatecli-closing-line-exitcode-detach-untested (class design, severity MEDIUM, open in rounds 1)
  - landingshed-parentalign-fixerend-crash-untested (class design, severity MEDIUM, open in rounds 1)
  - loomshed-terminal-on-crash-untested (class design, severity MEDIUM, open in rounds 1)
  - verifytree-crash-line-reading-untested (class design, severity MEDIUM, open in rounds 1)
  - webster-verify-gate-ignores-runner-crash (class scope, severity BLOCKING, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
