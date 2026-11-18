```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each diff change to a Goal point of the decision record and checked that every Goal point is built.

**Decisions.** These are the Goal points of the board entry. The record has no `Added after Discussion` entries.
- G1: one result type in place of the four-value return.
- G2: near-identical helpers merged. The entry names five: stop-result wrappers, fixer report parsers, table builders, the PR helpers repeated across Finalize and PR-Review, and repeated error wraps.
- G3: single-caller or test-only seams and adapters removed, `parentPuller` included.
- G4: the lock-freed, requeue and rebase layer designed away, or the report says why it stays.
- G5: the spec's rules and invariants hold, with no behaviour change, and tests change only where a signature changes.
- G6: the report gives code lines per spot, the estimate, and the test lines changed.

**Per change**
- C1, the `ending` type with `fail`, `stuck`, `halt`, `gateEnding` and `done`, plus every step and wrapper converted (`stuck.go`, `finalize*.go`, `parentalign.go`, `prreview.go`): G1, and G2's stop-result wrappers.
- C2, Finalize's merge, gate and land steps run once in a single retry loop (`finalize.go` `landUnderLock`): G2. The study names "one retry loop" (report.md:78).
- C3, `parentHoldsWork` read once per pass and passed to `ownerChecks`, `landCheck` and `syncParent`: G2. The study names "one read of 'parent holds the work' per pass" (report.md:78).
- C4, shared PR helpers `githubFailure`, `pushTask`, `pullRequestStatus`, `closeIfOpen`, `findOrOpenPullRequest`, `commitPRRecord`, `removePRRecord` and `landingSummary` (`finalize_pr.go`, `prreview.go`): G2, PR helpers and repeated error wraps.
- C5, `newResolver`, `newLandingGate`, `readMergeBefore` and `readRowAttempts` shared by Finalize and Parent-Align: G2. The study names "shared resolver construction", "one failure-record path" and "shared … fixer judgement" (report.md:79,81).
- C6, `parseVerdictReport` and `bullets` (`fixerreport.go`, `fixreview.go`): G2, fixer report parsers.
- C7, `soloSession.table`, `resolveModelChoice` and `requireValues` (`fixer.go`, `describe.go`, `fixreview.go`, `prsession.go`): G2, table builders.
- C8, `startFailure` merging the fixer and fix-review not-started checks (`fixerrun.go`, `fixreview.go`): none of the five named G2 categories. This is a finding.
- C9, `verifyGate.gateCall` and `runMarked` merging the command and base read and the marked verify of `check`, `checkHead` and `rerunTestFailure` (`verifygate.go`): none of the five named G2 categories, and not the study's "one failure-record path" or "one capped-list helper". This is a finding.
- C10, `parentPuller` removed (`parentalign.go`): G3.
- C11, `checkWiring` rewritten to use reflection, with `optionalPullRequestSeams`: G3. The study names "a 52-line wiring check" (report.md:110).
- C12, Finalize's `handedOver` field, `parentMerge`/`mergeIntoTask`, `noteFlakyPass` and `creditBlock` removed or inlined: G3.
- C13, the `PullRequestDeps` doc now names four optional fields (`deps.go:399`): forced by C11.
- C14, `prreview_wait_test.go:933` reads `.err`: forced by C1, a signature-only test change as G5 allows.
- C15, the `internal/shuttleparity/seats.go` Holders entries now name `soloSession.table`: forced by C7. The launch-spec literal scan (`producers_test.go:706-711`) needs the holder of each literal listed.
- C16, file headers and doc comments rewritten in `fixer.go`, `fixerreport.go`, `stuck.go` and others, and commit `811f9e31`'s comment re-breaks: forced by the moves in C1–C12.
- C17, `_lym/landing/summary.md` with its per-spot table, the estimate comparison, test lines and the section on the lock layer: G6 and G4.

**Per decision**
- G1: C1.
- G2: C1–C7, plus C8 and C9 outside its named categories.
- G3: C10–C12.
- G4: no code change. C17 explains why the layer stays, which is the record's "otherwise the report says why it stays" branch, so it is built.
- G5: no change of its own. The behaviour notes in C17 disclose departures; whether they hold is outside this lens.
- G6: C17.

**Findings**

1. LOW, class `scope`. `internal/landingshed/fixerrun.go:205-214` (`startFailure`) and its call sites at `fixerrun.go:149` and `fixreview.go:226`. Summary: merging the two not-started checks is outside the helper categories the Scope lists (`decision-record.md`, Goal "Scope", second bullet), and the study's spot list does not name it either (`report.md:78-81,108-110`). Evidence: diff piece 004 lines 116-146 and 287-298. Fix: revert to the two inline checks. If the parent wants this merge, it records an `Added after Discussion` entry instead.
2. LOW, class `scope`. `internal/landingshed/verifygate.go:228-259` (`gateCall`, `runMarked`) and its callers `check`, `checkHead` and `rerunTestFailure`. Summary: merging the landing gate's and the pr-review gate's command read, base read and marked verify is outside the categories the Scope lists. The study's gate items are "one failure-record path" and "one capped-list helper" (`report.md:79`), and this merge is neither. Evidence: diff piece 006 lines 191-310. Fix: revert, or get the parent to record the extension.

Tool calls:
1. Grepped `docs/research/rewrite/shrink-estimate-2026-11-18/report.md` for landingshed spot names. The Scope says "the spots the study names", and the study was not in my context.
2. Read `report.md` lines 64-115, the step 3 parts and categories tables, for the same reason.
