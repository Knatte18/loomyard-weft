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
- The strand removed from the environment of every command Go runs through `verifyrun` (Constraints).

Out:

- Go verify sites, `gatecli.RunGate` and every `--wait` call: they block to their end in-process, as today.
- The llm driver's `lym shed step --until-stop` wait and the Webster Master's waits (D8).
- `cmd/testtiming/resources.go`, a dev tool that keeps the fallback constant.
- Operator-audience verbs such as `lym webster verify`.

## Decisions

### D1 — A gate call an agent makes runs its tiers in a detached runner it starts or joins

A capped call (D2) does not run its tiers in its own process.
It forms a join key from the verb, the resolved run directory and, for `lym gate test`, the argument words as given (D6 keys `lym gate card-verify`), and works under a lock in a per-key directory below the worktree's never-tracked gate directory (`gateslot.WaitDir`'s tree).
Each runner gets its own run id and its own subdirectory there, and the key directory names the current run id; a runner writes only into its own subdirectory, so a superseded runner's late writes never reach a newer run's caller.

At start the runner claims its subdirectory by creating a process file exclusively, holding its pid and a start reading that tells a reused pid apart on every platform, `proc.StartTime` on Linux and `proc.CreationTime` on Windows, and records HEAD's tree and the worktree's dirty paths (`verifytree.DirtyPaths`).
A front's cap counts from the front's own start, so its cap deadline, to which D3 adds the grace, is that start plus the cap.
A front waits for the key directory's lock only while at least 30 s of its cap remain, so a claim wait fits inside the cap, and otherwise answers as at a passed cap, below.
The front that starts a run holds the lock until the runner's process file exists, checking at most once a second for at most 30 s, so no other front sees a run without one.
A runner that ends before it claims, or has not claimed by then, is a start failure: the front creates the process file itself by the same exclusive create, naming its own pid, expires the run's lease and exits 1 with the runner's stderr.
The two exclusive creates order the claim against the start failure: a runner whose create comes second refuses, and a front whose create fails finds the runner claimed in time and goes on as for any claim.
A front joins the current run only when its lease (D3) holds, its result is not yet delivered, HEAD's tree and the dirty path set equal the recorded ones, and the run either has an exit file without the abandoned mark (D3) or its process file names a live process whose start reading, taken with the same reader, equals the recorded one.
Otherwise it expires the current run's lease, so a live runner ends at its next lease read, kills a dead card-verify runner's recorded line group (D6), and starts a fresh runner under a new run id.

The runner is a hidden internal-audience verb under `lym gate` (new), with `agentname.StrandNameEnv` removed from its own environment.
For `lym gate test` it runs the same request in-process with `--wait` semantics, slot wait and tiers included; for `lym gate card-verify` it runs the card's line as D6 states.
It refuses, before it runs anything or writes an exit file, when its subdirectory holds no lease, a passed lease or a process file, naming `lym gate test` and `lym gate card-verify` as the way forward.
It writes the call's stdout and stderr to files in its subdirectory, keeps a phase file naming whether it waits for slots or runs a tier and which, and writes its exit code last and atomically.
The front waits with `shuttleengine.AwaitAny` on the exit file for what remains of its cap.

- Exit file present with the abandoned mark (D3): the front answers as at a passed cap, below, and the same command again joins or starts the current run.
- Exit file present without it: the front copies the recorded stdout, then the recorded stderr, to its own, marks the result delivered and exits with the recorded code, except that a recorded 75, which is then a test's or a line's own code, exits 1 with a stderr line saying so.
- Cap passed: the front prints an ok envelope with `done: false`, the phase file's phase or, where the run has none yet, the phase `starting`, and while the runner waits for slots `slot_free: false` with each holder's worktree, site and tier read from the pool, and the message to run the same command again in a turn that does nothing else, and exits with `gatecli.SlotBusyExit` (75).

Rationale: the board idea: the same command again joins the run and prints the result once it ended, and agents learn no new verb.
Exit 75 with an ok envelope keeps one meaning, "go on, run the same command again", for the slot wait and the run wait.

Bound: the join compares the tree and the dirty path set, not the dirty files' content, so an edit to an already-dirty file between two calls with the same words gets the run of the earlier content; the gate rule bars such a turn, and the green record still records a pass only over an unchanged clean tree.
The replay gives the same content and exit code as an in-process call, with stdout and stderr no longer interleaved, except that a recorded 75 exits 1 with one added stderr line.
A second front with the same words that comes back after another front took the result starts a fresh run, which costs one more gate run and never a verdict.
A direct call of the runner verb reaches only a subdirectory a front made, and one its runner already claimed refuses it; a lease written by hand lets a run start, and that run ends at its lease like any other, so a direct call adds at most a detached run bounded by the lease it found.

