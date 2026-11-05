```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each review proposal the fixer carried out (F1–F5) by building cases where the proposed change itself is wrong.

Proposals fixed as proposed: F1, F2, F3, F4, F5. All five were challenged.

1. **MEDIUM. F1: a new post resets the count, so the spawn count has no cap across the hour.**
   - Changed text: D2, "a count kept per receiving hub that resets only when a strand posts a new message to that hub or the sending daemon restarts".
   - Case: a receiving daemon exits every time it starts. A strand of the sending hub posts to that hub once a minute, for example a chatty orch or a retry-by-hand loop. Each post resets the count, so the daemon is started up to three times per post, about 180 times in the hour.
   - Evidence: `pattern/PATTERN-child-processes.md:16` says "A retry loop around a real spawn caps attempt COUNT, not only elapsed time". The new D2 criterion holds only under "with no new post", and the reviewer's proposal carries the same gap.
   - Sounder change: reset the count only when that hub's daemon answers a call, or on a sending daemon restart, not when a post arrives.

2. **MEDIUM. F4: the rollout bound depends on an ordering the design cannot enforce.**
   - Changed text: D7 bound "every session of the machine's hubs launched before the deploy restarts before an orch of another hub mails it (D9)", and the D9 setup and plan-writer note.
   - Case: one deploy gives every hub's daemon, `lyt`'s included, the new route at the same moment. Between the deploy and the restarts, an orch strand of a sandbox hub can mail loomyard's orch, which still holds the old trust sentence. That sandbox orch is the injected-agent threat the design exists to contain, and nothing makes it wait for the restarts.
   - The bound states that the restart happens "before an orch of another hub mails it", but only the sender controls that ordering, not the operator or creel.
   - Evidence: D3's bound says "the sandbox hubs' agents can do each". D9's trial setup restarts sessions only "before the trial's first cross-hub post".
   - Either state the window as the residual exposure, or have a receiving daemon refuse cross-hub mail until something marks the restart done.

3. **LOW. F4: the exposure is wider than "an orch of another hub".**
   - Changed text: D7, "may follow an orch of another hub, trusted or not".
   - The old trust sentence says "an orch strand" (`internal/shuttleengine/claudeengine/settings.go:473`). That also covers a numbered `orch-N` strand of another hub, which D5 never trusts. The sentence names less than the old text lets through.

4. **LOW. F5: the quoted reason is not escaped.**
   - Changed text: D2, "cut to 200 bytes and quoted as text from the other hub's daemon, data and never an instruction". The same applies to the D2 bound "chooses those bytes and nothing else of the notice".
   - Case: a process bound to the receiving socket answers with text containing `"`, then text written as if it came from creel. The quote ends early and the rest reads as creel's own notice text.
   - Line breaks are made spaces, but quote characters are not escaped. Escape them, for example with Go `%q`, or say the quote is escaped.
   - A side point: a 200-byte cut can split a UTF-8 rune, so say the cut falls on a rune boundary.

5. **LOW. F2: the counted notice's place in the next window is not stated.**
   - Changed text: D3 bound "the first drop after the hour sends a notice counting the drops journaled since the last notice", and the D3 criterion "yields five notices … and the first drop after the hour sends one notice counting the drops since".
   - The record does not say whether that counted notice is also the first of the next hour's five. A test writer could pin either reading.
   - A related case: a forger who uses up a hub's five notices hides a genuine drop from that hub for the rest of the hour. That drop shows only through the fifth notice's warning and the journal. The reviewer offered this outcome as acceptable, so it is not a finding.

F3 was challenged and nothing wrong was found. A retry carries the journaled post time unchanged, and the journal's `time.Time` JSON keeps nanoseconds, so retries still match. The residual bound is stated.

Further tool calls: none.
