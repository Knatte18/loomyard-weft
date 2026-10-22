# Fixer report — discussion round 2

## Changes

- F1 (MEDIUM), fixed as proposed: D4 states that the SIGTERM-first cancel, with its own SIGKILL timer and group-empty poll, is a new cancel used only by `verifyrun`, in `internal/proc`'s platform files, and that `proc.ConfigureGroupKill` keeps its immediate group SIGKILL for `gitexec` and the `lyxbin` test kit.
  Scope Out gains a bullet naming `ConfigureGroupKill`'s cancel, and with it `gitexec`'s, as unchanged.
- F2 (LOW), fixed as proposed: D1's `ErrPaused` carries the slug its read judged.
  D2's run-start and `DeliverPendingPrompt` bullets wait through `pauserecord.WaitUncovered` on that carried slug instead of the runner's `waitWhileCovered`.
  A new acceptance criterion covers a run start whose target slug differs from its runner's.
- F3 (LOW), fixed as proposed: D1 drops the orch runner's setter and hub record path, and states that the orch runner stays told no geometry, so its run-start lines read uncovered.
  The orch runner no longer counts toward the every-production-runner claim.
  D2's run-start bullet and the `PATTERN-pause-holds-prompts` text say the same.
- F4 (LOW), fixed as proposed: D8's failed-lookup refusal opens with "end your turn now", and its acceptance criterion asserts that opening.

## Self-check

- fix-consistency, MEDIUM (D4 places the SIGKILL timer in both `verifyrun` and `internal/proc`): fixed as proposed. The earlier sentence now reads "a timer of the new cancel's own".
- fix-consistency, LOW, merged with fix-proposal LOW 1 (the D3 gate hold and D2's gate bullet still judge the runner's slug, so a covered target re-runs the gate every tick): fixed with my own change.
  D3 now says `run.covered` judges the slug D1 judges for the run's own session, falling back as D1 does, which also makes D2's "once the run reads uncovered" judge that slug.
  The gate acceptance criterion gains the differing-slug case.
- fix-consistency, LOW (the "what tests use" sentence in D1 and the PATTERN text overstate which runners read uncovered): fixed as proposed. D1 names tests and the orch runner, and the PATTERN text scopes its claim to production runners that type a line.
- fix-claims, LOW (`internal/loomcli/looparm.go`'s runner is a second production runner told no geometry): fixed with my own change. The D1 claim now covers only "every production runner that types a line"; that runner only calls `StopStepRuns`.
- fix-proposal, LOW 2 (after "end your turn now" Master cannot act on the way forward unprompted): disputed, see below.
- fix-way-forward: no findings.

## Disputed

- fix-proposal LOW 2, ground `every fix worse`: dropping "end your turn now" brings back F4's defect, a Master working inside a running turn under the pause.
  The record already states and accepts the resulting stall in D8's Bound ("it moves on only when it is next prompted") and in Open risks ("no line wakes Master").

## Deferred

None.