### D2 — Which calls take the capped form

A `lym gate test` or `lym gate card-verify` call takes D1's form only when all hold:

- `--wait` is not given;
- it does not run inside an inherited slot (`gateslot.InheritEnv` naming a held slot);
- the caller has a strand, `agentname.StrandNameEnv` set, and the resolver (D4) gives it a non-zero cap.

Every other call runs in-process as today: `--wait`, a nested call under a verify or card-verify line, `gatecli.RunGate`, and a call with no strand, the operator at a terminal.
A call with no strand, or whose resolved cap is zero, also waits for its slots without a cap, which changes today's capped slot wait for the operator.
`cardverify.Runner` removes `agentname.StrandNameEnv` from the environment of every line it runs, under `lym gate card-verify` and a Go verb alike, as the gate's tier environment already does, so a nested gate call in a line runs in-process inside and outside a hub.

Rationale: only an agent session has a prompt cache to keep warm; a Go site and a nested call must block, and inheritance must keep going in-process so a verify command cannot deadlock.

Bound: the strand is read from the caller's environment, so a forged or absent strand chooses only how long a call blocks, never what runs or its verdict.

### D3 — The runner is bounded by a lease the fronts renew, and is a registered run process

Each front writes the lease into the run's subdirectory, under the key directory's lock, when it joins or starts: its own cap deadline plus a grace of 120 s, to cover the agent's next turn.
It writes its deadline only when that is later than the lease already there, so the lease is the latest deadline any front wrote; the supersede expiry and the start-failure expiry (D1) are the writes that move it earlier.
A front that starts a run writes the lease before it spawns the runner, so the runner's first read finds it.
The runner reads the lease at most once a second from its start, its slot wait included; once the lease has passed, it cancels its call as a terminating signal does today: the tiers' trees are killed, the slots released, and its exit file records exit 1, a line saying the run was abandoned, and the abandoned mark.
A terminating signal to the runner cancels its call the same way, and its exit file records 128 plus the signal number and the abandoned mark.
A front never replays a marked exit file (D1), so a run that a lease, a supersede or a signal ended reaches no caller as a verdict.
A runner killed without writing its exit file, by SIGKILL, a crash or an OOM kill, is found gone by the next front, which supersedes it (D1).
A `lym gate test` runner's slots free with its OS file locks, and each tier's own process gets its `proc.StartTied` parent-death SIGKILL; a card-verify runner's line ends as D6 states.

The runner is spawned detached from the front, its spawn logged at Info, and registered in `internal/runproc` with a new gate role (`runproc.RoleGate` (new)) under the task worktree's slug, so pair teardown and a session end that ends the slug's processes end it.

A terminating signal to a capped front ends the front alone with 128 plus the signal number; the runner goes on until its lease passes.

Rationale: an agent that is interrupted, held or dead stops renewing, and its slots free within the latest front's cap plus the grace instead of a whole tier timeout.

Bound: an abandoned runner holds its slots until the latest deadline a front wrote, at most the long cap (D5) plus the grace after the front that wrote it.
A runner killed without an exit file costs a front already waiting on it at most one cap, after which it answers 75 and its next call starts afresh.
On Linux a parent-death SIGKILL reaches a tier's own process and never its descendants, so a runner or a `--wait` gate child killed without an exit file leaves a tier's descendants, such as the test binaries `go test` started, running to their own end without a slot.
A superseded runner (D1) holds its slots for at most one second past the expiry plus its kill time, so two runners of one key may briefly hold slots together.
In the prime and outside every hub the runner is not registered, `lym tender` does not list it, and only its lease and its tiers' `timeout_min` end it; on Windows, where `runproc` signals nothing, only the lease ends it, and the join reads the runner's liveness through `proc.CreationTime` (D1).

### D4 — The keep-warm cap is claude.yaml config beside the prompt-cache lifetime, per role

`claude.yaml` gains `keep_warm_cap`, the hub-wide default, template `250s`, and `keep_warm_cap_roles`, a per-role map with an empty template that replaces the default for a role it names, both `CategoryNone`, beside `prompt_cache_ttl` and `prompt_cache_ttl_roles`.
The claude engine validates them with the lifetimes, at run start and on `lym config claude`: each cap must be at least 60 s, so a front's lock and claim waits (D1) fit inside it, below the lifetime its role resolves to and at most the long cap (D5), and is refused otherwise naming `lym config claude`.

