```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked every bound and claim in the record. Four are broken: one MEDIUM, one LOW-to-MEDIUM, and the rest LOW.

**Claims attacked**
1. **Broken.** D1/D3: each front's lease keeps the runner alive for every front still joined.
2. **Broken.** D1 bound: the replay gives the same content and exit code as an in-process call.
3. **Broken.** D5: the median leaves out an abandoned call.
4. **Broken (partly).** D2 bound and Constraint 2: a strand only decides how long a call blocks, and every gate call a Go path runs carries `--wait`.
5. **Unstated, can break.** D3: the runner reads the lease from its own start.
6. **Held.** D1: a superseded runner's late writes never reach a newer caller. Each run has its own id and subdirectory, and the key directory names the current id.
7. **Held.** D1 bound: an edit to an already-dirty file gets the earlier content's run. The bound is stated, and `gatecli/doc.go:194` records a green pass only over an unchanged clean tree.
8. **Held.** D3 bound: a superseded runner frees its slots within one lease read plus its kill time. Its slot wait runs under a context that the once-a-second lease read can cancel (`run.go` `acquireSlot` passes `waitCtx` to `AcquireSlots`).
9. **Held.** D5: the long cap fits the Bash ceiling. 55m plus the 120 s grace is under `bashMaxTimeout` of one hour (`claudeengine/command.go:120`). The gate rule already tells agents to use 3600000 ms (`settings.go:274`).
10. **Held.** D2: a nested gate call inside a card-verify line runs in-process. Inside a hub a non-`--wait` line holds a slot and passes `InheritEnv` (`cardverify/runner.go:75`). A `--wait` line is in-process by its flag.
11. **Held.** D4: numbered strands resolve to the same role the lifetime lookup uses. `reedengine/strand.go:243` numbers only the strand name; `Spec.Role` stays the base role that `resolvePromptCacheTTL` reads.
12. **Held.** D8 bound: a hub whose `prompt_cache_ttl_roles` drops `webster` puts the Master on the 5-minute lifetime. Matches `claudeengine/template.yaml`.
13. **Held.** D3 bound: an abandoned runner holds its slots for at most cap plus grace after its last front, given finding 1 is fixed.

**Findings**

- **F1. MEDIUM, design, D3 (with D1, D5).** A later front can shorten the lease below an earlier front that is still waiting, and the run then ends as abandoned.
  - D3 says each front writes the lease as its own cap deadline plus 120 s. It does not say the lease only grows.
  - Counterexample: front A takes the long cap (D5) and writes now + 55m + 2m, then blocks. Front B makes the same call with the same words, so it has the same join key. B can be another strand, a fork whose role cap differs, or a call made after the median moved below the break-even. B joins with the short cap and writes now′ + 250 s + 120 s.
  - About 6 minutes later the runner reads a lease that has passed and kills its tiers. Its exit file records "no caller came back" with exit 1. A is still blocked, and it replays exit 1, a failed verdict, even though a caller was waiting.
  - This breaks D3's bound and D5's claim that "a wrong pick costs cache money only, never a verdict".
  - Fix: the lease is the latest deadline any front has written, and only the supersede expiry may move it earlier.
  - Evidence: `decision-record.md` D3 "Each front writes the lease…" and D5 bound.

- **F2. LOW, consistency, D1.** The bound "The replay gives the same content and exit code as an in-process call" contradicts D1's own rule that a recorded 75 exits 1.
  - Under `--wait`, the runner can only record 75 as a test's own code (`gatecli/run.go` `acquireSlot`: the slot-busy answer exists only without `--wait`). An in-process call returns that 75 unchanged, so the replay's exit code differs.
  - Fix: the bound names the exception.
  - Evidence: `decision-record.md` D1 first bullet against its Bound paragraph.

- **F3. MEDIUM, design (deferred research question), D5.** "Leaving out … an abandoned call" depends on a record fact that does not exist.
  - `gateslot.RunRecord` has no abandoned or cancelled field (`internal/gateslot/runlog.go:38-73`).
  - A tier the runner's cancellation kills still writes its record through `execution.report`. It gets outcome failed and its exit code, and only later tiers go unrecorded (`internal/gatecli/run.go:479-497`; `halt` is `e.ctx.Err() != nil`).
  - So a call abandoned in its last listed tier leaves a record for every listed tier, each "to an outcome". It reads as a full call with a short wall time and pulls the median down.
  - The record also does not say which time the median reads: the summed `WallSec` or the span from `WaitStart` to `End`. Only the span includes the slot wait the agent actually blocks through.
  - Fix: record which fields mark an abandoned call. That can be the exit file's abandonment, or a new record field marked (new). Also record which time the median reads.

- **F4. LOW–MEDIUM, design, Constraints and D2 bound.** Constraint 2 says every gate call a Go path runs carries `--wait` or runs in an inherited slot. One Go path does neither for operator text.
  - Batten's Land row runs the landing config's operator-written `deploy` command through `verifyrun.Run` with a nil env, so it inherits the process environment, and it holds no slot (`internal/battencli/land.go:145-155`).
  - If that environment carries `LYM_STRAND_NAME` and the deploy command is a `lym gate test` without `--wait`, D2 detaches it. The call returns 75 `done: false` within the cap, and the Land row reads 75 as a failed deploy.
  - So the D2 bound "a forged or absent strand chooses only how long a call blocks, never what runs or its verdict" fails for that path.
  - Unverified: I did not confirm whether dobby's process, which runs the Land row, carries a strand. My grep found no strip of it in `internal/dobbyengine`, and `verifytree.Verify` strips it explicitly (`verify.go:382`), which suggests other Go runners do not.
  - Fix: the Constraint states how operator-configured commands that Go runs are covered, for example by stripping the strand in `verifyrun` callers. Or the record researches and records that no such path carries a strand.

- **F5. LOW, design, D3.** The order between starting the runner and writing the first lease is not stated.
  - The runner "reads the lease … from its start". A front that spawns the runner and then writes the lease gives a window where the runner's first read finds no lease.
  - If no lease reads as passed, the runner abandons at once. If it reads as holding, a front that dies between the spawn and its write leaves a runner with no lease bound.
  - Fix: the record states that the front writes the lease before spawning, or how the runner reads a missing lease.

**Tool calls**
- `internal/battencli/land.go` `runDeploy`, and a grep of the dobby and orch beam-up env: to see whether a Go path runs operator text with the inherited strand and no `--wait`. Not in the manifest.
- `internal/gatecli/tierrun.go` cancellation lines and `run.go` `execution.report`: to see what record a tier killed by cancellation writes, which D5's "abandoned" exclusion depends on.
