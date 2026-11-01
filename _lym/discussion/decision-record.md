# Decision record — gate-run-keeps-cache-warm

## Goal

An agent blocked on `lym gate test` gets an answer within its prompt-cache lifetime even when the gate run itself outlasts it, so the agent re-runs the same command and keeps its cache warm.
The keep-warm cap stops being the constant `shuttleengine.DefaultAwaitCap` and becomes config beside the engine's prompt-cache lifetime, per role, for every keep-warm wait.

## Scope

In:

- The detached, joinable gate run (D1 to D3) for `lym gate test` and `lym gate card-verify` (D6).
- The cap config and its resolver (D4), read by the gate's wait and `lym shuttle await`.
- The break-even long wait (D5).
- Gate calls rendered for agents drop `--wait` (D7).
- The prompt and help text that names the 75 answer or a keep-warm Bash timeout (D9).

Out:

- Go verify sites, `gatecli.RunGate` and every `--wait` call: they block to their end in-process, as today.
- The llm driver's `lym shed step --until-stop` wait and the Webster Master's waits (D8).
- `cmd/testtiming/resources.go`, a dev tool that keeps the fallback constant.
- Operator-audience verbs such as `lym webster verify`.

## Decisions

### D1 — A gate call an agent makes runs its tiers in a detached runner it starts or joins

A capped call (D2) does not run its tiers in its own process.
It forms a join key from the verb, the resolved run directory and the argument words as given, and works under a lock in a per-key directory below the worktree's never-tracked gate directory (`gateslot.WaitDir`'s tree).
Each runner gets its own run id and its own subdirectory there, and the key directory names the current run id; a runner writes only into its own subdirectory, so a superseded runner's late writes never reach a newer run's caller.

At start the runner records HEAD's tree and the worktree's dirty paths (`verifytree.DirtyPaths`).
A front joins the current run only when its lease (D3) holds, its result is not yet delivered, and HEAD's tree and the dirty path set equal the recorded ones.
Otherwise it expires the current run's lease, so that runner ends at its next lease read, and starts a fresh runner under a new run id.

The runner is a hidden internal-audience verb under `lym gate` (new) that runs the same request in-process with `--wait` semantics, slot wait and tiers included, with `agentname.StrandNameEnv` removed from its own environment.
It writes the call's stdout and stderr to files in its subdirectory, keeps a phase file naming whether it waits for slots or runs a tier and which, and writes its exit code last and atomically.
The front waits with `shuttleengine.AwaitAny` on the exit file for at most its cap.

- Exit file present: the front copies the recorded stdout, then the recorded stderr, to its own, marks the result delivered and exits with the recorded code, except that a recorded 75, which is then a test's or a line's own code, exits 1 with a stderr line saying so.
- Cap passed: the front prints an ok envelope with `done: false`, the phase file's phase, and while the runner waits for slots `slot_free: false` with each holder's worktree, site and tier read from the pool, and the message to run the same command again in a turn that does nothing else, and exits with `gatecli.SlotBusyExit` (75).

Rationale: the board idea: the same command again joins the run and prints the result once it ended, and agents learn no new verb.
Exit 75 with an ok envelope keeps one meaning, "go on, run the same command again", for the slot wait and the run wait.

Bound: the join compares the tree and the dirty path set, not the dirty files' content, so an edit to an already-dirty file between two calls with the same words gets the run of the earlier content; the gate rule bars such a turn, and the green record still records a pass only over an unchanged clean tree.
The replay gives the same content and exit code as an in-process call, with stdout and stderr no longer interleaved.

### D2 — Which calls take the capped form

A `lym gate test` or `lym gate card-verify` call takes D1's form only when all hold:

- `--wait` is not given;
- it does not run inside an inherited slot (`gateslot.InheritEnv` naming a held slot);
- the caller has a strand, `agentname.StrandNameEnv` set, and the resolver (D4) gives it a non-zero cap.