`shuttleengine.Engine` gains a method that answers a role's cap and long cap (D5), beside `PromptCacheLifetime` (new); a zero cap means the engine keeps nothing warm.
`internal/shuttleengines` gains one resolver (`shuttleengines.KeepWarm` (new)) that takes the config dirs and the caller's strand, takes the strand's role with a numbered suffix (`agentname.MatchesRole`'s base-N form) dropped, and asks the engine `shuttle.yaml`'s `engine` names.
`lym gate test`, `lym gate card-verify` and `lym shuttle await`'s `--cap` default read it; an explicit `--cap` still wins for `lym shuttle await`.
`lym shuttle await` resolves the worktree's config dirs when its working directory lies in a git worktree and needs nothing more, so outside one it still needs no hub.

`shuttleengine.DefaultAwaitCap` stays as the fallback when the config cannot be loaded, the strand does not parse or `lym shuttle await` finds no worktree; the gate logs the fallback at Warn once per call and `lym shuttle await` at Debug.

Rationale: the board puts the cap in config beside the engine's lifetime, with a hub-wide default and per-role values, since each recipe row's agent runs under its own role.
The template keeps 250 s for every role, because a fork's Bash shell carries its session's strand while its own cache runs at Claude Code's subagent lifetime, which the claude engine leaves at its 5-minute default; a role-wide cap above five minutes would leave that role's forks cold.
Evidence for the fork lifetime: the settings text built into Claude Code 2.1.316, the version this hub runs, describes the lifetime for subagents, workflows, background and helper requests as "Unset = automatic (5 minutes unless ENABLE_PROMPT_CACHING_1H=1)", with `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL` taking precedence; the claude engine sets neither (`internal/shuttleengine/claudeengine/doc.go`), and its `CLAUDE_CODE_PROMPT_CACHE_TTL` sets the main session's lifetime only.
An operator environment that sets `ENABLE_PROMPT_CACHING_1H` gives forks the 1-hour lifetime, under which 250 s is still warm.

Bound: a cap set above five minutes for a role leaves its forks' caches cold during a long gate, which costs cache money only; the validation reads the configured lifetimes, so a lifetime changed after a session started or forced by `FORCE_PROMPT_CACHING_5M` is not seen.

### D5 — A gate key whose runs outlast the break-even waits in one long call

`keep_warm_max_reruns`, a new `shuttle.yaml` key, `CategoryNone`, template `10`, is the break-even: a re-run costs a cache read, about a tenth of the context's price, against one cache write at about 1.25 times its price on the 5-minute lifetime.
`gateslot.RunRecord` gains a call id (new field) that every record of one call carries.
Today a record carries no mark of a cancelled call: `execution.report` (`internal/gatecli/run.go`) writes a tier the cancel cut with its exit code, so a call cancelled in its last listed tier has a record for every listed tier.
`gateslot.RunRecord` therefore gains `Abandoned` (new field), which `execution.writeRecord`, the path every record of a call goes through, the build step's and a blocked tier's included, sets on a record it writes after the call's context was cancelled, whatever cancelled it: a lease, a supersede or a terminating signal.
`lym gate report` counts records carrying `Abandoned` apart, as it counts the build step's runs, with their own run count and wait and wall totals, so its other counts and totals, the launch totals among them, cover runs that ended to an outcome while the slot time cut runs spent stays visible.
A call's wall time is the span over the records sharing its call id from the earliest start, a record's `WaitStart` or its `Start` where `WaitStart` is zero as on a build step's record (`internal/gatecli/build.go`), to the latest `End`; it covers the slot waits, the build and the tiers, and leaves out the pre-tier phase: the config load, the scope computation and the guard scan.
For a `--gate <key>` call, the front reads this machine's run records of that key through `gateslot.ReadRunRecords` and takes the median wall time of the latest five calls that ran every tier they listed to an outcome, leaving out a call with a carried or empty-scope tier and a call any of whose records carries `Abandoned`.
A refused call, and a call cancelled in a slot wait before its last listed tier, lack a record for a listed tier and drop out by themselves.
When that median exceeds the cap times `keep_warm_max_reruns`, the front waits with the long cap instead: the claude engine answers it as its one-hour Bash ceiling less five minutes, 55m.
Fewer than three such calls, a call without `--gate`, `keep_warm_max_reruns: 0` or an unreadable record directory keep the short cap.

Rationale: past the break-even a cold return is cheaper than the re-runs; one long call still answers before the Bash tool's ceiling ends it, so the runner and its lease stay the one mechanism and a two-hour gate is about three long calls.

Bound: an estimate from a few earlier calls can mis-pick; a wrong pick costs cache money only, never a verdict.

### D6 — `lym gate card-verify` takes the same capped form

A card-verify call that D2 admits runs its card's line in a D1 runner keyed by the verb, the resolved run directory and the matched card's own id (`card.ID()`), so every accepted form of one card, `4` and `4-<slug>` alike, joins one run.
The runner runs the line through `cardverify.Runner`, which runs a line by its kind (`internal/cardverify/runner.go`):

- A `--wait` gate line, the form D7 keeps for a card's gate call, runs unslotted as a child process that takes its own slots and runs its tiers in-process; the runner holds no slot, and its phase file names the line as running, with no `slot_free` answer, since the slot wait happens inside the child.
- Any other line takes one slot inside the runner; its phase file names the slot wait, with `slot_free: false` and the holders as D1 states, and then the line as running.

`cardverify.Runner` takes the caller's context in place of its own `context.Background()`, for the slot acquire and the line, so the lease cancel (D3) reaches both.
A cancelled line's process group gets SIGTERM and then SIGKILL through `verifyrun.Run`'s group terminate, and a `--wait` gate child cancels on that SIGTERM as a terminating signal does today, killing its tiers and releasing its slots.

The runner ties each line to itself, so a runner killed without an exit file (D3) leaves no line running past the next front.
On Linux the line's shell gets a parent-death SIGKILL, as `proc.StartTied` gives a tier, and the runner records the line's process group (`proc.TreeRecord`) in its subdirectory; a front that supersedes a dead runner (D1) kills that group through `proc.KillTree`, which reaches the shell's descendants the parent-death signal misses, a `--wait` gate child among them, whose own tiers get their `proc.StartTied` parent-death SIGKILL with the bound D3 states.
On Windows the runner holds a kill-on-close job (`proc.HoldNamedJob`) that the line's tree inherits, so the runner's death ends the whole tree.

Bound: on Linux a dead runner's line runs on until a front supersedes the runner, or to its own end when no front comes, holding the slots its gate child took or running without the slot the runner held.

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
- Text that names a 300000 ms Bash timeout for a keep-warm wait, in `internal/shuttlecli/await.go`'s help and the `shuttleengine.DefaultAwaitCap` doc comment, says 3600000 ms, so a long cap (D5) or a raised role cap fits.

Rationale: the rule and the prompts must cover the new answer and card-verify's new 75, or a fork counts a 75 as a failed card.

## Constraints

- PATTERN-gate-slot: the runner holds `internal/gateslot` slots as the in-process call does; the green record, carry and verify sites are untouched.
- Every gate call a Go path runs carries `--wait` or runs in an inherited slot, so the strand-based choice (D2) never detaches a Go caller.
  An operator-written command Go runs, such as a landing config's deploy command or a batten recipe row's command, may hold a gate call without `--wait`, so every `verifyrun.Run` caller removes `agentname.StrandNameEnv` from the command's environment, as `verifytree` does: `cardverify.Runner` by D2, and `internal/battencli/land.go`'s `runDeploy` and `runRowCommand` (new), which today pass a nil environment and so inherit the strand.
- PATTERN-shuttle-provider-seam: the caps reach `internal/gatecli` and `internal/shuttlecli` only through `shuttleengine.Engine` behind `internal/shuttleengines`; the lifetimes, the Bash ceiling and the cap validation stay in the claude engine.
- PATTERN-config-layers: met by D4 and D5's `CategoryNone` keys in degrading modules.
- PATTERN-run-process-owner and PATTERN-spawn-observability: met by D3; tests never re-exec `os.Executable()` under `go test`, so the runner launch is a seam a test replaces.
- PATTERN-daemon-wakeups: the front's wait, its lock wait, its claim wait and the runner's lease read wake at most once a second.
- PATTERN-durable-vs-ephemeral-state and PATTERN-lym-temp-root: the run directories sit under the worktree's never-tracked `.lym` gate tree, named through `internal/lymdirs`.
- PATTERN-tolerant-records: the call id and `Abandoned` are optional on read, and records without them still total in `lym gate report`; records carrying `Abandoned` are counted apart (D5).
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

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/discussion/round-2-review.md; fixer report: _lym/reviews/discussion/round-2-fixer-report.md.
  - cap-excludes-claim-and-lock-waits (class design, severity MEDIUM, open in rounds 2)
  - card-verify-killed-runner-orphans-line (class design, severity MEDIUM, open in rounds 2)
  - windows-join-start-time (class design, severity BLOCKING, open in rounds 2)
<!-- /lym:carry-over Discussion-Review -->

## Acceptance criteria

- D1: an agent-strand `lym gate test` call whose tiers outlast its cap exits 75 with `done: false` and the running tier while the tiers go on; the same command again after the run ended prints the run's stdout, stderr and exit code; a third call starts a fresh run.
- D1: a call joining a runner still waiting for slots answers `slot_free: false` and names each holder.
- D1: a call with the same words after HEAD's tree or the dirty path set changed starts a fresh run and the superseded runner ends; a late write of the superseded runner never reaches the new call.
- D1: a finished run whose own exit code is 75 is replayed as exit 1.
- D1: a call with the same words after its runner was killed without an exit file starts a fresh run.
- D1: a direct call of the runner verb on a subdirectory with no lease, a passed lease or a claimed process file refuses before it runs anything, naming `lym gate test` and `lym gate card-verify`.
- D1: a starting front whose runner has not claimed within 30 s exits 1 and expires the lease, and a runner that claims after the front's own exclusive create refuses.
- D1: a starting front whose runner claims late still answers within its cap, and a front that waits for the key lock until less than 30 s of its cap remain answers 75 with `done: false`.
- D1: on Windows, a front joins a live runner and supersedes one whose pid was reused.
- D2: `--wait`, an inherited slot, `gatecli.RunGate` and a call with no strand run in-process and block to their end; a call with no strand waits for slots without a cap; a line card-verify runs sees no strand.
- D3: a runner whose lease passes with no front kills its tiers, frees its slots and records the abandonment, in its slot wait as well as in a tier; a terminating signal to a front leaves the runner going; a terminating signal to the runner records 128 plus the signal number with the abandoned mark; a front that finds a marked exit file, from a lease, a supersede or a signal, answers 75 with `done: false` and replays nothing; a short-cap front joining a long-cap front's run leaves the run going until the long-cap front's deadline; the runner is registered under the task slug with the gate role and pair teardown ends it.
- D4: with the template, every role resolves 250 s; a `keep_warm_cap_roles` entry overrides the default for its role and a numbered strand of that role; a cap below 60 s or at or above its role's lifetime is refused naming `lym config claude`; an unloadable `claude.yaml` falls back to `shuttleengine.DefaultAwaitCap` and the gate call still runs.
- D4: `lym shuttle await` without `--cap` waits the resolved cap inside a worktree and the fallback outside one, and an explicit `--cap` wins.
- D5: a `--gate` key whose median over its latest five full calls exceeds cap times `keep_warm_max_reruns` waits with the long cap; carried, abandoned and refused calls are left out, a call cancelled in its last listed tier or in the build step before it included; a call's wall time spans its earliest start to its latest `End`, a build step's record counting from its `Start`; fewer than three calls, no `--gate` or `keep_warm_max_reruns: 0` keep the short cap; records without a call id or `Abandoned` still total in `lym gate report`, and records carrying `Abandoned` are counted apart from its other totals.
- D6: an agent-strand `lym gate card-verify` whose line outlasts the cap exits 75 with `done: false`, and the same command again prints the line's result and exit code; a lease that passes while a `--wait` gate line runs ends the line's process group and frees its slots; a front that supersedes a SIGKILLed card-verify runner kills its line's process group; `4` and `4-<slug>` join one run.
- D7: the implementer's card and batch gate calls and the verify fixers' `fix_gate` call carry no `--wait`, and Go sites' calls still do.
- D8: the llm driver's stencil is unchanged.
- D9: the gate rule and the implementer stencil read 75 with `done: false` from `lym gate test` and `lym gate card-verify` as "run again"; `lym shuttle await`'s help and the `shuttleengine.DefaultAwaitCap` doc name a 3600000 ms timeout.

## PATTERN changes

- `PATTERN-keep-warm-cap` — adds: Bounding a wait an agent blocks on: the cap from `shuttleengines`' one keep-warm resolver for the caller's role, never a constant bar its fallback; a long gate run goes on in a detached runner the same command joins.

## Notes for the plan writer

- `shuttleengine.Engine`'s `PromptCacheLifetime` has no production caller today, and the fake in `internal/testkit/shuttlefake` answers zero; a fake answering a zero cap makes every test-built caller run in-process under D2.
