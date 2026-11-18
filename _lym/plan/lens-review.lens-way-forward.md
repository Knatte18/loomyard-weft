```yaml
lens: way-forward
outcome: reported
```

Way-forward lens: I checked each refusal, stop and recovery step the plan adds, and whether the next step it names can succeed from the state it leaves. I found five defects.

Next step for each refusal, and whether it progresses:
- **Card 3, uncaused halt:** commit a `Loosened-Check:` trailer or a revert, then `lym loom resume`. Progresses: the trailer changes the key, so the file is ruled again.
- **Card 2, judgement refusals:** a re-prompt through `NewJudgementGate`. Progresses.
- **Card 8, dirty worktree:** commit or discard, then run the verb again. Progresses.
- **Card 8, unparsable file:** rewrite the file, then run the verb again. Does not progress, because the way forward names a path the agent cannot read (finding 2).
- **Card 8, no pull-request record:** wait for PR-Review to open the pull request. Progresses.
- **Card 8, a mark already recorded:** the verb answers the send-back again. Progresses.
- **Card 8, a failed record commit after a write:** run the verb again. Does not progress for fix-here (finding 1).
- **Card 9, dirty-tree halt:** commit or discard, then `lym loom resume`. Progresses: the resumed row meets its own Stuck as the newest entry and reroutes.
- **Card 9, reroute-cap halt:** the shed resume. Progresses: `rerouteCount` stops at the RerouteCapped entry. See finding 5 for the case where it cycles.
- **Card 6, missing fragment:** a read error that names no way forward (finding 4).

1. MEDIUM, card 8, step 6 and its commit. Running the verb again after a failed commit counts the same fix-here findings twice.
   - The verb writes `FixHereCount` into the record, then commits through `pullRequestRecordCommitter` "as `pr-continue` does".
   - `pr-continue`'s failure text says to run it again (`internal/loomcli/prcontinue.go:84`). That verb is idempotent; the fix-here branch is not.
   - Running it again adds the same set to the count a second time, which can push the total past `smallMax` and send back a set the run should fix in place.
   - Neither the card nor `TestPRFindingsVerb` covers a commit failure or a repeated fix-here call. Name a way forward that does not double-count, such as a commit-only retry, and plan a row that reaches it.

2. MEDIUM, card 8, refusal 2. The way forward points the PR-Review agent at `contracts/stencils/burler/burler-review-format.md`.
   - That path exists only in the lym source repository. A hub's task worktree has no `contracts/` directory; the stencils are seeded into the Bolt checkout's `_lym/stencils/`, as the header comment of `landing-template-pr-review.md` (lines 1-4) says.
   - So the agent cannot follow the named next step. Name the seeded stencil path, resolved from the stencils directory, or state the format's required keys in the message.

3. MEDIUM, card 8, the error branch of `sendBackEdge`. The card declares `(bool, string, error)` but never says what the verb answers when the error is non-nil.
   - The error can come from a target `EntryGate` evaluation or from `loomrecipe.New` while the command wires the producers.
   - No refusal text or way forward is given, and no `TestPRFindingsVerb` row reaches it.
   - `PATTERN-refusal-way-forward` wants a read failure worded with its way forward, as `samplerDoneReadFailure` does (`internal/loomcli/samplerdone.go:41`). The card should also say the record is left unchanged on this error.

4. LOW, card 6, the missing-fragment error. The error a missing plain include raises "names the stencil" but no way forward.
   - The stencil's own include is broken, so the agent cannot fix it; the error ends the producer's render.
   - Such errors are usually worded with `lym stencil diff` and `lym stencil sync`, as the friction warning at `internal/friction/friction.go:117` does. The planned `TestReadForEngine` row asserts only the name.

5. LOW, cards 8 and 9, the reroute-cap way forward. For a BLOCKING finding that Code-Review keeps approving, the cap halt's way forward cycles.
   - Each PR-Review entry after a send-back asks the parent again. If it returns the same BLOCKING finding, the verb always sends it back while the edge can fire.
   - Resuming after the cap grants only more of the same reroutes. The one exit that ends in fix-here is a Code-Review budget of 0 set through `run_settings`, and no halt text names it.
   - The decision record accepts the cap as the bound, but the halt the parent sees gives no step that lets the run finish. Card 9 or the PR-Review brief could name that fallback.

I made no further tool calls; my context held everything this lens needed.
