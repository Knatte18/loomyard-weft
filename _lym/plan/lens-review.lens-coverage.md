```yaml
lens: coverage
outcome: reported
```

Coverage lens: does each decision, criterion and branch have a carrying card and a test row that would fail?

**BLOCKING**
- D8 / card 11 step 8. Every entry of the row records `GatedHead` as `TaskHead()` after the push, and this includes re-entries. Suppose the agent commits and the run crashes or resumes before the gate runs. The re-entry then pushes that ungated head and records it as gated, so a merge or approval on that head ends Done (card 13) and lands it. That breaks D8 ("nothing lym lands rests on an ungated head") and the D7/D9 ungated-merge halt. No row covers a re-entry whose task head is past the recorded `GatedHead`. Evidence: `_lym/plan/11-pr-review-row-opens.md:74`, `:78`.

**MEDIUM**
- D9 pause departure (Shared Decisions, card 13). The plan replaces the transient mark with `Yield` and asserts that "the next call resumes the wait". `shedengine` handles `Yield` by ending the loop with `RunYielded`, with nothing written (`internal/shedengine/run.go:471-474`, `:526-527`). No `internal/` package outside shedengine reads `RunYielded`, so nothing shown re-steps the run after the pause lifts. The plan's only row is "a covering pause yields". No row shows the run resuming, so D9's criterion "a covering pause ... ends the call transient" and its effect go unverified.
- D5 remainder / D9 poll (card 13). The `ErrTokenUnresolvable` halt (`gh auth login`) and the "any other failure is Stuck with Halt" branch of the `Status` poll have no row in `TestPRReview_Wait`. Its rows cover only a transient poll error. Evidence: `13-pr-review-row-waits.md:42`, `:53`.
- D11 (card 15). These Finalize branches have no row in `TestFinalize_PullRequest`:
  - an unreadable parent `beam.yaml` halt;
  - a `Status` read failure;
  - a transient merge failure, which is an error;
  - a `PushTask` failure in `mergeOnGitHub`.
  
  Two branches are not specified at all: failure of `ReadParentTips(true)` and of `FastForwardPair` after the GitHub merge. `PATTERN-refusal-way-forward` requires every halt to name its way forward and have a test that reaches it. Evidence: `15-finalize-through-pull-request.md:31`, `:39-41`, `:43`.
- D3 (card 11 step 4). On a no, only a recorded PR whose `Status` reads open is closed and its record removed. The plan does not say what happens to a record whose PR is merged or closed, or when `Status` fails. No row reaches any of those. Evidence: `11-pr-review-row-opens.md:70`, `:78`.
- D7 (card 14). A failed session start has two branches, transient error and Stuck with Halt naming `lym loom resume`; neither has a row. The plan leaves `SessionLive` and `RegisterSession` error handling unspecified and untested. Evidence: `14-pr-review-session.md:44`, `:49`.
- D3 criterion "reconcile prunes it" (card 12). The planned rows only assert that a file holding `pr_review` loads. No row asserts that `lym config reconcile` prunes `loom.pr_review` or `darn.pr_review`. Evidence: `12-retire-pr-review-keys.md:21`.
- `PATTERN-github-auth` change (card 16). The new clause, "only PR-Review, Finalize and pair teardown open, merge or close a pull request", is marked `(test)`. No card adds a test enforcing it, and the existing `cmd/lym/ghguard_test.go` only bans `gh` shell-outs. Evidence: `16-teardown-closes-pull-request.md:38`.

**LOW**
- D8 (card 13). The "tree dirty after the gate discards the verdict" branch has no row; the planned row covers only a head that moved during the gate. Also unreached: a head that moves between a counted pass and the push, `MailAgent` failing, the no-strand-recorded log branch, and errors from `PauseCovered` and `CommitsMissing`. Evidence: `13-pr-review-row-waits.md:41`, `:53`.
- D13 (card 11). No row asserts that the pawl, loom and darn `PR-Review` rows carry no `entry_gate`. `wantPawlRows` and `TestWire_BeamRecipe` only add the row. Evidence: `11-pr-review-row-opens.md:83`.
- D2 for a beam. `EntryField` reads `seedSlug(l.WorktreeName)`, and for a pawl run that is the beam's project entry. `ParentMode` reads the beam's parent. Neither has a pawl or beam row; `TestLandingDeps_PullRequestModeReadsTheParent` covers only a task under the prime. Evidence: `11-pr-review-row-opens.md:85-86`.
- D7 / D13 stencil. Whether `run_material` lists only material that exists on disk is not reached by a row: card 14's row asserts paths only. Evidence: `14-pr-review-session.md:41`, `:49`.
- Card 9. The "unreadable or unwritable record" error envelope has no row. Evidence: `09-pr-continue-verb.md:28`, `:30`.
- Card 16. The "unreadable record" and "origin URL read failure" warns have no row. Evidence: `16-teardown-closes-pull-request.md:30`, `:33`.
- Card 13 continue flag. The order between `Continue` and a merged or approved poll in the same wake is unspecified, and no row covers a continue flag set while the PR reads merged. Evidence: `13-pr-review-row-waits.md:50`.

Tool calls:
- Grepped `shedengine` for `Yield` and the session-refused verb wording: I needed to know whether a yielded call is re-stepped, which the context did not show.
- Read `run.go:465-476`, grepped for `RunYielded` consumers, and grepped the `clihelp` audience constants: I needed what a yielded run does next and whether `AudienceRole` exists for card 9.