Every other call runs in-process as today: `--wait`, a nested call under a verify or card-verify line, `gatecli.RunGate`, and a call with no strand, the operator at a terminal.
A call with no strand, or whose resolved cap is zero, also waits for its slots without a cap, which changes today's capped slot wait for the operator.
`lym gate card-verify` removes `agentname.StrandNameEnv` from the environment of the line it runs, as the gate's tier environment already does, so a nested gate call in a line runs in-process inside and outside a hub.

Rationale: only an agent session has a prompt cache to keep warm; a Go site and a nested call must block, and inheritance must keep going in-process so a verify command cannot deadlock.

Bound: the strand is read from the caller's environment, so a forged or absent strand chooses only how long a call blocks, never what runs or its verdict.

### D3 — The runner is bounded by a lease the fronts renew, and is a registered run process

Each front writes the lease into the run's subdirectory when it joins or starts: its own cap deadline plus a grace of 120 s, to cover the agent's next turn.
The runner reads the lease at most once a second from its start, its slot wait included; once the lease has passed, it cancels its call as a terminating signal does today: the tiers' trees are killed, the slots released, and its exit file records the abandonment with exit 1 and a line saying no caller came back.

The runner is spawned detached from the front, its spawn logged at Info, and registered in `internal/runproc` with a new gate role (`runproc.RoleGate` (new)) under the task worktree's slug, so pair teardown and a session end that ends the slug's processes end it.

A terminating signal to a capped front ends the front alone with 128 plus the signal number; the runner goes on until its lease passes.

Rationale: an agent that is interrupted, held or dead stops renewing, and its slots free within one cap plus the grace instead of a whole tier timeout.

Bound: an abandoned runner holds its slots for at most the cap plus the grace after its last front.
A superseded runner (D1) holds its slots for at most one second past the expiry plus its kill time, so two runners of one key may briefly hold slots together.
In the prime and outside every hub the runner is not registered, `lym tender` does not list it, and only its lease and its tiers' `timeout_min` end it; on Windows, where `runproc` signals nothing, only the lease ends it.

### D4 — The keep-warm cap is claude.yaml config beside the prompt-cache lifetime, per role

`claude.yaml` gains `keep_warm_cap`, the hub-wide default, template `250s`, and `keep_warm_cap_roles`, a per-role map with an empty template that replaces the default for a role it names, both `CategoryNone`, beside `prompt_cache_ttl` and `prompt_cache_ttl_roles`.
The claude engine validates them with the lifetimes, at run start and on `lym config claude`: each cap must be positive and below the lifetime its role resolves to, and is refused otherwise naming `lym config claude`.

