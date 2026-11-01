# Support log — gate-run-keeps-cache-warm

## Interview

Autonomous run: no operator answered, and no advisors were named. Each turn below is a question the writer put to itself after exploration, with the auto-pick.

### Turn 1 — Scope

- Read the board entry (T6), the board and `lym loom peers`. The dependency `gate-fewer-binaries-trial` is done and is research only; no peer owns gate waits. `machine-switch` touches process state across machines, noted as an open risk.
- Q: Is a detached runner the right thing to build, or should the gate be split into per-tier calls the agent makes? Auto-pick: detached runner the same command joins, as the board's idea says; per-tier calls change every prompt that names a gate call.
- Q: Which other waits block an agent? Traced `lym shuttle await`, the llm driver's background `lym shed step --until-stop`, the Webster Master's turn-end gate and backgrounded verbs, Go verify sites (all `--wait`), `lym gate card-verify`, and the cold verify fixer's rendered `--wait` gate call. Auto-pick: card-verify and the fixer's render join; the driver and Master stay (D8).

### Turn 2 — Cap source

- Found `claude.yaml`'s `prompt_cache_ttl` and `prompt_cache_ttl_roles` (template: driver 5m, webster 1h, orch 1h) and `shuttleengine.Engine`'s `PromptCacheLifetime`, unused since the parent-review keep-alive pings (commit 414d8b71) went away.
- Q: A separate cap config per role, or derive the cap from the role's lifetime? Auto-pick: derive, lifetime less `keep_warm_margin`; the per-role lifetime already gives a per-recipe, row or role cap and cannot drift from the cap.
- Q: How does a gate call know its caller's role? Auto-pick: the caller's `LYM_STRAND_NAME`, parsed through `internal/agentname`; no strand means no cache to keep warm.

### Turn 3 — Break-even and the Bash ceiling

- The Bash tool's ceiling is one hour (`bashMaxTimeout` in the claude engine), so a two-hour gate cannot block to its end in one call. Auto-pick: past the break-even the front waits with a long cap, the ceiling less the margin, and the same runner and lease serve both.
- Q: Break-even in minutes or in re-runs? Auto-pick: re-runs (`keep_warm_max_reruns`), which scales with the cap.
- Q: Expected length from what? Auto-pick: median wall time of the latest five calls of the same gate key on this machine; run records gain a call id to group a call's tier records.

### Turn 4 — Edge cases

- Abandoned runner (agent interrupted or dead): lease renewed by each front, cap plus grace; runner kills its tiers and frees slots when it passes.
- Nested call under a verify or card-verify line: inherited slot keeps it in-process, so no deadlock and no detach.
- Operator at a terminal: no strand, blocks to the end, Ctrl-C keeps killing the tiers as today.
- Tree edited between re-runs: stated as a bound; the gate rule forbids it and the green record still guards passes.

### Turn 5 — Lens review

Six forks, all reported; findings merged across lenses and judged together.

