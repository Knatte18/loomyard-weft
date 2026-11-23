```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's changes agree with the sibling cards and cross-references they touch.

**[MEDIUM] Card 2, `RunCommitsAhead`: the ended-run read covers the records branch only, while `TipsOnOrigin` now covers both sides**
- Changed text: `02-fabric-pusher-reads.md`, the `RunCommitsAhead` bullet "A pair's records branch whose run the ended set reads ended reads level too…" next to the `TipsOnOrigin` paragraph "A pair whose run the ended set… reads ended reads held on both sides".
- Summary: the two reads disagree for an ended pair. The code side of `RunCommitsAhead` still reads ahead when the code branch holds the named tip and its unpruned tracking ref lacks it. That brings back the F13 restart loop on the code side.
- Evidence: the code side's `checkOrigin` returns `false, nil` on `view.ended` (`internal/fabricengine/pushclassified.go:641-642`), so the push "lands" with no error. Card 5's `Run` step 4 then reads `Ahead` again and pushes once a second while it reads ahead. Card 8's tick restarts a pusher on `StartIfAhead` as well. `TipsOnOrigin` reads the same pair held on both sides, so the two reads give opposite answers for one pair.

**[LOW] Card 5, the `Target` paragraph gives a reason card 3 no longer supports**
- Changed text: `05-runpusher-package.md:53`, "A prime's target always carries an empty slug, whatever slug the caller passes, so the prime's push reads every batten run's takeover (card 3) and never one run's alone."
- Summary: after the F1 fix, card 3 runs `primeTakeovers` on every prime records push "whatever `runSlug` is". The empty slug is no longer what makes the prime's push read every takeover, so the "so" clause is stale.
- Evidence: card 3's rewritten `retryWeftPush` bullet, "runs `primeTakeovers` after its fetch and before any replay or lock, whatever `runSlug` is".

Tool calls:
- One grep over `_lym/plan/0*.md` and `_lym/plan/1*.md` for StartPush, RestartPusher, slug, owned-elsewhere, tender, StatusPushLockWait, TipsOnOrigin, RunCommitsAhead and pusher_started. It finds sibling mentions in cards 9 and 11 and in the overview, which my context showed only in part.
