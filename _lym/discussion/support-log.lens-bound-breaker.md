```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens report

Findings:

1. MEDIUM. D1 Rationale and D6. The claim that "Exit 75 already means 'go on, run the same command again' to every agent prompt" is false for `lym gate card-verify`.
   - The implementer stencil (webster-body-implementer.md, step 5) says a non-zero exit from `lym gate card-verify` "fails the card exactly like the gate in step 3".
   - Its "Slot-busy exits" section exempts only "a gate call made without `--wait` that exits with code `75`, answering `slot_free: false`".
   - Counterexample: an implementer fork's card-verify passes the cap under D6 and exits 75 with `done: false`. The fork counts that as a card gate failure, spends a self-fix attempt, and after `self_fix_cap` reports FAILED.
   - D1 also replaces `slot_free: false` with `done: false`, which the stencil names verbatim.
   - D8 and D9 change only the timeout text and the shellguard rule. No decision updates the implementer stencil's 75 and `slot_free` text for the new answer or for card-verify.

2. MEDIUM. D7. The record claims that the renders filling an agent prompt are the verify fixers' `fix_gate` marker. They are not the only ones.
   - `internal/websterengine/cardgate.go` renders the card gate and the batch gate (`gate.GateCall("card", …)` and `gate.GateCall("batch", ids...)`) into the implementer prompt's `{{.batch_gate}}` and card-gate steps.
   - Those calls carry `--wait`, and the stencil says "A call made with `--wait` queues for its slot and never exits `75`".
   - Counterexample: an implementer fork's batch gate (tiers 0 and 1) runs in-process with no cap, so the blocked-agent wait the task targets stays for the most frequent agent gate call.
   - This is a wrong premise in D7's rule, not a missing call-site list: the rule should cover every gate call rendered into an agent prompt.

3. MEDIUM. D4 and D6. The cap for a fork comes from the session's strand role, but the fork's cache lifetime does not.
   - The claudeengine doc says the subagent bucket (`CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL`) "is left at Claude Code's default". A fork's Bash shell still carries the parent's `LYM_STRAND_NAME`.
   - Counterexample: a Webster implementer fork's strand role is `webster`, whose template lifetime is 1h, so the cap is 59m10s. The fork's own cache runs at the subagent default, 5m by default. Its card-verify blocks up to 59 minutes and its cache goes cold every time.
   - That defeats D6's stated purpose. D4 states no bound for fork callers.

4. MEDIUM. D1 Bound and D2 Bound. "Never what runs or its verdict" and "neither gets a wrong exit code for its own run" break when a later caller joins an earlier caller's runner.
   - The join key leaves out the tree, and the lease outlives the front by the cap plus the grace.
   - Counterexample: an agent's `lym gate test ./internal/x` front is killed when the operator interrupts with Escape. The operator says "fix X first". The agent edits, commits and re-runs the same words within 250 s plus the grace. That call joins the runner that started on the pre-edit tree and gets that tree's pass.
   - The record's bound covers only "edited between two calls" under the gate rule. An interrupted turn is outside that rule.
   - Same failure: two forks in one worktree that run identical words at different commits.
   - Fix: put the HEAD tree and the dirty state in the join key or check them on join.

5. MEDIUM. D4. "`keep_warm_margin` must be positive and below the shortest lifetime the engine accepts; `LoadConfig` refuses another value" cannot be done where it is placed.
   - `shuttleengine.LoadConfig` cannot know which lifetimes an engine accepts: PATTERN-shuttle-provider-seam bars `shuttleengine` from importing a concrete engine, and the accepted set (`5m` or `1h`) lives in claudeengine's `validatePromptCacheTTL`.
   - The check has to sit in `internal/shuttleengines` or behind an Engine method. As written, D4 contradicts its own Constraint.

6. LOW. D4. The cap is resolved from the current `claude.yaml`, but a session's lifetime was fixed at its launch ("fixed for the session's life, since a resume line is never rebuilt"). Two counterexamples:
   - The operator edits `prompt_cache_ttl_roles` mid-run, changing `webster` from 5m to 1h. A running 5m Master's gate calls get a 59m10s cap and a cold cache.
   - `FORCE_PROMPT_CACHING_5M` in Claude Code's environment "still wins", per claudeengine/doc.go, and the resolver does not see it. A 1h-mapped role gets 59m10s against a real 5m lifetime.

7. LOW. D4. The strand's role segment is not always the `Spec.Role` that Prepare resolved the lifetime from.
   - A numbered strand (`MatchesRole`, base-N) or a seat role with a suffix resolves an unmapped role, so it gets the default lifetime.
   - Counterexample: the default is 1h and `driver: 5m`. A `driver-2` strand gets a 59m10s cap while its session runs at 5m.

8. LOW. D5. "Median wall time of the latest five completed calls" counts calls that do not reflect a real run.
   - A fully carried call with near-zero wall time and an abandoned (D3) or killed call both count, and pull the median down.
   - The record does not define "completed" to exclude carried, abandoned or refused calls.

9. LOW. D2. "A nested call under a card-verify line" relies on an inherited slot. Outside every hub card-verify runs unslotted, and verifyrun does not strip `LYM_STRAND_NAME` (only gatecli's tier environment does, at run.go:127).
   - A line such as `cd sub && lym gate test --tier 0 ./x` has no leading `lym gate test` prefix, so the `--wait` requirement (plancontext's `hasFieldPrefix`) does not reach it.
   - Inside card-verify that line takes the capped form, exits 75, and card-verify reports 75 as the line's verdict.

10. LOW. D1. "A finished run reads exactly like today's in-process call" holds for content only.
    - stdout and stderr are replayed as two whole files at the end, so tier output no longer interleaves with the summary lines.
    - A failing test binary whose own exit code is 75 is replayed as 75. The agent reads it as "go on", re-runs, and starts a fresh run, because the result was delivered.

Claims attacked:
- D1, exit 75 meaning "go on" to every prompt: broken (1).
- D1, a finished run replays like in-process: holds for content; ordering and a test's own exit 75 noted (10).
- D1 and D2, join never changes the verdict: broken (4).
- D1, two racing fronts never get a wrong code: broken by the same counterexample (4).
- D2, nested calls stay in-process through the inherited slot: held in a hub, broken outside one for lines without the prefix (9).
- D2, tier environment carries no strand: held (gatecli run.go strips it).
- D3, teardown ends the runner: held; `pairteardown` and fabriccli anchordeps call `EndSparing` on the slug.
- D3, abandoned runner frees its slots within cap plus grace: held.
- D4, 250 s and 59m10s: held arithmetically. The lifetime source is broken for forks (3), mid-run edits and FORCE_PROMPT_CACHING_5M (6), and numbered roles (7).
- D4, the margin validation in `LoadConfig`: broken by the provider seam (5).
- D5, the median estimate: weak (8). Long cap under the 1h Bash ceiling: held.
- D7, the list of agent-facing renders: broken (2).
- D8, Master waits on a 1h role: held for the template map only (an operator map that omits `webster` drops Master to 5m). Low and unlisted.

Tool calls:
- Grepped `EndSparing` callers and `StrandNameEnv` stripping in verifyrun, cardverify and gatecli: my context did not show who ends run processes at a session end or whether card-verify lines keep the strand.
- Read webster-body-implementer.md lines 70–120: my context did not show how implementer forks treat card-verify exit codes and exit 75.
