```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case, feeding each step the data the step before it actually produces.

**Decisions walked:**
- D1, hardener recipe on the loom arm: `verifyGatedRows` gives the HardenFix row the verify budget once it joins `LoomBuildEngines`, and with no Code-Review ring there is no second row. Purpose met.
- D2, task run_settings `{"hardener":{"auto_accept":true,"packages":[...]}}` through Seed-Child: refused at `PrepareRunSettings`. Purpose not met (F-W1).
- D3, list [Opus, Fable, Sonnet] expanded to hard cap 10: rounds 1–10 get 1,2,3,1,2,3,1,2,3,1, and Preflight's adjacency check covers the first/last wrap. A review with no findings still runs the round's gates (`burlerengine/fix.go:6`), so the live report is written. Purpose met.
- D4 step 1, `harden-live` set to [2] over a package that has only tier-1 tests: `{test_targets}` narrows to nothing, so the run is green with no test started. Purpose not met (F-W3).
- D4 `--copies 3` on lym's hub (3 slots, tier2 asks 1 slot): the call holds all 3 slots, then its own tier run asks for one more. Purpose not met (F-W2).
- D4 live_attempts spent on a red `harden-live`: the row ends, the report is marked red, the failures are deferred and the next facts carry them. Purpose met.
- D5, `sandbox new`, then a round, then the end-of-row reap: the root lives under the run worktree's `.lym`, recorded owners are ended through `runproc.End`, and the hub list derives each server key. Met, except the `selfreport` half of the committed override (F-W4).
- D6, round n's facts read by the judge after the fix row: the models, live report, fix commits and stray report are all written before the Bouncer's next pass. Purpose met.
- D7, archive at the end of round n's fix row: the review, live report and strays are round n's, but the latest ledger is the previous judge pass's. Labelling gap (F-W5).
- D8, hub `agents.yaml` with `skills: {treadle-judge: [...]}`: `applyReinterpretations` rewrites the base file once behind its marker, and `skillLists` then loads. Purpose met.
- D9, run_settings override of every key: guard-category keys are refused by the local layer. Purpose not met (F-W1).
- D10, recipe Bouncer –on_stuck→ Review –on_done→ Fix –on_done→ Bouncer: the Bouncer lies on the on_done path after Record-Write, the fix row is reached through ring edges, and a second HardenFix row is refused by `onlyRowOf`. Purpose met.
- D11, D12: cobra subtree and docs move. Nothing to execute; met.

**Findings:**

- **F-W1 — MEDIUM, design** — `_lym/discussion/decision-record.md:216` (D9, with D2 at :49)
  - **Summary:** D9 says a task's run_settings may override any `hardener.yaml` key. But run_settings become the pair's local layer, which accepts only keys classified `CategoryNone`.
  - **Walk:** `{"hardener":{"auto_accept":true}}` → `PrepareRunSettings` → `checkLocalDocument` refuses the key, because the repo classifies these keys as guards:
    - `auto_accept` (`internal/reviewbudget/reviewbudget.go:42`)
    - the fan keys (`internal/loomengine/config.go:649-656`)
    - `tree_scope`, whose source says "no task run_settings sets it" (`internal/darnengine/config.go:59`)
  - **Also affected:** `packages` narrows the live gate and `llm_driving: true` switches off the concurrent copies. By PATTERN-config-layers' rule ("what a key's value can do to a guard") both are guard keys too, yet D2 needs both set per task.
  - **Gap:** the record classifies no `hardener.yaml` key, so D9's override promise and D2's per-task keys are left unresolved.
  - **Evidence:** `internal/configreg/runsettings.go:92-95`, `internal/configengine/doc.go:103`.

- **F-W2 — MEDIUM, design** — `_lym/discussion/decision-record.md:107-110` (D4 `--copies`)
  - **Summary:** the `--copies` call acquires n slots "for the call". Its tier run then takes the tier's own slots through `acquireSlot` → `pool.AcquireSlots`, with tier2's default ask of 1.
  - **Walk:** the call skips that acquire only when `LYM_GATE_SLOT` names a slot an enclosing process holds (`exe.inherited`). With n equal to the pool size (3 on lym's hub, concurrency 3), the call holds every slot itself, so the tier's acquire waits forever. The record does not say the tier runs inside the call's own lease.
  - **Second gap:** "runs n concurrent copies of each named package's compiled test binary" does not fit the gate's tier path. That path renders the hub's tier command (`go test -json {test_targets} {args}`, counter `go-json`) and holds no compiled binary to copy. Running copies bypasses the key's command, counter and timeout, which D4's own rationale ("the live runs are keys") rests on.
  - **Evidence:** `internal/gatecli/run.go:149,737`, `internal/gateslot/tiers.go:194`, `internal/gateslot/pool.go:179-298`, hub `gate.yaml` tier2.

- **F-W3 — MEDIUM, design** — `_lym/discussion/decision-record.md:87-94` (D4 keys) and `:54` (D2 Preflight)
  - **Summary:** lym's hub sets both live keys to `[2]`. A tier's `{test_targets}` is narrowed to the packages holding a test of that tier, so a `packages` list with no tier-2 test starts no test binary in either live run.
  - **Consequence:** the gate is green every round. CONVERGED's "green live gate" then rests on nothing, and the concurrent amplifier never runs, for a campaign over a tier-1 module such as a git-driving one. D9's defaults cite the fabric campaign.
  - **Gap:** Preflight checks that the key exists and does not name tier 3, but not that `packages` holds a test of the key's tiers.
  - **Evidence:** `pattern/PATTERN-test-speed.md:36-38`, hub `gate.yaml` (tier1 "integration tests", tier2 "tmux tests").

- **F-W4 — LOW, consistency** — `_lym/discussion/decision-record.md:126` (D5) and criterion `:313`
  - **Summary:** the committed sandbox `loom.yaml` sets `selfreport: false`, but `selfreport` is a retired `loom.yaml` key that the loader ignores. Only `friction: ""` switches off the automatic filer, since the friction reflection files issues through `lym selfreport create`.
  - **Consequence:** the criterion pins a key that does nothing, and Open risks' "the committed override stops only the automatic paths" rests on the `friction` half alone.
  - **Evidence:** `internal/loomengine/config_test.go:211-212` (row "retired selfreport key ignored"), `internal/friction/doc.go:49`.

- **F-W5 — LOW, consistency** — `_lym/discussion/decision-record.md:181` (D7)
  - **Summary:** the archive copy runs at the end of round n's fix row, before Harden-Bouncer judges round n. At that point "the latest ledger" is the ledger of the judge pass that opened round n.
  - **Consequence:** `round-&lt;n&gt;/` pairs round n's review and live report with the previous round's verdict. Round n's own verdict reaches the archive only in `round-&lt;n+1&gt;/` or the final settle. The record does not say so, so a reader of `round-&lt;n&gt;/` misattributes the ledger.
  - **Evidence:** the ring order in `internal/shedadapters/bouncer.go:265-268` (the Bouncer is the segment's entry and its OnStuck names the round producer).

**Tool calls:**
1. `internal/loomrecipe/darnrecipe.go` and a burler zero-findings grep: the other `LoomBuildEngines` consumer, and whether a no-findings round still runs its gates. Neither was in context.
2. `fabriccli.CloneAndWire` grep for registry side effects: D5's builder was not read.
3. `internal/fabriccli/clone.go` comment scan: the same question; nothing conclusive.
4. grep for the `selfreport` config key: D5's override key was unverified.
5. `selfreport` in the loomengine and friction sources: to locate the key's status. It found nothing in a template.
6. `loomengine/config_test.go` and `friction/doc.go` lines for `selfreport`: to confirm the key is retired and to find the automatic filer path.