- existing-paths 1 (BLOCKING) and bound-breaker 3, fork gets the 1-hour cap on a 5-minute cache: fixed; the cap became `claude.yaml` config with a 250 s template for every role, and D4 states the fork bound.
- board-scope 1, cap not config of its own beside the lifetime: fixed by the same D4 rewrite (`keep_warm_cap`, `keep_warm_cap_roles`); `keep_warm_margin` dropped.
- walkthrough 2 and bound-breaker 5, margin validation in shuttle's `LoadConfig` breaks the provider seam: fixed; validation moved into the claude engine with the lifetimes.
- walkthrough 1 and bound-breaker 7, numbered strand roles miss the map: fixed; the resolver drops the base-N suffix.
- bound-breaker 6, mid-run TTL edit and `FORCE_PROMPT_CACHING_5M`: fixed as a stated bound in D4.
- interactions 1, superseded runner's late exit reaches a new call: fixed; per-run id and subdirectory in D1, with the superseded runner's lease expired.
- bound-breaker 4, join across a tree change: fixed; D1 joins only over the same HEAD tree and dirty path set, with the remaining content bound stated.
- bound-breaker 1 and walkthrough 8, card-verify 75 read as a failed card and `slot_free` text: fixed; D9 updates the implementer stencil and gate rule, D1 keeps `slot_free: false` in the slot phase.
- bound-breaker 10 and walkthrough 8, a run's own exit 75 misread as "go on": fixed; replayed as exit 1 (D1).
- bound-breaker 2 and existing-paths 3, D7 missed the implementer's card and batch gates and other agent-facing renders: fixed; D7 is now a rule over every render an agent runs.
- interactions 4, unstated invariant that Go gate calls carry `--wait`: fixed as a Constraint.
- bound-breaker 9, nested line without the `lym gate test` prefix outside a hub: fixed; card-verify strips the strand from its line (D2).
- interactions 2, existing-paths 2 and walkthrough 3, `lym shuttle await` resolves no config: fixed; D4 resolves the worktree's dirs when present, falls back at Debug otherwise.
- interactions 3, walkthrough 7 and bound-breaker 8, carried and abandoned calls in the median: fixed; D5 counts only full calls.
- walkthrough 5, no source for the running tier: fixed; runner phase file (D1).
- walkthrough 6, grace unvalued and slot wait unclear: fixed; 120 s, lease read from the runner's start (D3).
- existing-paths 5 and walkthrough 4, replay loses interleaving: fixed as D1's bound.
- existing-paths 4, prime runner not listed by tender: fixed in D3's bound.
- interactions 5, wrong gate key in D7's rationale: fixed (`webster-fix`).
- interactions 6, Scope wording about a capped slot wait: fixed by the Scope rewrite.
- board-scope 2, `machine-switch` risk loosely stated: fixed by dropping it; D3's lease ends a runner whose agent is held.
- leanness, 59m10s near the ceiling risk: fixed by dropping it; the long cap is now 55m and the template cap 250 s.
- leanness, D8 duplication, Scope, Goal, Constraints and Auto-mode assumptions restating decisions, plan-level notes: fixed; trimmed.
- existing-paths 6, stale `gateslot` doc on `cli_wait_sec`: rejected; a doc line the plan writer finds, not a design point.
- walkthrough scope note, the 11-minute Webster gate is a Go site while Master idles: rejected as a finding; D8 covers it with the 1-hour `webster` lifetime and states the bound for a hub that drops it.

## Rejected alternatives

- Deriving the cap from the role's lifetime less a margin (the first draft): a fork shares its session's strand but runs at the subagent lifetime, so a 1-hour role would give its forks a 59-minute cap on a 5-minute cache; and it left the cap no config of its own, against the board.
- Allowing an agent to background `lym gate test` and await a result file with `lym shuttle await`: needs a change to the gate rule every agent carries and a file contract per call.
- A new exit code for "run goes on": every prompt already re-runs on 75.
- Keep-alive pings to an idle agent while a Go-run gate runs: the agent spends a turn per ping, and the parent-review pings were removed once.
- Capping the llm driver's `lym shed step` wait: producers run up to hours, past the break-even, and the driver's context is small.
- Blocking to the end without re-runs past the break-even: the Bash tool ends a call at one hour, so the long wait is capped at the ceiling instead.

## Review rounds

_No rounds yet._

## Question ledger

- Resolved (auto-pick): build a detached runner joined by the same command (D1).
- Resolved (auto-pick): capped form only for agent-strand calls without `--wait` or an inherited slot (D2).
- Resolved (auto-pick): lease with grace ends an abandoned runner; registered under the task slug with a new gate role (D3).
- Resolved (auto-pick, revised after lens review): cap is `claude.yaml`'s `keep_warm_cap` and `keep_warm_cap_roles`, template 250 s, resolved in `internal/shuttleengines` (D4).
- Resolved (auto-pick): `keep_warm_max_reruns` break-even, median of five full calls, long cap 55m from the claude engine (D5).
- Resolved (auto-pick): card-verify in scope (D6); every agent-facing gate render drops `--wait` (D7).
- Resolved (auto-pick): llm driver and Webster Master waits unchanged (D8).
- Resolved (auto-pick): `machine-switch` needs nothing from this task; D3's lease ends a runner whose agent is held.
