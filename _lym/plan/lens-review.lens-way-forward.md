```yaml
lens: way-forward
outcome: reported
```

Task: review the plan's refusals, halts and recovery steps through the way-forward lens.

1. BLOCKING. Covering pause (card 13 wait step 1; overview "A covering pause yields"). The `Yield` recovery depends on a "next call" that nothing is shown to make. The card says the shed loop's pause check holds the run and the next call resumes the wait. But a yielded step ends the loop and the driver pass with nothing decided, and the plan names nothing that re-drives a run left `running` at PR-Review after the pause lifts. No card or test covers the resume after a yield. Evidence: `internal/shedverbs/step.go:124`, `internal/sheddriver/doc.go:17`, `internal/sheddriver/drive.go:221`. `internal/shedengine/producer.go:22-26` says only batten's InnerRun returns Yield.

2. MEDIUM. Card 11 step 8 and card 13 poll: "any other failure is Stuck with Halt naming the call and the error, then `lym loom resume`". Plain `lym loom resume` cannot fix a persistent 4xx. Examples: a 401 from a rejected `GH_TOKEN`/`GITHUB_TOKEN` (authRT returns its own error, not `ErrTokenUnresolvable`), or a 403/404 when the gh user cannot reach the repository. In those states the resume fails the same way again. The halt needs to name the credential or access fix. Evidence: `internal/githubclient/doc.go` (env token 401 skips the replay and returns an error naming the env var); `internal/githubclient/token.go` (`ErrTokenUnresolvable` only when no source gives a token).

3. MEDIUM. Card 15, yes with a record whose pull request is closed and the record lacks `ClosedForContinue`. The halt's only way forward is `lym loom goto PR-Review`, which cannot progress alone. Card 11 step 6 reuses the recorded pull request and never opens a new one. Card 13's closed verdict then halts again with "reopen it on GitHub". The Finalize halt should name reopening on GitHub, or `lym batten cancel <slug>`, as card 13's closed halt does.

4. MEDIUM. Card 15, `RefusalOther` or a pending refusal past the minute. The halt names "approving the pull request again … then `lym loom resume`". Finalize has just pushed the parent merge and fixer commits, and branch protection may have dismissed the approval. If no second person is available, the operator cannot approve their own pull request (decision record Constraints: GitHub refuses the author's own approval). The halt does not name the operator's own ending through the agent, `lym loom goto PR-Review` then `lym loom pr-continue`, so for the operator alone this halt has no way forward that can succeed.

5. MEDIUM. Card 13, gate failure while no agent strand is live. The failure is "logged when no strand is recorded", and the plan does not say what happens when the recorded strand is dead (the operator closed the window, which card 14 allows). The wait then runs forever on a failing head with nothing surfaced. Card 17 also suppresses the quiet notice for the whole wait, so the operator gets no next step. No card says how to get the session back. Card 14 only says "until the next entry of the row", and a running row has no next entry without a pause or goto.

6. MEDIUM. Card 13, merged with a head other than `GatedHead`. The way forward ("run the gate over the parent branch in its worktree … then `lym loom goto Finalize`") runs the gate before anything brings the local parent level with the GitHub merge. Only Finalize's sync fast-forwards the parent pair, under the lock, so the gate tests the stale local parent, not what GitHub merged. The step cannot judge the landed head. It needs a fetch or fast-forward of the parent first, or the gate run after Finalize.

7. MEDIUM. Card 13, merged lacking task commits, way forward "file them as a board note, then `lym batten cancel <slug>`". The pull request's work is already on the parent, but cancel settles the entry as cleared or draft, never done (`internal/pairteardown/doc.go` "Board claim"). The entry whose work landed then looks unlanded. The way forward also needs `lym board set-status <slug> done`.

8. LOW. Card 13, continue ending. If the close succeeds but the `ClosedForContinue` record write fails, the next wake sees closed-and-not-for-continue. It halts with "reopen it on GitHub", even though the record still carries `Continue`. The closed verdict should accept `Continue` as well as `ClosedForContinue`.

9. LOW. Card 13 and card 11 re-entry. A record left with `Continue` and `ClosedForContinue` makes a later PR-Review entry end Done at once. Example: a loom Finalize reroute through Parent-Align, the perch and Describe back to PR-Review. The new reroute commits are then never shown in a review or gated by the row. No card resets the flags on re-entry.

10. LOW. Cards 11, 15 and 16. Several failure paths have no outcome. Card 11 step 4 and card 15's "no" branch: a failed close of the recorded open pull request. Card 15: a failed `Status` read in `pullRequestLanding`, and a failed `FastForwardPair` after a successful GitHub merge.

11. LOW. Card 14, failed non-transient session start: Stuck naming `lym loom resume`. The card does not say what `StartSession` does under a covering pause at row entry. `ensureSession` runs before `wait`'s pause check, and shuttle's start under a pause may block or refuse.

Tool calls:
- `grep Yield` in `internal/shedengine/producer.go` and `run.go`: what `Yield` does was not in my context.
- `grep RunYielded` outside shedengine: who handles a yielded run was not in my context.
- `grep Yielded` in shedverbs, sheddriver, loomcli and battenshed: whether a driver re-calls after a yield was not in my context.
