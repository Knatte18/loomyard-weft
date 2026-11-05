```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that every refusal, error, next step, row or test the fix added can be reached and leads to progress.

**F1 (MEDIUM): a creel refusal that never clears blocks every later Slack message, presence commands included, and the Slack answer gives no next step.**
- Changed text: D4's second bullet: "…is retried with the poll's backoff, without limit, so later messages wait behind it; after a minute of failures the relay answers once in Slack that the message waits, quoting creel's refusal."
- The operator's `away`, `back` and `status` texts arrive as ordinary messages, so they also queue behind the stuck one. From the phone the operator cannot even check `status`.
- The quoted refusal names no step. `internal/creelengine/identity.go:69` reads: "creel: the calling process is in no known strand pane and the pane process of X is unknown, so it cannot be told from a strand; nothing was done".
- No decision says what clears an unknown pane process, or that it clears without the terminal.
- Under the same condition the relay's own creel read is also refused, so its outbound mail stops too.
- So the operator, who is reachable only through Slack, is told the relay is stuck but gets no action that unsticks it.

**F2 (LOW): the `lym slack away` refusal quotes the same creel refusal, which offers no remedy.**
- Changed text: D6's second bullet: "…and names creel's refusal when creel refused that read, as it does while a live strand's pane process is unknown (D4)."
- The quoted text (`identity.go:69`) ends "nothing was done". The operator at the terminal learns why `away` failed but not what to do: which strand to end or fix, or whether waiting helps.

Checked with no finding:
- The `login`, `setup` and `start` refusals name a terminal or an orch's pane, and both can be reached.
- A read is released by any answer other than the acknowledgement, per the `awaitAck` doc at `server.go:478`.
- The re-exec Bound's loser exits before polling, leaving one relay.
- The added criteria (channel change, refused creel post, backlog past the window, the in-pane relay start) can each be set up in a test from outside every pane.

Tool calls:
- One grep in `internal/lymdirs` and `internal/pauserecord`, `internal/creelengine`: it looked for where the pause record lives and found nothing, then read creel's refusal text, because my context did not show the full message the new Slack answer and `away` refusal quote.
