```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked the changed decision record and support log against their own siblings and criteria.

1. **MEDIUM.** The D3 acceptance criterion contradicts the new D6 refusal.
   - Changed text, D3 criterion: "`lym slack away` and `lym slack start` run from a non-orch strand's pane bring up a relay whose first read creel accepts as `&lt;shortname&gt;:SLACK`".
   - Changed text, D6 bullet 4: "`lym slack away`, `lym slack back` and `lym slack stop` refuse when run from a live strand's pane other than an orch's".
   - Changed text, D6 criterion: "`lym slack away`, `back` and `stop` run from a non-orch strand's pane refuse and leave the presence unchanged".
   - The F1 criterion still has `away` succeed from a non-orch strand's pane, while the F8 refusal says it refuses there.
   - Only `lym slack start` can carry that F1 case now, or D6 must say a refused `away` still starts the relay.
2. **LOW.** D8 bullet 1 disagrees with the changed bullet 2.
   - Bullet 1 still says dobby "checks once a minute, while presence is `slack`".
   - The new bullet 2 has dobby record the wait start "whatever the presence", so it must also read children while presence is `terminal`.
   - Bullet 6, "on a return to `slack` the next check reminds of a wait already past the interval at once", depends on that.
   - Bullet 1 was left as it was and no longer describes when dobby reads a child.
3. **LOW.** In D5, bullet 4 acknowledges single messages, but creel acknowledges a whole read.
   - Bullet 4 (new): "At a read it acknowledges, without posting, each message creel stamped more than 24 hours earlier".
   - Bullet 5 (new) acknowledges "the read once every id is posted or skipped".
   - Evidence: `internal/creelengine/doc.go:35` says "A read's response is acknowledged on the same connection", so the response is acknowledged as one.
   - Bullet 4 should say the old messages are skipped, recorded under bullet 5's record and covered by that read's acknowledgement.

Tool calls:
- One grep of `internal/creelengine/doc.go` for acknowledgement wording, to learn whether creel acknowledges a read or a message. My context did not hold that text.