`shuttleengine.Engine` gains a method that answers a role's cap and long cap (D5), beside `PromptCacheLifetime` (new); a zero cap means the engine keeps nothing warm.
`internal/shuttleengines` gains one resolver (`shuttleengines.KeepWarm` (new)) that takes the config dirs and the caller's strand, takes the strand's role with a numbered suffix (`agentname.MatchesRole`'s base-N form) dropped, and asks the engine `shuttle.yaml`'s `engine` names.
`lym gate test`, `lym gate card-verify` and `lym shuttle await`'s `--cap` default read it; an explicit `--cap` still wins for `lym shuttle await`.
`lym shuttle await` resolves the worktree's config dirs when its working directory lies in a git worktree and needs nothing more, so outside one it still needs no hub.

`shuttleengine.DefaultAwaitCap` stays as the fallback when the config cannot be loaded, the strand does not parse or `lym shuttle await` finds no worktree; the gate logs the fallback at Warn once per call and `lym shuttle await` at Debug.

Rationale: the board puts the cap in config beside the engine's lifetime, with a hub-wide default and per-role values, since each recipe row's agent runs under its own role.
The template keeps 250 s for every role, because a fork's Bash shell carries its session's strand while its own cache runs at Claude Code's subagent lifetime, which the claude engine leaves at its 5-minute default; a role-wide cap above five minutes would leave that role's forks cold.

Bound: a cap set above five minutes for a role leaves its forks' caches cold during a long gate, which costs cache money only; the validation reads the configured lifetimes, so a lifetime changed after a session started or forced by `FORCE_PROMPT_CACHING_5M` is not seen.

### D5 — A gate key whose runs outlast the break-even waits in one long call

`keep_warm_max_reruns`, a new `shuttle.yaml` key, `CategoryNone`, template `10`, is the break-even: a re-run costs a cache read, about a tenth of the context's price, against one cache write at about 1.25 times its price on the 5-minute lifetime.
`gateslot.RunRecord` gains a call id (new field) that every record of one call carries.
For a `--gate <key>` call, the front reads this machine's run records of that key through `gateslot.ReadRunRecords` and takes the median wall time of the latest five calls that ran every tier they listed to an outcome, leaving out a call with a carried or empty-scope tier, an abandoned call and a refused one.
When that median exceeds the cap times `keep_warm_max_reruns`, the front waits with the long cap instead: the claude engine answers it as its one-hour Bash ceiling less five minutes, 55m.
Fewer than three such calls, a call without `--gate`, `keep_warm_max_reruns: 0` or an unreadable record directory keep the short cap.

Rationale: past the break-even a cold return is cheaper than the re-runs; one long call still answers before the Bash tool's ceiling ends it, so the runner and its lease stay the one mechanism and a two-hour gate is about three long calls.

Bound: an estimate from a few earlier calls can mis-pick; a wrong pick costs cache money only, never a verdict.

### D6 — `lym gate card-verify` takes the same capped form

A card-verify call that D2 admits runs its card's line in a D1 runner keyed by the verb and the card id, with its slot taken inside the runner.

Rationale: an implementer fork blocked on a card's `**Verify:**` line is the same wait as a gate call.

### D7 — A gate call rendered into text an agent runs carries no `--wait`

Every gate call rendered into a prompt or note an agent runs, the implementer's card and batch gates and the verify fixers' `fix_gate` among them, is rendered without `--wait`, through a render beside `gateslot.Config.GateCall`.
A gate call Go runs, at a verify site or through `internal/cardverify` from a Go verb, keeps `gateslot.Config.GateCall`'s `--wait` form, and a card's `**Verify:**` line keeps its required `--wait`, since Go re-runs it outside an agent hook.

Rationale: the implementer's batch gate and the cold verify fixer's `webster-fix` gate block an agent on the 5-minute lifetime today.

### D8 — The other agent waits stay as they are

The llm driver's `lym shed step --until-stop` job runs through producers lasting up to hours, past the break-even, with a small context; the Webster Master waits under the `webster` role, which the template gives the 1-hour lifetime.

Bound: a hub whose `prompt_cache_ttl_roles` leaves `webster` out puts the Master on the 5-minute lifetime and its waits go cold.

### D9 — Agents' text names the run going on

- The gate rule in `internal/shellguard/rows.go` tells the agent to run the same command again when `lym gate test` or `lym gate card-verify` answers `done: false`, in place of "no gate slot is free yet".
- The stencils that name exit 75 or `slot_free: false`, the implementer's among them, read 75 with `done: false` from either verb as "run the same command again", never as a failure, and drop the sentence that a `--wait` call is the queued form.
- Text that tells an agent to run a keep-warm wait with a 300000 ms Bash timeout, in `internal/shuttlecli/await.go`'s help and the stencils, says 3600000 ms, so a long cap (D5) or a raised role cap fits.

Rationale: the rule and the prompts must cover the new answer and card-verify's new 75, or a fork counts a 75 as a failed card.

## Constraints

- PATTERN-gate-slot: the runner holds `internal/gateslot` slots as the in-process call does; the green record, carry and verify sites are untouched.
- Every gate call a Go path runs carries `--wait` or runs in an inherited slot, so the strand-based choice (D2) never detaches a Go caller.
- PATTERN-shuttle-provider-seam: the caps reach `internal/gatecli` and `internal/shuttlecli` only through `shuttleengine.Engine` behind `internal/shuttleengines`; the lifetimes, the Bash ceiling and the cap validation stay in the claude engine.
- PATTERN-config-layers: met by D4 and D5's `CategoryNone` keys in degrading modules.
- PATTERN-run-process-owner and PATTERN-spawn-observability: met by D3; tests never re-exec `os.Executable()` under `go test`, so the runner launch is a seam a test replaces.
- PATTERN-daemon-wakeups: the front's wait and the runner's lease read wake at most once a second.
- PATTERN-durable-vs-ephemeral-state and PATTERN-lym-temp-root: the run directories sit under the worktree's never-tracked `.lym` gate tree, named through `internal/lymdirs`.
- PATTERN-tolerant-records: the call id is optional on read, and records without it still total in `lym gate report`.
- PATTERN-cli-cobra: the runner verb sits in the `lym gate` subtree.
- PATTERN-refusal-way-forward: a refused cap names `lym config claude`.

## Auto-mode assumptions

- D1: exit 75 serves both the slot wait and the run wait.
- D2: a caller with no strand blocks to the end.
- D4: the template cap stays 250 s for every role, for forks' sake.
- D5: the break-even counts re-runs; the estimate is the median of five full calls.
- D6: card-verify is in scope.
- D8: the llm driver and Master waits stay.

## Open risks

None.

## Acceptance criteria

- D1: an agent-strand `lym gate test` call whose tiers outlast its cap exits 75 with `done: false` and the running tier while the tiers go on; the same command again after the run ended prints the run's stdout, stderr and exit code; a third call starts a fresh run.
- D1: a call joining a runner still waiting for slots answers `slot_free: false` and names each holder.
- D1: a call with the same words after HEAD's tree or the dirty path set changed starts a fresh run and the superseded runner ends; a late write of the superseded runner never reaches the new call.
- D1: a finished run whose own exit code is 75 is replayed as exit 1.
- D2: `--wait`, an inherited slot, `gatecli.RunGate` and a call with no strand run in-process and block to their end; a call with no strand waits for slots without a cap; a line card-verify runs sees no strand.
- D3: a runner whose lease passes with no front kills its tiers, frees its slots and records the abandonment, in its slot wait as well as in a tier; a terminating signal to a front leaves the runner going; the runner is registered under the task slug with the gate role and pair teardown ends it.
- D4: with the template, every role resolves 250 s; a `keep_warm_cap_roles` entry overrides the default for its role and a numbered strand of that role; a cap at or above its role's lifetime is refused naming `lym config claude`; an unloadable `claude.yaml` falls back to `shuttleengine.DefaultAwaitCap` and the gate call still runs.
- D4: `lym shuttle await` without `--cap` waits the resolved cap inside a worktree and the fallback outside one, and an explicit `--cap` wins.
- D5: a `--gate` key whose median over its latest five full calls exceeds cap times `keep_warm_max_reruns` waits with the long cap; carried, abandoned and refused calls are left out; fewer than three calls, no `--gate` or `keep_warm_max_reruns: 0` keep the short cap; records without a call id still total in `lym gate report`.
- D6: an agent-strand `lym gate card-verify` whose line outlasts the cap exits 75 with `done: false`, and the same command again prints the line's result and exit code.
- D7: the implementer's card and batch gate calls and the verify fixers' `fix_gate` call carry no `--wait`, and Go sites' calls still do.
- D8: the llm driver's stencil is unchanged.
- D9: the gate rule and the implementer stencil read 75 with `done: false` from `lym gate test` and `lym gate card-verify` as "run again"; `lym shuttle await`'s help and the stencils name a 3600000 ms timeout.

## PATTERN changes

- `PATTERN-keep-warm-cap` — adds: Bounding a wait an agent blocks on: the cap from `shuttleengines`' one keep-warm resolver for the caller's role, never a constant bar its fallback; a long gate run goes on in a detached runner the same command joins.

## Notes for the plan writer

- `shuttleengine.Engine`'s `PromptCacheLifetime` has no production caller today, and the fake in `internal/testkit/shuttlefake` answers zero; a fake answering a zero cap makes every test-built caller run in-process under D2.
