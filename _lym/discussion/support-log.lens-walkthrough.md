```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case and checked the end state against the decision's purpose.

**Decisions walked**

- **D1** — case: a `driver`-role agent runs `lym gate test ./internal/x`, and the tiers take 7 minutes. The front forms the key, starts the runner and answers 75 at 250 s. The re-run joins at about 255 s, finds the exit file at 420 s, replays the output and exits with the run's code. Purpose met, with findings 4 and 5.
- **D2** — cases: a card-verify line `lym gate test --wait --card 3` runs inside the runner's slot, so it inherits the slot and runs in-process. A verify command inside a tier gets no strand, because the tier environment strips it, so it runs in-process. An operator call with no strand blocks to its end. Met.
- **D3** — case: the agent is interrupted after the 75 answer. The lease (cap plus grace) passes, the runner cancels, the slots free and the abandonment is recorded. A front arriving later starts a fresh run. Met, with finding 6.
- **D4** — cases: role `driver` gives 5m − 50s = 250 s and role `webster` gives 1h − 50s = 59m10s. The strand-role lookup and the `keep_warm_margin` validation step get different inputs than the record assumes (findings 1 and 2).
- **D5** — case: a 5m role, `--gate webster-fix`, with a 2-hour median. 2 h is above 250 s × 10 ≈ 41.7 min, so the long cap is 59m10s and the run takes two long calls, each followed by a cold return. That matches the stated rationale. Met, with finding 7.
- **D6** — case: an implementer fork's `lym gate card-verify 3`. The runner takes the slot and the inner `--wait` call inherits it. Met, with finding 8.
- **D7** — case: the cold verify fixer (role `verify-fixer`, 5m) runs the rendered `lym gate test --gate webster-fix` with no `--wait`. It gets the capped form, and D9's rule text tells it to re-run on 75. Met.
- **D8** — case: `lym shuttle await x` with no `--cap`. The await verb resolves no hub today (finding 3). Met once that is added.
- **D9** — case: the rule text covers a 75 answer for both a busy slot and a running run. Met.

**Findings**

1. **MEDIUM — D4.** The resolver reads the role from the caller's strand, but reed numbers duplicate roles. `reedengine/strand.go` forms the name with `agentname.NumberRole`, so a strand's role can be `webster-2`. `Prepare` resolves the session's TTL from the unnumbered `Spec.Role`. A second Webster session would get the 5m fallback lifetime and a 250 s cap, although its cache lives 1h. The error only ever runs toward a shorter cap, so it costs money, never a verdict. Fix: the resolver strips a numbered suffix (the `agentname.MatchesRole` form) before the lifetime lookup.
2. **MEDIUM — D4.** `keep_warm_margin` is to be refused by shuttle's `LoadConfig` when it is "below the shortest lifetime the engine accepts". That function cannot know the accepted lifetimes: `shuttleengine` imports no concrete engine (PATTERN-shuttle-provider-seam), and 5m and 1h are claude facts. The validation needs another site, such as the `shuttleengines` resolver or the claude engine's check that runs at run start, or a fixed bound stated in shuttle terms.
3. **LOW — D4/D8.** `lym shuttle await` skips resolution (`internal/shuttlecli/cli.go:62` skips it for `awaitVerb`) and is documented as needing no hub. The resolver needs config dirs. The record should say that await now resolves the worktree's config dirs when it can and falls back outside a hub, so the help text "needs no hub" stays true.
4. **LOW — D1.** The replay copies the recorded stdout and then the recorded stderr. An in-process call interleaves tier output and summary lines as they happen, so a replay reads differently from "exactly like today's in-process call". The verdict and exit code are unaffected. Fix: state the replay order as the bound, or weaken the claim.
5. **LOW — D1.** The cap answer names "the running tier", but no step in D1 hands the front that fact. The notes cover holders from the pool's state, not the runner's current tier. The pool's held-slot entries carry `tier` and `pid`, so the front can match the runner's pid. Name that source, or have the runner write its current tier to the run directory.
6. **LOW — D3.** The "fixed grace" has no value. It must cover the agent's next model turn, which can include an API retry or a compaction. A grace shorter than that kills a live caller's run. Passed tiers carry on the green record, so the cost is wasted time, not a verdict. State the grace or what bounds it. Also unclear: whether the lease is also checked during the runner's slot wait ("while it runs").
7. **LOW — D5.** On a 5m role, the median of the latest five calls includes carried or empty-scope calls that take near zero seconds. They pull the estimate down, so the long cap is picked less often. This only mis-picks, which the stated bound already covers. Optionally exclude carried calls from the median.
8. **LOW — D6.** A card's Verify line that is not a gate call and exits 75 itself replays as exit 75. The agent then re-runs, finds the result delivered, starts a fresh run and loops. Only the `done: false` envelope tells the two 75s apart. State that the agent re-runs only on `done: false`, or bound it.

**Scope note, outside this lens:** the board's motivating case is a Webster gate of about 11 minutes on OSL-1046. That gate runs as a Go verify site with `--wait` while the Master sits idle, so D1 does not touch it. D8 relies on the template giving `webster` the 1h lifetime, which another hub's `claude.yaml` may not do.

**Tool calls**

- grep for `NumberRole(` callers and for `awaitVerb` in shuttlecli: my context did not show whether reed numbers strand roles or how await skips resolution.
