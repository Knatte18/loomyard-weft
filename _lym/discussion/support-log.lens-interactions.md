```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of the record's decisions that share state, keys or commands, and the acceptance criteria and constraints against the decisions they touch.

**Findings**

1. MEDIUM, D1 × D3: the lease expiry races the runner's own abandonment.
   - D3 says the runner checks its lease "at most once a second" and then kills its tiers, writes an abandonment exit (exit 1) and frees its slots.
   - D1 says an expired lease starts a fresh runner, "replacing what the directory held".
   - Counterexample: a front arrives within that check window, or while the old runner is still killing its tiers. Under the lock it sees an expired lease and starts a second runner in the same per-key directory. The old runner then writes its abandonment exit file there, and the new front delivers "exit 1, no caller came back" as the result of its own fresh run. For a short time the two runners also hold slots at once.
   - The record does not say in what order a front checks the exit file and the lease, and does not say that a fresh start waits for the old runner to end or uses a per-run directory or id.

2. LOW, D4 × D8 (`lym shuttle await`): the cap resolver needs a hub, but `lym shuttle await` deliberately needs none.
   - D4 makes `shuttleengines.KeepWarm` take the hub's config dirs and the caller's strand.
   - The command's own doc says it "needs no hub, repository, config or engine", and its PersistentPreRunE skips resolution for it.
   - The record does not say how await finds config dirs, or whether outside a hub it falls back to `DefaultAwaitCap` with a Warn on every call. A Warn on every call of a re-run loop also sits badly with the Level policy.

3. LOW, D5 × D3: abandoned runs skew the break-even estimate.
   - D5 takes the median of "the latest five completed calls".
   - D3's abandoned runs end early with exit 1 but still write tier records under the call id. The record does not say whether those count as completed. If they do, the median drops and a long gate key stays on the short cap.

4. LOW, D2 × D7 and Go sites: the capped form is chosen by the strand, not by whether an agent made the call.
   - A Go verb that an agent runs inherits the agent's `LYM_STRAND_NAME`. Any `lym gate test` it shells out to without `--wait` and without a slot would now detach and could return 75.
   - Every site found today uses `--wait`: `verifytree`, `gateslot.Config.GateCall`, and the `plancontext` rule that card Verify lines carry `--wait`. D7 deliberately renders a call without `--wait` for agents.
   - No decision or constraint states the invariant that a gate call Go runs always carries `--wait` (or that a Go site strips the strand). That invariant is what keeps D2 safe.

5. LOW, D7 rationale: it names the wrong gate key.
   - The rationale says the cold fixer "blocks on the round gate". The render in both `verifyfixer.go:103` and `runlevel.go:699` is `GateCall("webster-fix")`.

6. LOW, Scope × D4: wording is out of step with D1.
   - Scope and D4 say the resolver is "used by the gate's run and slot wait". Under D1 the slot wait runs uncapped inside the runner, so the front's single cap covers both waits.
   - Nothing contradicts, but the wording could lead the plan writer to keep a separate capped slot wait.

**Pairs checked**
- D1×D2 (in-process versus runner; the runner runs with `--wait` semantics so it never detaches again): held.
- D1×D3 (lease, discard on expiry, abandonment exit): broken, finding 1.
- D1×D5 (one runner and lease serving short and long caps): held.
- D1×D6 (card-verify keyed by verb and card id): held.
- D1×D9 (exit 75 and the rule text): held.
- D2×D4 (zero cap goes in-process; fallback gives a non-zero cap): held. An unparseable strand falls back to 250 s and takes the capped form, which is consistent with D2's bound.
- D2×D6 (the line inherits the slot inside the runner and runs in-process): held. A `--wait` card line takes no slot and is in-process through `--wait`: held.
- D2×D7 and Go sites (strand-based choice): held today, invariant unstated, finding 4.
- D3×D5 (long-cap lease is the long cap plus grace; under the Bash ceiling by the margin, Open risk noted): held.
- D3×runproc session end, EndSparing: held.
- D4×D5 (one margin for both caps; a 1h role's cap equals its long cap): held.
- D4×D8 (await default cap; 3600000 ms text): held on timeouts, finding 2 on resolution.
- D5×D3 (abandoned records in the median): finding 3.
- D7×Go sites (both renders feed prompts only): held.
- D8×D4 (llm driver unchanged): held.
- Acceptance criteria D1 to D9 against their decisions: consistent. D1's "third call starts a fresh run" matches the delivered-result rule, and D2's "no strand waits for slots without a cap" matches D2. Constraints: PATTERN-gate-slot holds, since verify sites stay `--wait`.

**Tool calls**
- Read `internal/websterengine/runlevel.go` around line 699 and grepped the `fixGate` uses, to check whether the D7 render also feeds a Go verify site. My context had only the `GateCall` call sites, not where their output goes.
