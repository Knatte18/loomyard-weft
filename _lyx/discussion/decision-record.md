# Decision record — test-tiers

## Goal

Every test run in a hub, an agent's own and every verify gate's, goes through `lyx gate test`, which runs numbered tiers the hub declares in `gate.yaml` over a scope lyx computes, takes the gate slots each tier asks for, enforces each tier's timeout and test-count floor, and logs every tier run where it survives teardown.
`lyx gate test` stops being a `go test` wrapper: lyx hard-codes no test command, and a C# hub such as NORCE Models gets the same slotted, bounded, observable gate as this Go hub.

## Scope

In:

- `internal/gateslot` and `internal/gatecli`: tier config, the tier-running verb, per-tier slots and the drain, counters and floors, the per-run log.
- `internal/impactset` as the `go` scoper.
- Every verify site D8 lists, the card `**Verify:**` runner, and the stencils and specs that render gates or `verify_tiers`.
- The retirements in D9 and the shell guard row in D10.
- `CLAUDE.md`'s push rule (D12).

Out:

- Test selection for C# (note `dotnet-scoped-verify`): a C# hub uses the `none` scoper.
- Asynchronous tiers in shed and the baseline capture before implementation (note `regression-baseline-verify`); the NORCE Models baseline tier is declared and run by hand only.
- `lyx gate status` (note `gate-status`); D6 only adds the tier to the holder record it will read.
- A pause holding a gate between tiers or while it waits for slots: task `tender-pause-holds-reprompt` owns pause semantics for verify gates.
- From note `dotnet-verify-protocol`'s Required 3, the bullet listing each verify run's wall time in the change description stays in that note: Describe runs before the landing gates, so it cannot report them; D11's log carries every wall time.
- Exclusivity for timing runs (that note's Wanted 5) stays there; its later design reuses `slots: all`.

## Decisions

### D1 — Tiers are declared in the hub-wide `gate.yaml`

`gate.yaml` gains an open map `tiers` keyed `tier0`, `tier1`, ..., each tier holding:

- `about`: one line saying what the tier tests, rendered where `verify_tiers` was (D9).
- `command`: one shell command, with the placeholders `{targets}`, `{parallel}` and `{args}` (the words after `--`).
  A command without `{targets}` always runs whole; a call naming a narrower scope for it runs it whole and says so.
  Words after `--` for a tier whose command lacks `{args}` are refused, naming the tier and `lyx config gate`.
- `dir`: the run directory relative to the worktree root, default `.`.
- `env`: environment variables for the command; values may hold `{parallel}`.
- `all_targets`: the `{targets}` value of a whole-scope run.
- `parallel`: at least 1.
- `timeout_min`: at least 1; no tier is unbounded.
- `min_tests`: at least 1, the floor of an unfiltered whole run (D7).
- `counter`: `go-json` or `vstest` (D7).
- `covers`: lower tier numbers this tier's command runs anyway.
- `slots`: at least 1, or `all`; default 1.
- `build_tags`: the Go build tags the command compiles under, read only by the `go` scoper (D7).
- `prebuilt_lyx`: default false; true runs today's tagged-run `lyx` prebuild for the tier, at the tier's `parallel`, inside the tier's slots and timeout, only where the worktree root holds `cmd/lyx`.

Top-level keys beside `slots` and `busy_quit_sec`:

- `gates`: gate name to a list of tier numbers.
  The Go sites read `card`, `batch`, `round`, `final`, `landing` and `publish`; other names, such as loomyard's `push`, are the operator's own.
- `scoper`: `go` or `none`.
- `guard_tier`: the tier whose command runs the `//lyx:guard` tests (D5), read only under `go`.
- `refuse_raw`: command prefixes the shell guard refuses (D10), each one or more words of letters, digits, `.`, `_` and `-`.

Load validates each tier's own keys, the key floors, the counter and scoper names, `covers` entries lower than their tier, and the `refuse_raw` word shape, refusing with the key and `lyx config gate`.
Cross-references are checked at use, never at load: a call or site whose tier list, `covers` or `guard_tier` names a tier the hub does not declare, or that finds no `tier0`, is refused naming `lyx config gate`.
So the template, a hub mid-way through a key-by-key `lyx config gate` set, and every slot-only reader of the config still load.

The template declares no tier, `card`, `batch`, `round`, `final` and `landing` as `[0]`, `publish` as `[]`, `scoper: none`, `guard_tier: 0` and an empty `refuse_raw`.
Only `publish` may be empty: an empty list for any other Go-read gate is refused at use naming `lyx config gate`, so no gate verifies nothing silently.
`slots` and each tier's `parallel` take this machine's override; every other key is hub-file only, since a tier command runs any shell command in a task worktree.
`go_parallel` becomes a retired key in the hub and machine layers.

Rationale: `gate.yaml` is already hub-wide with a machine layer, and parallelism was its one machine key; one module owns what runs in a slot.

### D2 — `lyx gate test` runs a tier list over a scope

`lyx gate test [-C <dir>] [--tier N]... [--card <id>]... [--base <rev>] [--all] [--guards] [--wait] [<targets>...] [-- <args>]`:

- The tier list is the `--tier` values, default `[0]`.
  Tiers run ascending, one at a time, and the call stops at the first failing tier with its exit code.
  A listed tier another listed tier `covers` is skipped and named as covered.
- Scope, exactly one form: computed against the parent (default, D3), `--base <rev>`, `--card <id>` (D4), `--all`, or explicit targets passed verbatim as `{targets}`.
- `-C <dir>` sets the run directory for explicit targets and overrides the tier's `dir`; without it, the tier's `dir` applies.
- `--tags` is retired: refused before any slot, naming `--tier`.
- `--guards` keeps its meaning under `go` and is refused under `none`.
- `--wait` lifts the keep-cache-warm cap on the slot wait; every Go verify site passes it, so a busy hub never fails a gate.
- A call in a hub with no tier, or outside every hub with no worktree-layer `gate.yaml`, is refused naming `lyx config gate`; outside every hub a standalone worktree's own `gate.yaml` is read and the run is unslotted, as today.
- A tier's `timeout_min` covers all its invocations (D3's per-module runs and D5's guard step included) and starts once its slots are held; a tier that outlives it has its process tree killed and fails with a timeout line.
- The work before the first tier (scope computation, plan read, guard scan) is bounded by a fixed 10-minute ceiling, failing with a timeout line, so a Go site's call has no unbounded step other than the slot wait.
- One line per tier run goes to stderr: tier, scope form, counts, wall seconds, exit code and log path.
- The CPU watchdog and the environment stripping apply to each tier command.

Bound: `--base`, `--card` and explicit targets only change which targets a tier runs over, never which tiers run or the slot rules; `--all` only widens.
`--wait` lets any caller block one call for as long as the hub is busy; an agent that passes it only loses its own prompt cache.
Rationale: the board's operator decisions ask for tiers, computed scope with `--tier` and `--card`, and explicit targets as the exception.

### D3 — Computed scope

The scoper maps the diff from a base to the working tree, tracked changes and untracked files that are not ignored, onto `{targets}`.
The default base is the merge base of HEAD with the parent branch fabric's origin record names; with no parent, such as the prime on `main` or a standalone run, the run is whole.

- `none`: every run is whole; `{targets}` is the tier's `all_targets`.
- `go`: `internal/impactset` derives the impacted package set as today, as targets per module rather than a command, and falls back to whole wherever it falls back today.
  A tier then runs once per module, in that module's directory, with module-relative targets, under one slot hold.
  A diff touching only `.md` files is an empty set: every tier is skipped with nothing to test (D7), and only D5's guard step runs.

Rationale: card gates and an agent's own checks run before the commit, so the gate tests the working tree.
Bound: the `go` scoper narrows exactly where impactset narrows today; what its import graph cannot see is caught by the whole runs at `final` and `landing` (D8).

### D4 — Card scope

`--card <id>` reads the plan through `internal/planparser` and takes the union of the named cards' targets.
Under `go`, those become the cards' surviving package directories per module, the rule Webster's card gate applies today, plus D5's guard step; under `none` the run is whole.
An unknown card id is refused before any slot, naming the plan's card ids; a worktree with no plan is refused, naming `--all` or explicit targets.

Rationale: a card's gate keeps pointing at exactly the card that broke, as the operator decided for note `loom-gates-xdr-stacking`.

### D5 — Guard tests under the `go` scoper

A computed or card scope also runs the module's `//lyx:guard` tests outside the scope, by name in their own packages, through `guard_tier`'s command with the guard packages as `{targets}` and `-run '^(<names>)$'` as `{args}`, once per call after the listed tiers pass.
It is skipped when the call ran `guard_tier`, or a tier covering it, whole.
A `guard_tier` command lacking `{targets}` or `{args}` is refused at use, naming `lyx config gate`.
A guard scan error refuses the call with the file, line and way forward `--guards` gives today.

Rationale: the card and round gates run the guard tests today; the scoper keeps that coverage without a site spelling a guard command.
Cost: the guard step runs `guard_tier`'s whole command over the guard packages, its build and vet steps included.

### D6 — Per-tier slots and the drain

Each tier acquires its slots, runs and releases them before the next tier.
`slots: 1` takes the first free slot, as today.
A tier asking for more, or `all` (the current `slots` count; a larger number is clamped to it), first takes the pool's drain lock, then takes its slots one at a time in ascending order, holding what it has while it waits.
While another process holds the drain lock, a single-slot acquire waits.
A run inside an inherited slot (`gateslot.InheritEnv`) acquires nothing, whatever its tier asks for.
`Lease.Env` carries only the inheritance variable: parallelism and Go flags come from the tier's `command` and `env`.
The holder record gains the tier, and the gate writes its wait record whenever it waits; loom status reads the wait record ahead of the verify marker, so a waiting gate shows as waiting.

Bound: the drain lets one heavy tier at a time wait ahead of new single-slot acquires, and the holders it waits for are bounded by their own tier timeouts.
An inherited run, such as a test that calls `lyx gate test` inside the outer gate's tier, is not sequenced by its own `slots`, and runs inside its parent's.
Rationale: ordered acquisition prevents two multi-slot waiters from deadlocking, and the drain prevents starvation.

### D7 — Counters, floors and the zero-test failure

- `go-json`: the command emits `go test -json`; lyx renders the events' output text to the terminal in order, passes non-JSON lines through verbatim, and reports discovered, passed, failed and skipped counts.
- `vstest`: lyx passes output through and sums the fields of every VSTest summary line into the same counts.

Every tier run that executes and discovers zero tests fails, whole or narrower, and so does a run with words after `--` that discovers zero.
A run is whole when its scope is whole and it carries no words after `--`; a whole run that exits zero but discovers fewer than `min_tests` fails.

A scope that legitimately holds nothing to test is met before the run, never by passing an empty one:
the scoper drops every target that holds no tests for the tier, and when no target is left the tier is reported as skipped with nothing to test, an outcome distinct from pass and failure, printed and logged as such (D11).
A skipped tier runs no command and does not fail the call.
Under `go` a package holds tests for a tier when a `_test.go` file the tier's `build_tags` compiles declares a `Test`, `Benchmark`, `Fuzz` or `Example` function; `TestMain` alone counts as none.
`build_tags` is a tier key read only by the `go` scoper, the tags the tier's command compiles under, empty for untagged.
Under `none` every run is whole, so nothing is dropped and the floor applies.

Rationale: the board's operator decision is that zero discovered tests is a failure, since a run that discovers nothing and exits green passes vacuously; a target with nothing to test is not a run that discovered zero, so it is left out rather than run.

### D8 — Every verify site renders its gate's tier list

Each Go site runs `lyx gate test --wait` with its gate's list as `--tier` flags and its scope form, through `verifytree.Verify` as today; the rendered command is the verified-tree record's key.
A Go-run site reads `gate.yaml` at call time; the card and batch gates are rendered into the fork prompt at begin-batch and recover-batch.

| Site | Gate | Scope form |
|---|---|---|
| Webster card gate (fork prompt) | `card` | `--card <id>` |
| Webster batch gate (fork prompt) | `batch` | `--card` per card of the batch |
| Webster plan-level verify gate, `lyx webster verify` | `final` | `--all` |
| Webster-Burler-Fix, Merge-Burler-Fix | `round` | `--base` at the base command's last pass, falling back to the base command where there is none |
| Darn-Write, Darn-Burler-Fix | `round` | `--base` at the merge base, unless the diff is tree-scoped (D9) |
| Merge-In, catch-up merge-ins, Finalize | `landing` | `--all` |
| Publish | `landing`, then `publish` | `--all` |

- `final` and `landing` run whole, keeping the operator decision in note `loom-gates-xdr-stacking` that Webster's plan-level gate runs the whole suite.
- The round's base command is loom's `final` command and darn's `landing` command, the commands whose passes the round narrows from.
- Two sites with the same list and form render the same command, so Merge-In skips on a tree Webster's gate passed when `final` and `landing` hold the same list.
- An empty `publish` list takes the path of today's unset `publish_verify`.
- Card and batch gates keep `lyx loom lint-comments` and `lyx loom lint-pattern` after the gate call; the round keeps its lints in Go.
- The Publish failure record names the failing tier, and the round's confirm-the-fix pass runs that tier for the record's failing tests and packages.
  A record written before the deploy with kind `plan_verify` reads as the `landing` list, and `publish_verify` as the `publish` list.
- `verifytree.Verify` and the card `**Verify:**` runner no longer take a gate slot; every command they run is a `lyx gate test` call or is bounded as D9 and D10 say.
  `verifytree`'s 60-minute timeout no longer applies to a gate command; it still applies to darn's tree-scoped command.

Bound: a changed tier command or `about` does not invalidate a recorded pass on an unchanged tree, since the record keys on the rendered command; a changed tier list does.
Rationale: no site or agent spells a test command or a package list.

### D9 — Retired config and plan inputs

- The plan's `## verify:` section is no gate input.
  The parser keeps accepting it, so an in-flight plan still parses; the plan gate reports a present section as an informational finding, and the plan-level checks on it (missing package arguments, cross-module commands) are retired.
  The plan spec's verify model, the plan template and the Plan-Review rubric drop it; a card's `**Verify:**` stays the optional escape hatch.
- `landing.yaml`'s `publish_verify` is retired; the `publish` gate replaces it, and darn's skip seam skips the `publish` gate.
- `darn.yaml`'s `verify` is retired; darn's sites run their gate lists.
  `tree_scope` and `tree_scope_verify` keep their meaning, and a non-empty `tree_scope_verify` must start with `lyx gate test`: darn's config load refuses any other, naming `lyx config darn`, so the tree-scoped command is slotted too.
  `darnengine`'s doc example moves from `--tags` to `--tier`.
- `language.yaml`'s `verify_tiers` is retired; the prompts that rendered it render one line per hub tier with its number and `about`, and each Go-read gate's list.

Rationale: each retired key or section is a hand-spelled test command or rule the tiers now state once.

### D10 — Raw test commands are refused

The shell guard gains a row built from `refuse_raw`: a listed prefix in command position is refused, with a steer naming `lyx gate test`, `--tier` and `--card` and the gate rule's foreground and retry wording.
Every engine renders it; an empty list renders no pattern.
The `raw-go` row keeps its refusals; its steer drops `--tags` for `--tier`.
The plan gate refuses a card `**Verify:**` line holding a `refuse_raw` prefix in command position, naming `lyx gate test`, since Go runs that line outside any agent hook.

Bound: the row is a guardrail like every guard row, seeing static text only, and it lets through a test hidden behind a script or `bash -c`.
Settings render at spawn, so a session already running keeps the list it started with.
Rationale: the operator decided all testing goes through the gate; a hub-declared list keeps lyx naming no tool.

### D11 — The per-run log survives teardown

Every tier run writes one JSON record and its full output log under the board's ephemeral gate directory: worktree, slug where there is one, tier list, tier, command, run directory, scope form, slots held, wait start, start, end, wall seconds, exit code, timed-out flag, outcome (passed, failed or skipped with nothing to test) and the counts.
The gate prunes records and logs older than 14 days when it writes.
Outside every hub they go under the worktree's own `.lyx` gate directory.
The record is a domain log of tier runs, not the process audit log note `spawn-audit-log` plans.

Rationale: the 87-minute verify run in note `dotnet-verify-protocol` was undiagnosable because its log went with the worktree; the board persists past teardown and stays machine-local, like the slot locks.

### D12 — Hub values, applied by the orch

Hub config lives on each hub's board, which no task card writes; after the landing's deploy, the orch sets both hubs with `lyx config gate` before the next gate run.

loomyard (also the example in `internal/gatecli`'s doc):

- `scoper: go`, `guard_tier: 1`, `refuse_raw: ["go test"]`.
- Every tier: `env: {GOFLAGS: -trimpath}`, `all_targets: ./...`, `counter: go-json`, `parallel: 4`, a command ending `go test -json -p {parallel} <tags> {targets} {args}`.
- Each tier's `build_tags` is its tag, empty for tier0.
- `tier0`: untagged tests, the `go test` alone.
- `tier1`: `integration`; `go build ./...` and `go vet {targets}` under each of `integration`, `tmux` and `llm` first; `covers: [0]`; `prebuilt_lyx: true`.
- `tier2`: `tmux`; `covers: [0]`; `prebuilt_lyx: true`.
- `tier3`: `llm`; `covers: [0]`; in no gate list.
- `gates`: `card: [0]`, `batch: [0,1]`, `round: [0,1]`, `final: [0,1]`, `landing: [0,1]`, `publish: [2]`, `push: [0,1,2]`.
- The orch sets each tier's `timeout_min` from its measured whole-run wall time and its `min_tests` from a whole-run count.

NORCE Models (run directory `src/csharp/NORCE.Models`, facts from note `dotnet-verify-protocol`):

- `scoper: none`, `refuse_raw: ["dotnet test"]`, every tier `counter: vstest`.
- `tier0`: the fast suite, `dotnet test NORCE.Models.sln --nologo -m:{parallel} {args}`, `timeout_min: 10`.
- `tier1`: the slow suite, `dotnet test NORCE.Models.SlowTests -p:RunSlowTests=true --nologo {args}`, `timeout_min: 10`.
- `tier2`: the baseline in verify mode, `dotnet test NORCE.Models.NumericalBaseline -p:RunBaseline=true --nologo {args}`, `env: {NORCE_BASELINE_MODE: verify}`, `timeout_min: 3`; in no gate list.
- `gates`: `card`, `batch`, `round` and `final` `[0]`, `landing: [0,1]`, `publish: []`.
- Floors from each project's count, as for loomyard.

`CLAUDE.md`'s push rule becomes `lyx gate test --tier 0 --tier 1 --tier 2` on `main`, where no parent makes the scope whole.

Rationale: the board names both hubs' tiers; the values are config, so the orch owns them, and the loomyard block doubles as the documented example.

## Constraints

- `PATTERN-verified-tree`: every site still runs its command through `verifytree.Verify`; its background changes (the slot moves into the gate, sites render gate calls).
- `PATTERN-hub-wide-config` and `PATTERN-config-layers`: `gate.yaml` stays `HubWide` on the board with its machine override beside it, read only through `configengine`.
- `PATTERN-sole-parsers`: `--card` and the card `**Verify:**` check read the plan only through `internal/planparser`.
- `PATTERN-shell-guard-table`: the new row lives in `internal/shellguard` and every engine renders it.
- `PATTERN-no-denied-recovery`: no refusal, steer or stencil names a `refuse_raw` command as a way forward.
- `PATTERN-durable-vs-ephemeral-state`: the gate run log is diagnostic under `.lyx`; no run reads it to proceed.
- `PATTERN-trimpath-safe-root`: loomyard's tier `env` keeps `-trimpath` on every gate build.

## Auto-mode assumptions

No operator answered; every pick in the support log's question ledger is an auto-pick, the weightiest being D1 (tiers in `gate.yaml`), D6 (slots move from `verifytree` into the gate), D7 (floors on whole runs only), D8 (sites render resolved `--tier` lists, no `--gate` flag) and D12 (the orch applies hub values).

## Open risks

- Between the landing's deploy and the orch setting loomyard's tiers, every gate in an in-flight run is refused naming `lyx config gate`, and the run re-steps once the tiers are set; fork prompts rendered with `--tags` hit its refusal.
- Task `tuned-defaults-template` changes how reconcile treats seeded template values; if it lands first, the seeded `gates` lists must not override the orch's values, and the order of the two landings decides which applies.
- The `vstest` counter's summary-line shape is taken from the note's facts, not a Models run; a dotnet version printing another shape fails every Models tier on its floor at the first run.
- The `go-json` render must keep failure lines, panics and build errors readable; a lossy render makes every Go failure harder to fix.
- Windows kills only the tier's started process on timeout, not its tree (note `verify-watchdog`).
- Task `darn-verify-fixes` assumes `publish_verify` and a darn session that cannot run the tmux tier; this task retires the first and lets `lyx gate test --tier 2` fix the second, so that task is re-scoped after this one lands.
- Running task `origin-reconcile` edits `internal/landingshed` and `internal/loomcli`, which D8 also edits; expect merge-in conflicts.

## Acceptance criteria

- `lyx gate test` with no flags runs tier0 over the computed scope; `--tier 0 --tier 1` with `tier1` covering `[0]` runs only tier1.
- `--card` with a known id runs the card's packages and guard step; an unknown id is refused before any slot.
- `--all`, `--base`, explicit targets and `-C` set `{targets}` and the run directory as D2 and D3 say; a command without `{targets}` runs whole.
- A tier exceeding `timeout_min` across its per-module invocations is killed and fails with the timeout line; a hang before the first tier fails at the fixed ceiling.
- An unfiltered whole run below `min_tests` fails although the command exits zero; a narrower run over targets that hold tests, or a filtered run, that discovers zero fails; a scope with no test-holding target is skipped and reported as nothing to test, including a package whose only test function under the tier's `build_tags` is `TestMain`.
- A `slots: all` tier waits for current holders, blocks new single-slot acquires while it waits, and runs with every slot held; two multi-slot waiters never deadlock; an inherited-slot run acquires nothing.
- The template and a hub with tiers but a dangling gate entry both load; the call or site that uses the dangling entry is refused naming `lyx config gate`, as is an empty non-`publish` gate list.
- Each tier run leaves a JSON record and a log under the board's gate directory that outlive the worktree, and older ones are pruned.
- Every site in D8 runs a `lyx gate test --wait` call rendered from its gate list; Merge-In skips on a tree Webster's gate passed with the same list.
- An empty `publish` list runs nothing at Publish; the Publish failure record names the failing tier, an old-kind record maps as D8 says, and the round reruns the tier.
- A plan with `## verify:` parses and gets the informational finding; no site runs it and no plan-level verify check refuses it.
- `go_parallel`, `publish_verify`, darn's `verify` and `verify_tiers` load as retired keys; a `tree_scope_verify` not starting with `lyx gate test` is refused.
- With `go test` in `refuse_raw`, a raw `go test ./internal/x` in an agent's Bash call is refused, and a card `**Verify:**` line holding it fails the plan gate.
- `--tags` is refused naming `--tier`; a call in a hub with no tiers is refused naming `lyx config gate`.
- The `go-json` render passes a test failure, a panic and a build error through as readable text.

## PATTERN changes

- `PATTERN-gate-slot` — changes: Running a gate build or test: only inside a `lyx gate test` tier run, holding the `internal/gateslot` slots its tier asks for; the tier's config sets its parallelism and environment.
- `PATTERN-test-tiers` — adds: Running a hub's tests: only `lyx gate test` over `gate.yaml`'s tiers; a site or stencil names a gate's tier list and scope form, never a test command or target list.

## Notes for the plan writer

- `pattern/PATTERN-test-speed.md`'s tier table and "Gate per tier" section move to the hub's tier numbers (tier0 untagged to tier3 `llm`) and to D8's gates; `pattern/PATTERN-verified-tree.md` loses the slot line and gains the rendered gate calls.
- The per-tier `parallel` machine override needs a category under an open map; `configengine.Spec.Categories` classifies by dotted path covering every path under it, so this may need a wildcard form or a separate machine-category map.
- `cmd/testtiming` decodes the gate template directly, so it meets the template change.
- The Publish failure record's failing-test parse can read the `go-json` events instead of the text log.
- The shell guard row's patterns come from hub config, so the engine settings renderer must be told the list.
