# Card 3 — burler-halves-producers

**Create:**
- `internal/shedadapters#BurlerHalfRunner`
- `internal/shedadapters#BurlerHalfDeps`
- `internal/shedadapters#burlerReviewEngineLabel`
- `internal/shedadapters#burlerFixEngineLabel`
- `internal/shedadapters#BurlerReviewProducer`
- `internal/shedadapters#NewBurlerReviewProducer`
- `internal/shedadapters#BurlerReviewProducer.Call`
- `internal/shedadapters#BurlerReviewProducer.roundProfile`
- `internal/shedadapters#BurlerReviewProducer.acceptedExit`
- `internal/shedadapters#BurlerReviewProducer.probeLiveReview`
- `internal/shedadapters#BurlerFixProducer`
- `internal/shedadapters#NewBurlerFixProducer`
- `internal/shedadapters#BurlerFixProducer.Call`
- `internal/shedadapters#BurlerFixProducer.fixedExit`
- `internal/shedadapters#BurlerFixProducer.probeLiveFix`
- `internal/shedadapters#roundAcceptancePath`
- `internal/shedadapters#acceptanceHeader`
- `internal/shedadapters#writeAcceptance`
- `internal/shedadapters#readAcceptance`
- `internal/testkit/shedfake#BurlerHalfRunner`
- `internal/testkit/shedfake#BurlerHalfRunner.Review`
- `internal/testkit/shedfake#BurlerHalfRunner.ProbeReview`
- `internal/testkit/shedfake#BurlerHalfRunner.ResumeReview`
- `internal/testkit/shedfake#BurlerHalfRunner.Fix`
- `internal/testkit/shedfake#BurlerHalfRunner.ProbeFix`
- `internal/testkit/shedfake#BurlerHalfRunner.ResumeFix`
- `internal/testkit/shedfake#TestBurlerHalfRunner_LastEntryRepeats`
- `internal/shedadapters/burlerreview_test.go`
- `internal/shedadapters/burlerfix_test.go`

**Edit:**
- `internal/shedadapters/doc.go`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shedadapters#highestCompleteRound`
- `internal/shedadapters#hydrationPaths`
- `internal/shedadapters#recordedVerdict`
- `internal/shedadapters#archiveStaleOutputs`
- `internal/shedadapters#mergeRoundUsage`
- `internal/shedadapters#newHalfUsage`
- `internal/shedadapters#readRoundFocus`
- `internal/shedadapters#excludedLenses`
- `internal/shedadapters#gateFailedReason`
- `internal/shedadapters#gateAttemptsPointer`
- `internal/shedadapters#describeHalf`
- `internal/burlerengine#ErrReviewChanged`

**Intent:**
D1, D3 and D4 at the Shed adapter: the two producers each half of a review segment's round runs as, added beside `BurlerProducer`, which card 4 stops wiring and card 10 deletes.
They live in new files `burlerreview.go`, `burlerfix.go` and `acceptance.go` of `internal/shedadapters`.

`BurlerHalfRunner` is the seam both producers drive: `Review`, `ProbeReview`, `ResumeReview`, `Fix`, `ProbeFix` and `ResumeFix`, with the signatures card 2 gave `*burlerengine.Engine`, which a compile-time assertion proves satisfies it.
`BurlerHalfDeps` carries the `Runner` and the per-round `Models`; no anchor path, since no ready marker exists.
Each constructor validates as `NewBurlerProducer` does (non-nil runner, non-empty name, absolute run directory), and `NewBurlerFixProducer` also refuses an empty `reviewRow`.

The acceptance record (`acceptance.go`): `round-<n>-acceptance.md` in the run directory, YAML frontmatter holding the round and the accepted review's digest, written only by the review row and read by both.
It is one of the round's paths: every archive of a whole round moves it with the review and the fixer report, so no record outlives its review.
A record that is absent or does not parse reads as absent; any other read error is returned.

