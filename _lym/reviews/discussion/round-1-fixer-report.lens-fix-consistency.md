```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-1 fix to the decision record and support log agrees with the sections it touches and with their counterparts elsewhere (fix-consistency lens).

Findings:

1. **MEDIUM.** D1, line 27: "Claim, 8 KiB budget, hold, pause and the acknowledge-after-write order stay as `Store.ClaimAtBoundary` has them". This contradicts the new D1 paragraph.
   - The new paragraph says "A message the old binary stored whose line passes 10,000 characters is left out of every claim".
   - `ClaimAtBoundary` (`internal/creelengine/verbs.go:183-198`) has no such exclusion today. So the claim does change, and line 27 now says it does not.

2. **MEDIUM.** D9: "to `append-system-prompt.txt` beside the run's `settings.json`". This location does not exist for every session D9 covers.
   - D9 says "Every Claude session shuttle launches", and the auto-mode assumption adds "standalone ones included".
   - The fix's own D5 text says a `lym shuttle open` session "runs under its worktree's own settings".
   - `internal/shuttleengine/open.go` says "It writes no run directory", and `buildOpenCmd` passes no `--settings` and no appended prompt.
   - So the new file location has no home for open sessions, and D9 does not say whether open sessions get the sentence.

3. **LOW.** D7: "it validates the post first (receiver, body size, sender)". D1 replaced the body cap with a cap on the message's line, so this should say line size, not body size.
   - Evidence: D1 now reads "That line cap replaces the 64 KiB `maxBodyBytes` cap".

4. **LOW.** D6 and the acceptance criterion "Agent mail is delivered in full" now disagree with D1's exclusion.
   - D1 says an old-binary line past 10,000 characters "is left out of every claim".
   - D6 still says "agent mail is delivered in full by D1 and D2 like any other mail".
   - The criterion bullet states the same, with no exception.

5. **LOW.** "Restarted" is now ambiguous.
   - D5 says the `CreelHooks` mark is "kept through reed's verbatim resume". So a reed resume replays the old launch line and leaves the strand unmarked, and only a fresh shuttle launch marks it.
   - Open risks ("until restarted") and Scope Out ("they get posts once restarted") do not say a reed resume does not count.

6. **LOW.** The support log's question ledger still has Q3 reading "shuttle-launched strands only (D5)". The fix narrowed D5 to strands marked `CreelHooks`, which leaves out shuttle-launched strands such as `lym shuttle open` sessions and sessions launched before the deploy. The fix revised Q10 but left this matching entry unchanged.

Tool calls:
- `sed -n 129,260p` of `decision-record.md`: my context showed the changed record only up to D7's heading. I needed the changed D7–D11, Constraints, Open risks and Acceptance criteria.
