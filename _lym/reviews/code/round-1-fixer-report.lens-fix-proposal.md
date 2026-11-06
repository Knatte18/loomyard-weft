```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1 proposal (restore `window.notices` and `window.undisclosed` when the notice post fails) as carried out in `receive.go` and `crosshub_test.go`.

No findings.

I built these cases against the proposal and none holds:
- **Count after restore:** `undisclosed` is restored to `counted`, which already includes the current drop, so the next drop's notice says "2 drops". The new row asserts this.
- **Failed fifth notice:** `notices` goes back to 4, so the "further drops this hour are journaled without a notice" suffix is not claimed by a notice that never reached the orch. The next successful post carries it.
- **Retry cap:** the five-per-window cap now limits successful notices, not post attempts, so a persistently failing post is retried on every drop. That needs the journal to fail on the notice write right after the drop's own write passed, and it costs one rejected write per drop. It is not a breakage.
- **Window rollover:** a failed post in a new window restores the window's own `undisclosed`, which already holds the carried count. Nothing is lost.
- **Partial write:** if `writeLine` fails after the line reached the file, for example on a sync failure, the restore could allow one duplicate notice later. That is the same rare failure as the finding, and a duplicate notice is safer than a lost one.

One call-site check: `noticeDropLocked` has a single caller, `dropLocked`, and its signature is unchanged.

Tool calls: none. My context already held every file and the review.