Review row (`BurlerReviewProducer.Call`), never returning Done:
- entry check, create the run directory, resolve `highestCompleteRound`;
- a highest complete round with no recorded verdict hands on BudgetExempt with its review as `Path` and spawns nothing, so the fix row passes it through to the Bouncer for judgment;
- for round `highest + 1`, a review with an acceptance record hands on BudgetExempt with no spawn: a restart with the fixer pending resumes at the fix row without re-running the reviewer;
- a live reviewer is probed before anything is archived (`probeLiveReview`): both live is impossible, so a live half is resumed through `ResumeReview` and mapped like a spawn, a resumed half that died or timed out falls through to a fresh attempt 1, and a probe or resume error wrapping `burlerengine.ErrHalfNotStopped` returns without archiving;
- otherwise the round's review, fixer report and acceptance record are archived (an orphan review without a record is reviewed again) and `roundProfile` builds the round's profile exactly as `BurlerProducer.Call` does today: review and fixer-report paths, focus directive, hydration of earlier complete rounds, and the cluster exclusions from round 2 on;
- at most two attempts, the second only after a died or timed-out first, each with its round token (`<n>`, then `<n>b`) and `Models.Pick(round)`'s review choice;
- an accepted review (`acceptedExit`) writes the acceptance record with the result's digest, merges the review half's usage (fan, lenses, the review `halfUsage`) into the round's usage record, and returns Stuck with the review as `Path`, `BudgetExempt` set;
- a reviewer whose gate fails, whose strict parse fails, that ends two consecutive attempts died or timed out, or with an unrecognized outcome, is a hard error with the round's three paths archived, as today, since a Stuck with no review would be misread by the Bouncer as a seed call; a failed acceptance write is that same hard error;
- a reviewer that could not be stopped returns its error with nothing archived and no retry; cancellation behaves as `BurlerProducer`'s does.

Fix row (`BurlerFixProducer.Call`), never returning Done:
- a highest complete round with no recorded verdict hands on to the Bouncer BudgetExempt with no spawn (a fixer report already on disk, or the review row's pass-through);
- for round `highest + 1`, no acceptance record is a hard error: `shedadapters: <name> (burler-fix): round <n> has no accepted review; way forward: run "lyx loom goto --to <reviewRow>", then "lyx loom resume", in the task worktree`.
  Its inputs are a goto into the fix row with no accepted review for the next round, which the review row's hand-on never produces, and a resume after the fix row archived a changed review (below), since Shed resumes a failed run at the failing row; a test in `burlerfix_test.go` reaches it over a run directory holding no acceptance record, and `refusal-spec.md` gains its row in the loom section, quoting that way forward;
- a live fixer is probed first (`probeLiveFix`) and resumed through `ResumeFix` with the recorded digest;
- otherwise only a stale fixer report is archived, and at most two attempts run `Fix` with the recorded digest, `Models.Pick(round)`'s fix choice, the round token, the row's gate (`opts.Gate`) and the friction note id `burler-<run subdir>-r<n>`;
- a done result with a passing or absent gate, a skipped empty review included (`fixedExit`), merges the fix half's usage and returns Stuck with the review as `Path`, `BudgetExempt` set;
- a done result whose gate failed archives the review, fixer report and acceptance record and returns a counted Stuck with an empty `Path`, the gate's reason and attempts, so the Bouncer's next round reviews afresh; the row's MaxBounces caps these per episode;
- `ErrReviewChanged` archives the review, fixer report and acceptance record and returns a hard error ending with the same way forward as the missing-acceptance refusal, so the round is re-reviewed once the goto moves the run to the review row (Shared Decision `changed-review-recovery`);
- a fixer that ends two consecutive attempts died or timed out, an unrecognized outcome or any other runner error archives the fixer report only and returns a hard error, keeping the accepted review and its record so a resume re-spawns the fixer without re-running the reviewer;
- a fixer that could not be stopped returns its error with nothing archived and no retry.

`shedfake.BurlerHalfRunner` is the fake of the seam: scripted results and errors per method whose last entry repeats, unscripted probes answering a gone half, per-method call counts, the profiles, options and digests each call received, and `DuringReview` and `DuringFix` hooks that run inside the call with its index; `TestBurlerHalfRunner_LastEntryRepeats` pins the scripting.

`burlerreview_test.go` and `burlerfix_test.go` pin every branch above over the fake, carrying over the cases of `BurlerProducer`'s own tests that the rows keep (round scan over stamped siblings and malformed names, hydration of earlier complete rounds, focus and cluster exclusions, round tokens and note ids, the retry, cancellation and archive-on-exit), and including that neither producer ever returns Done, that the pass-through hand-backs are BudgetExempt and a gate-exhausted Stuck is not, the archive set of each exit, the usage record merge, and the restart cases of the acceptance criteria: an acceptance record without a fixer report resumes at the fix row with no review spawn, a review without one is re-reviewed, a live reviewer or fixer is resumed or re-spawned alone, and a fixer report on disk hands on with no spawn.
`internal/shedadapters/doc.go` describes the two rows, the acceptance record and their archive sets beside the existing `BurlerProducer` text, which card 10 removes.
The `internal/shedadapters` and `internal/testkit/shedfake` rows of `test-budget.yaml` rise to the packages' new counts.

**ImpactSummary:** additive: two new producers, a new seam and a new fake; nothing wires them until card 4, and `BurlerProducer` is untouched.

**Commit:** 3: shedadapters: a review row and a fix row for each Burler round
