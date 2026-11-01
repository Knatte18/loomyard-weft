```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: I traced each existing code path that writes or reads what D1–D9 introduce or change.

**Findings**

1. **BLOCKING — D4 and D6: a fork's gate call gets the 1-hour cap while its cache lives 5 minutes.**
   - The cap is resolved from the caller's strand role. A fork runs inside its Master's session, so it has the same strand: an implementer or fixer fork under Webster resolves the `webster` role and a 59m10s cap.
   - The claude engine doc in context says the subagent cache bucket (`CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL`) is left at Claude Code's default. So a fork's own cache does not get the role's 1h lifetime.
   - Result: an implementer fork blocked on `lym gate card-verify`, the main caller D6 targets, waits up to 59 min on a cold cache. The record never says how a fork is told apart from its Master, or that it keeps the 5-minute cap.

2. **MEDIUM — D4: `lym shuttle await` cannot reach the config as it stands.**
   - `awaitCmd`'s doc in context says the verb's PersistentPreRunE skips resolution because it "needs no hub, repository, config or engine".
   - D4 has its default `--cap` read `shuttle.yaml`, `claude.yaml` and an engine through config dirs that the verb never resolves.
   - The record names only a fallback "when the config cannot be loaded". It does not say the verb now resolves a hub, or what it does outside one. As written, every call from a 1h role falls back to 250s, so D4's acceptance line for that role cannot hold for `await`.

3. **MEDIUM — D7: one agent-facing render is missing.**
   - `internal/loomcli/roundgate.go`'s `gateCallSource` renders `cfg.GateCall(key)` for "the verify gate, the landing producers and the Publish failure note".
   - The Publish failure note is text that an agent or fixer re-runs, and it still carries `--wait`.
   - D7 lists only the two webster fixer renders. The record should say whether the note keeps the blocking form.

4. **LOW — D3: an unregistered runner in the prime.**
   - D3 leaves a prime runner unregistered.
   - `tenderengine` reaps "unregistered orphans" only outside registered worktrees, and the prime is one, so this holds.
   - Still, `lym tender` reports would not show a live prime runner. Worth one line in D3's bound.

5. **LOW — D1: replay loses interleaving.**
   - Today a tier's output goes to stdout and its summary and closing lines to stderr, interleaved as they happen.
   - D1 stores them in two files and replays them, so the agent sees all output, then all summary lines (or the reverse).
   - The verdict and exit code are unchanged, but a failing tier's summary line no longer sits next to its output. The record says the result "reads exactly like today's in-process call", which overstates it.

6. **LOW — D4: stale `gateslot` doc.**
   - `internal/gateslot/doc.go` says `cli_wait_sec` is retired "since the slot wait is bounded by the keep-cache-warm cap with no knob".
   - D4 adds the `keep_warm_margin` knob, so that rationale and the gatecli Slots section go stale. A doc update only, not named in the record.

**Paths traced, per decision**

- **D1**
  - Gate wait records (`wait-*.yaml` glob) and `lym gate status` `held`: these record the runner's pid, and the per-key run directory sits beside them without matching the glob. Survives.
  - `green.json` and its lock: the runner writes it as the in-process call does. Survives.
  - Run records: `WriteRunRecord` prunes files older than 14 days and runs in the runner. Survives.
  - Watchdog `LYM_GATE_TEST_PID`: the runner is the outermost run. Survives.
  - Output replay order: breaks the "exactly like today" claim (LOW, finding 5).
- **D2**
  - Verify sites (`verifytree.Verify`, `gateslot.Config.GateCall`'s `--wait`): survives.
  - `cardverify.Runner` re-run from Go under Master's inherited strand: `plancontext` requires `--wait` on gate Verify lines, and in a hub the inherited slot keeps it in-process. Survives.
  - `adopt check` through `RunGate`: survives.
  - Shellguard deny on agents naming the strand variable: the spawn is from Go. Survives.
- **D3**
  - Tender reap of a slug-owned record: the slug's worktree exists, so it is never an orphan. Survives.
  - Prime runner: survives, with a reporting gap (LOW, finding 4).
  - Pair teardown through `runproc.End`: survives.
  - Hibernate and pause: no gate or slot handling found in those engines, so a runner keeps running while the worktree is paused and its lease decides. The record is silent here, but this is not a regression I could pin.
- **D4**
  - Strand role versus `Spec.Role`: numbered strand roles such as `webster-2` miss the map and get the shorter cap. That is safe.
  - Fork strand: breaks (BLOCKING, finding 1).
  - `lym shuttle await` resolution skip: breaks (MEDIUM, finding 2).
- **D5**
  - `lym gate report` and the adopt observer: these read records without the new call id, which tolerant reads absorb. Survives.
- **D7**
  - `verifyfixer.go` and `runlevel.go` webster-fix renders: survives.
  - `loomcli` `gateCallSource` failure note: not covered (MEDIUM, finding 3).
- **D8 and D9:** no conflicting path found.

**Tool calls**

- Grepped `tenderengine` doc for orphan and reap rules: my context lacked how tender treats records and unregistered processes.
- Grepped `gateslot` `wait.go` and `doc.go` for sweeps and prunes: I needed to know whether anything clears unknown files in the wait directory.
- Grepped `hibernateengine` and `pauserecord` for gate or slot handling: my context lacked how pause and hibernate meet a running gate (no hits).
- Read `websterengine/runlevel.go` around the webster-fix render and `loomcli/roundgate.go`: I needed every other gate-call render site D7 must cover.
