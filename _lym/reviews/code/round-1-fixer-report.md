# Fixer-report, code review round 1

All eight findings are fixed. `lym gate test` over the touched packages passes at tier 0 and tier 1, including the new tests.

## Changes

- **F1** (`websterengine`, `webstercli`), fixed as proposed.
  - The webster verify gate (`verifygate.go`) tests `res.RunnerCrash` after the timeout check on the first run and on the rerun.
  - It returns a terminal failing result naming the crashed runs and the log, and says no fixer runs.
  - It builds no `VerifyGateReport` and records no pre-fix head.
  - `verifyGateStuckReason` reads the crash findings by a prefix constant and ends in reentry alone, since no fixer commit exists to reset to.
  - `lym webster verify` (`webstercli/verify.go`) returns an error envelope naming the detail and log, with the way forward of running the verb again.
  - Rows: `TestVerifyGate_RunnerCrashFailsTerminalWithoutFixer` (first run, rerun, no report, no pre-fix head, stuck reason) and `TestVerifyVerb_RunnerCrashIsNoFailureToFix`.
  - The engine `doc.go` and the verb's help text name the behaviour.
- **F2** (`gatecli` tests), fixed as proposed.
  - `TestGateTest_Scenario` has steps for a crash alone (line printed) and for a crash beside a failing tier, a lint finding and a build-blocked tier (no line).
  - `TestRunWithFixer/RunnerCrashLineSpawnsNone` covers the card fixer's skip.
  - `TestExitCodeOf` covers a child ended by SIGSEGV, SIGABRT and SIGKILL, an exit, and a command that never started.
- **F3** (`verifytree`), fixed as proposed: `TestVerify_ReadsTheRunnerCrashAndTimeoutLines` covers a crash line, a timeout line beside a crash line (timeout wins) and neither.
- **F4** (`landingshed`), fixed as proposed: a Parent-Align row halts with no fixer session and no landing failure record, and a `TestFixerRun_Outcomes` row ends `fixerEndCrash` after one turn with the gate's own reason.
- **F5** (`loomshed`), fixed as proposed: `TestVerifyGate_TerminalOnlyForARunnerCrash` requires `Terminal` for a crash line and not for a plain failure.
- **F6** (`gatecli/tierrun.go`), fixed as proposed: `retryTier` sets the crash label only when the first run was a crash. Row `ShortageThenCrashCarriesNoCrash` in `TestRetryTier`.
- **F7** (`gatecli`), fixed differently: the review offered either option, and the fixer first took the doc option.
  - The self-check showed the doc option changes a build's exit code from 1 to 139 or 134, and `classifyHang` reads an exit code of 128 or above as a hang, so the webster gate would no longer rerun a build a signal killed.
  - So `exitCodeOf` keeps its old meaning and a new `tierExitCodeOf`, which only the tier command calls, reads a crash signal as 128 plus the signal.
  - The doc paragraph is unchanged from the original.
- **F8** (`landingshed`), fixed as proposed (the halt branch of the finding's `fix` key).
  - `checkHead` returns a `crashed` flag, and `gateHead` halts the row with the crash reason, which opens with `LandingStopPrefix`, instead of mailing the agent.
  - The reason names `lym loom resume` as the way forward; a resume gates the head again, since a crash sets no `MailedHead`.
  - Row: `ARunnerCrashHaltsTheRowNamingTheCrashAndMailsNothing`.
  - The package `doc.go` names the pr-review halt.

Commits: one per finding, `<module>: fix F<n> — …`, plus a follow-up for F7, the docs and the F1 verb test after the self-check.

## Disputed

None.

## Deferred

None.

## Self-check

Lens reports are saved beside this file as `round-1-fixer-report.lens-<lens>.md`. All four forks returned a report.

- fix-consistency, MEDIUM: `landingshed/doc.go` still said a failing `pr-review` gate always mails the agent, and its runner-crash paragraph left out the pr-review gate. Fixed as proposed (both paragraphs edited).
- fix-consistency, LOW: `websterengine/doc.go` did not describe the terminal runner crash. Fixed as proposed (paragraph added).
- fix-proposal, LOW: F7's doc option turns a build killed by a crash signal into exit 139 or 134, which the webster gate reads as a hang and does not rerun. Fixed differently: `tierExitCodeOf` for the tier command only; the build keeps exit 1.
- fix-proposal, LOW: the verb's crash test waited out the 10 s production rerun delay. Fixed differently: the tier prints the crash line itself and exits 1, which is no crash signal, so no rerun happens. The reading of a real crash is covered by the `gatecli` scenario steps and the `verifytree` test.
- fix-proposal, LOW: after F6 a shortage followed by one crash is an ordinary failure, so callers may hand it to a fixer. This is the behaviour F6 asks for, since one crash beside a shortage is not a repeat crash, and no code claims otherwise. No change.
- fix-code-correctness: no findings.
- fix-affected-set: no findings.
