# Decision record — test-tiers

## Goal

Every test run in a hub, an agent's own and every verify gate's, apart from D13's non-tier slot holders, goes through `lyx gate test`, which runs numbered tiers the hub declares in `gate.yaml` over a scope lyx computes, takes the gate slots each tier asks for, enforces each tier's timeout, counts its tests, and logs every tier run where it survives teardown.
`lyx gate test` stops being a `go test` wrapper: lyx hard-codes no test command, and a C# hub such as NORCE Models gets the same slotted, bounded, observable gate as this Go hub.

## Scope

In:

- `internal/gateslot` and `internal/gatecli`: tier config, the tier-running verb, per-tier slots and the drain, counters and floors, the per-run log.
- `internal/impactset` as the `go` scoper.
- Every verify site D8 lists, the card `**Verify:**` runner and its `lyx gate card-verify` verb, and the stencils and specs that render gates or `verify_tiers`.
- The retirements in D9 and the shell guard row in D10.
- The non-tier slot holders in D13.
- `lyx config`'s flow-mapping `--set` for an open-map entry (D1).
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
  A command without `{targets}` ignores any narrower scope and runs whole, saying so, except that an empty computed, `--base` or card scope runs no listed tier's command (D3).
  Words after `--` for a tier whose command lacks `{args}` are refused, naming the tier and `lyx config gate`.
- `dir`: the run directory relative to the worktree root, default `.`.
- `env`: environment variables for the command; values may hold `{parallel}`.
- `all_targets`: the `{targets}` value of a whole-scope run.
- `parallel`: at least 1.
- `timeout_min`: at least 1; no tier is unbounded.
- `counter`: `go-json` or `vstest` (D7).
- `covers`: lower tier numbers this tier's command runs anyway.
- `slots`: at least 1, or `all`; default 1.
- `prebuilt_lyx`: default false; true runs today's tagged-run `lyx` prebuild for the tier, at the tier's `parallel`, inside the tier's slots and timeout, only where the worktree root holds `cmd/lyx`.

Top-level keys beside `slots` and `busy_quit_sec`:

- `gates`: gate name to a list of tier numbers.
  The Go sites read `card`, `batch`, `round`, `final`, `landing` and `publish`; other names, such as loomyard's `push`, are the operator's own.
- `scoper`: `go` or `none`.
- `guard_tier`: the tier whose command runs the `//lyx:guard` tests (D5), read only under `go`.
- `refuse_raw`: command prefixes the shell guard refuses (D10), each one or more words of letters, digits, `.`, `_` and `-`.

Load requires each declared tier to hold `command`, `timeout_min` and `counter`, and `all_targets` when its command holds `{targets}`, and validates the key floors, the counter and scoper names, `covers` entries lower than their tier, and the `refuse_raw` word shape, refusing with the key and `lyx config gate`.
A tier is therefore written whole: `lyx config gate --set` gains a YAML flow-mapping value for an open-map entry, as in `--set 'tiers.tier0={command: "go test -json {targets} {args}", timeout_min: 60, counter: go-json}'`, which replaces the entry whole and is validated by the loader like any value.
Inside the mapping, a value holding a brace, comma or bracket is a double-quoted string, since a YAML flow-context plain scalar may hold none of them.
Cross-references are checked at use, never at load: a call or site whose tier list, `covers` or `guard_tier` names a tier the hub does not declare, or that finds no `tier0`, is refused naming `lyx config gate`.
So the template, a hub whose gate lists name a tier not yet declared, and every slot-only reader of the config still load.
A machine-layer `parallel` for a tier the hub file does not declare is dropped at load with a logged warning naming the operator's `lyx config gate --layer machine` editor session as the way to remove it, so a tier the hub removes leaves no machine entry that fails the load.

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
  `-C` with any other scope form is refused before any slot, naming explicit targets.
- `--tags` is retired: refused before any slot, naming `--tier`.
- `--guards` keeps its meaning under `go` and is refused under `none`.
- `--wait` lifts the keep-cache-warm cap on the slot wait; every Go verify site, every `tree_scope_verify` (D9) and every card `**Verify:**` gate call (D10) passes it, so a busy hub never fails a gate.
- A call in a hub with no tier, or outside every hub with no worktree-layer `gate.yaml`, is refused naming `lyx config gate`; outside every hub a standalone worktree's own `gate.yaml` is read and the run is unslotted, as today.
- A tier's `timeout_min` covers all its invocations, D3's per-module runs included, and starts once its slots are held; D5's guard step is a run of `guard_tier` under that tier's own timeout; a tier that outlives it has its process tree killed and fails with a timeout line.
- The work before the first tier (scope computation, plan read, guard scan) is bounded by a fixed 10-minute ceiling, failing with a timeout line, so a Go site's call has no unbounded step other than the slot wait.
- One line per tier run goes to stderr: tier, scope form, counts, wall seconds, exit code and log path.
- The environment stripping applies to each tier command.
  The CPU watchdog watches Go test binaries only, so a `vstest` tier is bounded by its `timeout_min` alone.

Bound: `--base`, `--card` and explicit targets only change which targets a tier runs over and never the slot rules.
They add no tier to the list except D5's guard step, a run of `guard_tier`, and an empty computed, `--base` or card scope runs no listed tier's command (D3); `--all` only widens.
`--wait` lets any caller block one call for as long as the hub is busy; an agent that passes it only loses its own prompt cache.
Rationale: the board's operator decisions ask for tiers, computed scope with `--tier` and `--card`, and explicit targets as the exception.

### D3 — Computed scope

The scoper maps the diff from a base to the working tree, tracked changes and untracked files that are not ignored, onto `{targets}`.
The default base is the merge base of HEAD with the parent branch fabric's origin record names; with no parent, such as the prime on `main` or a standalone run, the run is whole.

- `none`: a computed, `--base` or card scope is whole, with `{targets}` the tier's `all_targets`; explicit targets still pass verbatim as a narrower run (D2).
- `go`: `internal/impactset` derives the impacted package set as today, as targets per module rather than a command, and falls back to whole wherever it falls back today.
  A tier then runs once per module, in that module's directory, with module-relative targets, under one slot hold.
  A diff touching only `.md` files is an empty set: no tier command runs, each listed tier passes with zero discovered and is logged so, and only D5's guard step runs.

Rationale: card gates and an agent's own checks run before the commit, so the gate tests the working tree.
Bound: the `go` scoper narrows where impactset narrows today, and one step further: an empty set runs no tier command, so `tier1`'s `go build ./...`, which today's command runs for an empty set, is left to the whole runs at `final` and `landing` (D8), as is what impactset's import graph cannot see.

### D4 — Card scope

`--card <id>` reads the plan through `internal/planparser` and takes the union of the named cards' targets.
Under `go`, those become the cards' surviving package directories per module, the rule Webster's card gate applies today, plus D5's guard step; under `none` the run is whole.
An unknown card id is refused before any slot, naming the plan's card ids; a worktree with no plan is refused, naming `--all` or explicit targets.

Rationale: a card's gate keeps pointing at exactly the card that broke, as the operator decided for note `loom-gates-xdr-stacking`.

### D5 — Guard tests under the `go` scoper

A computed or card scope also runs the module's `//lyx:guard` tests outside the scope, by name in their own packages, through `guard_tier`'s command with the guard packages as `{targets}` and `-run '^(<names>)$'` as `{args}`, once per call after the listed tiers pass.
The step is a run of `guard_tier`, whether or not the call lists it: it acquires `guard_tier`'s slots after the last listed tier released its own, and runs under `guard_tier`'s `timeout_min` and `prebuilt_lyx`.
It is skipped when the call ran `guard_tier`, or a tier covering it, whole.
Only a guard step that is not skipped checks `guard_tier`'s command: one lacking `{targets}` or `{args}` is refused then, naming `lyx config gate`.
A guard scan error refuses the call with the file, line and way forward `--guards` gives today.

Rationale: the card and round gates run the guard tests today; the scoper keeps that coverage without a site spelling a guard command.
Cost: the guard step runs `guard_tier`'s whole command over the guard packages, its build and vet steps included, and its `lyx` prebuild when `guard_tier` sets `prebuilt_lyx`, as loomyard's does.

### D6 — Per-tier slots and the drain

Each tier acquires its slots, runs and releases them before the next tier.
`slots: 1` takes the first free slot, as today.
A tier asking for more, or `all` (the current `slots` count; a larger number is clamped to it), first takes the pool's drain lock, then takes its slots one at a time in ascending order, holding what it has while it waits.
While another process holds the drain lock, a single-slot acquire waits.
A run inside an inherited slot (`gateslot.InheritEnv`) acquires nothing, whatever its tier asks for.
`Lease.Env` carries only the inheritance variable: parallelism and Go flags come from the tier's `command` and `env`, or from the holder itself for a slot holder that is not a tier (D13).
The holder record gains the tier, and the gate writes its wait record whenever it waits; loom status reads the wait record ahead of the verify marker, so a waiting gate shows as waiting.

Bound: the drain lets one heavy tier at a time wait ahead of new single-slot acquires, and the holders it waits for are bounded by their own tier timeouts.
An inherited run, such as a test that calls `lyx gate test` inside the outer gate's tier, is not sequenced by its own `slots`, and runs inside its parent's.
Rationale: ordered acquisition prevents two multi-slot waiters from deadlocking, and the drain prevents starvation.

### D7 — Counters

- `go-json`: the command emits `go test -json`; lyx renders the events' output text to the terminal in order, passes non-JSON lines through verbatim, and reports discovered, passed, failed and skipped counts.
- `vstest`: lyx passes output through and sums the fields of every VSTest summary line into the same counts.

A tier run's outcome is its command's exit code, or failure on a timeout (D2); the counts are printed and logged (D11) and decide nothing, so a run discovering zero passes, as the operator decided below.
The scoper drops no target: a narrower run covers every impacted, card or explicit target, test-less ones included.

Rationale: a named counter is the format knowledge a Go hub needs anyway to render `-json` output, and it makes every run's count observable.

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
| Merge-In, catch-up merge-ins, Finalize | `landing` | `--all`; a darn run with a tree-scoped diff runs `tree_scope_verify` instead (D9) |
| Publish | `landing`, then `publish` | `--all`; a darn run with a tree-scoped diff runs `tree_scope_verify` in place of `landing` and skips `publish` |

- `final` and `landing` run whole, keeping the operator decision in note `loom-gates-xdr-stacking` that Webster's plan-level gate runs the whole suite.
- The round's base command is loom's `final` command and darn's `landing` command, the commands whose passes the round narrows from.
  Merge-In and Publish name that base command on Site, so its pass entry survives a landing-list pass where `final` and `landing` differ.
- Two sites with the same list and form render the same command, so Merge-In skips on a tree Webster's gate passed when `final` and `landing` hold the same list.
- An empty `publish` list takes the path of today's unset `publish_verify`.
- Card and batch gates keep `lyx loom lint-comments` and `lyx loom lint-pattern` after the gate call; the round keeps its lints in Go.
- The Publish failure record names the failing tier, and the round's confirm-the-fix pass runs that tier for the record's failing tests and packages.
  Each failing-test run and the package pass is its own `lyx gate test --wait --tier <failing tier>` call, a failing test's package as explicit target with its `-run` filter as the words after `--`, and the package pass with the packages as explicit targets; they join the round's gate call with `&&`, as the raw `go test` steps do today.
  A failing tier whose command lacks `{targets}` or `{args}` gets D1's and D2's handling of such a tier.
  A failing test the fix renamed or removed discovers zero there and passes (D7).
  A record written before the deploy with kind `plan_verify` reads as the `landing` list, and `publish_verify` as the `publish` list.
- `verifytree.Verify` no longer takes a gate slot or applies its 60-minute timeout: every command it runs is one `lyx gate test --wait` call, or such calls joined by `&&` (D9), so it holds no slot itself, and each call is bounded by D2's fixed ceiling and its tiers' timeouts.
- The card `**Verify:**` runner runs a line that is one `lyx gate test --wait` call, with no shell control operator, unslotted and outside its 10-minute timeout, which the gate's own bounds replace.
  Any other line keeps today's single gate slot and the timeout that starts once the slot is held; it is a non-tier holder (D13).
- The implementer fork runs a card's `**Verify:**` only through a new verb, `lyx gate card-verify <card id>`, which runs the card's lines in order through that same runner, passes each line's output through to the terminal where Go's re-run discards it, stops at the first failing line with its exit code, and waits for its slot as `--wait` does.
  The verb sits under `lyx gate`, since the fork-mode hook refuses every `lyx webster` command from a fork.
  The fork and recovery prompts render that call in place of the lines, so the fork's run and Go's re-run of a line hold the same slot under the same bounds.

Bound: a changed tier command or `about` does not invalidate a recorded pass on an unchanged tree, since the record keys on the rendered command; a changed tier list does.
Rationale: no site or agent spells a test command or a package list.

### D9 — Retired config and plan inputs

- The plan's `## verify:` section is no gate input.
  The parser keeps accepting it, so an in-flight plan still parses; the plan gate reports a present section as an informational finding, and the plan-level checks on it (missing package arguments, cross-module commands) are retired.
  The plan spec's verify model, the plan template and the Plan-Review rubric drop it; a card's `**Verify:**` stays the optional escape hatch.
  The Webster fork and recovery prompts stop rendering the plan's verify section, an in-flight plan's included, and render the hub's tier lines and gate lists instead.
- `landing.yaml`'s `publish_verify` is retired; the `publish` gate replaces it, and darn's skip seam skips the `publish` gate.
- `darn.yaml`'s `verify` is retired; darn's sites run their gate lists.
  `tree_scope` and `tree_scope_verify` keep their meaning, and a non-empty `tree_scope_verify` must be one `lyx gate test --wait` call holding no shell control operator: darn's config load refuses any other, naming `lyx config darn`, so the whole tree-scoped command is slotted and waits out a busy hub.
  When the rendered `{targets}` is empty, as for a docs-only tree diff, darn runs no command, logs it, and the site passes, so an empty list never falls back to D3's computed scope.
  `darnengine`'s doc example moves from `--tags` to `--wait --tier`, and the darn template's `tree_scope_verify` comment drops its empty-`{targets}` requirement for this rule.
- `language.yaml`'s `verify_tiers` is retired; the prompts that rendered it render one line per hub tier with its number and `about`, and each Go-read gate's list.

Rationale: each retired key or section is a hand-spelled test command or rule the tiers now state once.

### D10 — Raw test commands are refused

The shell guard gains a row built from `refuse_raw`: a listed prefix in command position is refused, with a steer naming `lyx gate test`, `--tier` and `--card` and the gate rule's foreground and retry wording.
Every engine renders it; an empty list renders no pattern.
The `raw-go` row keeps its refusals; its steer drops `--tags` for `--tier`.
The plan gate refuses a card `**Verify:**` line holding a `refuse_raw` prefix in command position, naming `lyx gate test`, since Go runs that line outside any agent hook.
It also refuses a `lyx gate test` call in a `**Verify:**` line that lacks `--wait`, naming `--wait`.

Bound: the row is a guardrail like every guard row, seeing static text only.
Command position is the shell guard's: the start of the command or after a separator, past an `env`, `exec`, `xargs`, `sudo` or `time` launcher.
The row lets through a test hidden behind a script or `bash -c`, a flag between the prefix's words such as `go -C dir test`, and an assignment prefix such as `GOFLAGS=x go test`.
Settings render at spawn, so a session already running keeps the list it started with.
Rationale: the operator decided all testing goes through the gate; a hub-declared list keeps lyx naming no tool.

### D11 — The per-run log survives teardown

Every tier run writes one JSON record and its full output log under the board's ephemeral gate directory: worktree, slug where there is one, tier list, tier, command, run directory, scope form, slots held, wait start, start, end, wall seconds, exit code, timed-out flag, outcome (passed or failed), whether it was D5's guard step, and the counts.
The gate prunes records and logs older than 14 days when it writes.
Outside every hub they go under the worktree's own `.lyx` gate directory.
The record is a domain log of tier runs, not the process audit log note `spawn-audit-log` plans.

Rationale: the 87-minute verify run in note `dotnet-verify-protocol` was undiagnosable because its log went with the worktree; the board persists past teardown and stays machine-local, like the slot locks.

### D12 — Hub values, applied by the orch

Hub config lives on each hub's board, which no task card writes; after the landing's deploy, the orch sets both hubs with `lyx config gate` before the next gate run, writing each tier whole with one flow-mapping `--set` (D1).

loomyard (also the example in `internal/gatecli`'s doc):

- `scoper: go`, `guard_tier: 1`, `refuse_raw: ["go test"]`.
- Every tier: `env: {GOFLAGS: "-trimpath -p={parallel}"}`, so every go command in the tier, its build and vet included, takes the cap; `all_targets: ./...`, `counter: go-json`, `parallel: 4`, a command ending `go test -json <tags> {targets} {args}`.
- `tier0`: untagged tests, the `go test` alone.
- `tier1`: `integration`; `go build ./...` and `go vet {targets}` under each of `integration`, `tmux` and `llm` first; `covers: [0]`; `prebuilt_lyx: true`.
- `tier2`: `tmux`; `covers: [0]`; `prebuilt_lyx: true`.
- `tier3`: `llm`; `covers: [0]`; in no gate list.
- `gates`: `card: [0]`, `batch: [0,1]`, `round: [0,1]`, `final: [0,1]`, `landing: [0,1]`, `publish: [2]`, `push: [0,1,2]`.
- The orch first sets every tier's `timeout_min` to 60, the ceiling `verifytree` applies today, then rewrites each tier with a `timeout_min` fitting its measured whole-run wall time.

NORCE Models (run directory `src/csharp/NORCE.Models`, facts from note `dotnet-verify-protocol`):

- `scoper: none`, `refuse_raw: ["dotnet test"]`, every tier `counter: vstest`.
- `tier0`: the fast suite, `dotnet test NORCE.Models.sln --nologo -m:{parallel} {args}`, `timeout_min: 10`.
- `tier1`: the slow suite, `dotnet test NORCE.Models.SlowTests -p:RunSlowTests=true --nologo {args}`, `timeout_min: 10`.
- `tier2`: the baseline in verify mode, `dotnet test NORCE.Models.NumericalBaseline -p:RunBaseline=true --nologo {args}`, `env: {NORCE_BASELINE_MODE: verify}`, `timeout_min: 3`; in no gate list.
- `gates`: `card`, `batch`, `round` and `final` `[0]`, `landing: [0,1]`, `publish: []`.

`CLAUDE.md`'s push rule becomes `lyx gate test --tier 0 --tier 1 --tier 2` on `main`, where no parent makes the scope whole.

Rationale: the board names both hubs' tiers; the values are config, so the orch owns them, and the loomyard block doubles as the documented example.

### D13 — Slot holders that are not a tier

These paths hold one gate slot outside a `lyx gate test` tier run:

- a card `**Verify:**` line that is not one `lyx gate test` call, whether Go's re-run or the fork's `lyx gate card-verify` runs it (D8), under the runner's 10-minute timeout from the moment the slot is held;
- `internal/planglyph`'s type load;
- `cmd/testtiming` in every mode, its default timing mode, `-redundancy` and `-resources`, holding one slot for the whole run, its `lyx` prebuild included; today only `-resources` and the prebuild hold one.

Once `Lease.Env` carries only the inheritance variable, each appends GOFLAGS `-trimpath` itself, as today, plus `-p=<tier0's parallel>` in place of today's `go_parallel` when the hub declares `tier0`, and no `-p` when it does not; `cmd/testtiming` appends the same flags to every `go` command of every mode and takes the same value as its own parallelism, and the CPU count outside a hub, where it holds no slot.
A `lyx gate test` call inside any of them inherits the slot (D6), and `internal/gateslot`'s doc lists them.
`cmd/testtiming` stays outside the gate because its modes measure what a tier run would hide: per-package wall time and resources, and per-test coverage for `-redundancy`.

Rationale: each holder runs a build or test the gate cannot express as a tier, and the slot keeps it from overloading the machine.

### Added after Discussion (operator, 2026-10-21): Zero discovered tests is not a failure

Decision: A test run that discovers zero tests passes, in every scope: when the diff touches no tests, the run passes. There is no minimum-test floor and no separate skipped outcome; the discovered count is still logged. This replaces D7's zero-test handling and withdraws the round-1 parent review's zero-tests ruling.

Rationale: The operator assumes no bug of the dropped-flag kind. The board line saying zero discovered tests is a failure came from note dotnet-verify-protocol and was never an operator decision; the orch misattributed it.

## Constraints

- `PATTERN-verified-tree`: every site still runs its command through `verifytree.Verify`; its background changes (the slot moves into the gate, sites render gate calls).
- `PATTERN-hub-wide-config` and `PATTERN-config-layers`: `gate.yaml` stays `HubWide` on the board with its machine override beside it, read only through `configengine`.
- `PATTERN-sole-parsers`: `--card` and the card `**Verify:**` check read the plan only through `internal/planparser`.
- `PATTERN-shell-guard-table`: the new row lives in `internal/shellguard` and every engine renders it.
- `PATTERN-no-denied-recovery`: no refusal, steer or stencil names a `refuse_raw` command as a way forward.
- `PATTERN-durable-vs-ephemeral-state`: the gate run log is diagnostic under `.lyx`; no run reads it to proceed.
- `PATTERN-trimpath-safe-root`: loomyard's tier `env` and D13's non-tier holders keep `-trimpath` on every gate build.

## Auto-mode assumptions

Apart from the zero-test rule the operator decided after Discussion, every pick in the support log's question ledger is an auto-pick, the weightiest being D1 (tiers in `gate.yaml`), D6 (slots move from `verifytree` into the gate), D7 (named counters), D8 (sites render resolved `--tier` lists, no `--gate` flag) and D12 (the orch applies hub values).

## Open risks

- Between the landing's deploy and the orch setting loomyard's tiers, every gate in an in-flight run is refused naming `lyx config gate`, and the run re-steps once the tiers are set; fork prompts rendered with `--tags` hit its refusal.
  An in-flight plan's card `**Verify:**` line holding `--tags` passed the old plan gate and fails its card re-run after the deploy, since D2 refuses `--tags`; it is fixed by rewriting the line to `--tier` with `--wait`, then running `lyx webster rebaseline` naming that card.
  An in-flight line that is a gate call lacking `--wait`, such as `lyx gate test ./internal/x`, is not one `lyx gate test --wait` call, so the runner takes its single slot and its nested gate call inherits that slot (D6, D8); it waits for the slot without the keep-cache-warm cap, runs under the runner's 10-minute timeout, and does not fail on a busy hub.
- Task `tuned-defaults-template` changes how reconcile treats seeded template values; if it lands first, the seeded `gates` lists must not override the orch's values, and the order of the two landings decides which applies.
- The `vstest` counter's summary-line shape is taken from the note's facts, not a Models run; a dotnet version printing another shape logs wrong counts for every Models tier, unnoticed until someone reads the log.
- The drain does not hold back processes still running the pre-deploy binary, such as an in-flight card runner or `verifytree`: they keep acquiring single slots, unaware of the drain lock, until they exit.
- The `go-json` render must keep failure lines, panics and build errors readable; a lossy render makes every Go failure harder to fix.
- Windows kills only the tier's started process on timeout, not its tree (note `verify-watchdog`).
- Task `darn-verify-fixes` assumes `publish_verify` and a darn session that cannot run the tmux tier; this task retires the first and lets `lyx gate test --tier 2` fix the second, so that task is re-scoped after this one lands.
- Running task `origin-reconcile` edits `internal/landingshed` and `internal/loomcli`, which D8 also edits; expect merge-in conflicts.

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 4: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/plan/round-4-review.md; fixer report: _lyx/reviews/plan/round-4-fixer-report.md.
  - absent-gate-yaml-refused-at-wiring (class consistency, severity BLOCKING, open in rounds 4)
  - decodeconfig-resign-arrow-missing (class consistency, severity MEDIUM, open in rounds 4)
  - shared-fixtures-missed-by-new-behavior (class design, severity BLOCKING, open in rounds 3, 4)
<!-- /lyx:carry-over Plan-Review -->

<!-- lyx:carry-over Webster-Review -->
- Webster-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/webster/round-1-review.md; fixer report: _lyx/reviews/webster/round-1-fixer-report.md.
  - triage-section-names-retired-verify-suite (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Webster-Review -->

## Acceptance criteria

- `lyx gate test` with no flags runs tier0 over the computed scope; `--tier 0 --tier 1` with `tier1` covering `[0]` runs only tier1.
- `--card` with a known id runs the card's packages and guard step; an unknown id is refused before any slot.
- `--all`, `--base`, explicit targets and `-C` set `{targets}` and the run directory as D2 and D3 say, and `-C` with another scope form is refused; a command without `{targets}` runs whole under a non-empty scope, and under the `go` scoper for a `.md`-only diff runs no command, like every other tier; under `none`, a `.md`-only diff runs the listed tiers whole, skipping a covered one as D2 says.
- Under `none`, a `--base` run takes `all_targets`, an explicit-target run passes its targets verbatim, and a run with words after `--` passes them as `{args}`.
- A tier exceeding `timeout_min` across its per-module invocations is killed and fails with the timeout line; a hang before the first tier fails at the fixed ceiling.
- A run that discovers zero passes and logs zero, whole or narrower, filtered or not, including a package with only a `TestMain`, a Benchmark-only package and a confirm-the-fix run whose failing test was renamed.
- Under `go`, a narrower `tier1` run vets every impacted package, test-less ones included; a `.md`-only diff runs no tier command and only the guard step.
- loomyard's `tier1` build and vet run with `-p={parallel}` in GOFLAGS.
- The guard step of a `card: [0]` call with `guard_tier: 1` acquires tier1's slots and runs under tier1's `timeout_min`; a call that ran tier1 whole checks no `guard_tier` placeholder.
- A `slots: all` tier waits for current holders, blocks new single-slot acquires while it waits, and runs with every slot held; two multi-slot waiters never deadlock; an inherited-slot run acquires nothing.
- The template and a hub with tiers but a dangling gate entry both load; the call or site that uses the dangling entry is refused naming `lyx config gate`, as is an empty non-`publish` gate list.
- A declared tier missing `command`, `timeout_min` or `counter`, or `all_targets` under a `{targets}` command, is refused at load naming the key; a flow-mapping `--set` writes a whole tier; a machine-layer `parallel` for an undeclared tier is dropped with a warning and the config loads.
- Each tier run leaves a JSON record and a log under the board's gate directory that outlive the worktree, and older ones are pruned.
- Every site in D8 runs a `lyx gate test --wait` call rendered from its gate list; Merge-In skips on a tree Webster's gate passed with the same list.
- A darn Merge-In, Finalize and Publish of a tree-scoped diff run `tree_scope_verify`, and Publish skips `publish`; with an empty rendered `{targets}` darn runs no command and the site passes.
- On a hub whose slots are all held past the keep-cache-warm cap, a `tree_scope_verify` and a card `**Verify:**` gate call wait and pass, without the card runner's or `verifytree`'s timeout firing.
- A card `**Verify:**` line `go vet ./internal/x` runs holding one slot, both in Go's re-run and in the fork's `lyx gate card-verify` call, and that call shows the line's output; a line that is one `lyx gate test --wait` call runs unslotted.
- `cmd/testtiming` holds one slot in each of its modes.
- Each non-tier holder, a card `**Verify:**` line that is not one `lyx gate test` call whether Go's re-run or `lyx gate card-verify` runs it, the `internal/planglyph` type load and every `cmd/testtiming` mode, passes its child GOFLAGS carrying `-trimpath`, plus `-p=<tier0's parallel>` when the hub declares `tier0` and no `-p` when it does not; `cmd/testtiming` outside a hub uses the CPU count and holds no slot.
- An empty `publish` list runs nothing at Publish; the Publish failure record names the failing tier, an old-kind record maps as D8 says, and the round reruns the tier.
- A plan with `## verify:` parses and gets the informational finding; no site runs it and no plan-level verify check refuses it.
- `go_parallel`, `publish_verify`, darn's `verify` and `verify_tiers` load as retired keys; a `tree_scope_verify` that is not one `lyx gate test --wait` call, or that holds a shell control operator, is refused.
- With `go test` in `refuse_raw`, a raw `go test ./internal/x` in an agent's Bash call is refused, and a card `**Verify:**` line holding it, or a `lyx gate test` call lacking `--wait`, fails the plan gate.
- `--tags` is refused naming `--tier`; a call in a hub with no tiers is refused naming `lyx config gate`.
- The `go-json` render passes a test failure, a panic and a build error through as readable text.

## PATTERN changes

- `PATTERN-gate-slot` — changes: Running a gate build or test: only holding `internal/gateslot` slots, either inside a `lyx gate test` tier run, holding the slots its tier asks for, with the tier's config setting its parallelism and environment, or in one of the non-tier holders `internal/gateslot`'s doc lists, holding one slot and setting its own `-trimpath` Go flag, plus `-p` from `tier0`'s parallel when the hub declares `tier0`.
- `PATTERN-test-tiers` — adds: Running a hub's tests: only `lyx gate test` over `gate.yaml`'s tiers, apart from a card `**Verify:**` line and `cmd/testtiming`'s runs in every mode; a site or stencil names a gate's tier list and scope form, never a test command or target list.

## Notes for the plan writer

- `pattern/PATTERN-test-speed.md`'s tier table and "Gate per tier" section move to the hub's tier numbers (tier0 untagged to tier3 `llm`) and to D8's gates; `pattern/PATTERN-verified-tree.md` loses the slot line and gains the rendered gate calls.
- The per-tier `parallel` machine override needs a category under an open map; `configengine.Spec.Categories` classifies by dotted path covering every path under it, so this may need a wildcard form or a separate machine-category map.
- `cmd/testtiming` decodes the gate template directly, so it meets the template change.
- `contracts/stencils/webster/webster-body-implementer.md` moves from running each `verify:` line as a Bash call to one `lyx gate card-verify` call per card (D8): step 5, the foreground rule at its head and the `FAILED` status wording that speak per line.
- `lyx gate card-verify` reuses the card runner in `internal/websterengine/cardverify.go`; the runner may move to a package both `gatecli` and `websterengine` import.
- The Publish failure record's failing-test parse can read the `go-json` events instead of the text log.
- The shell guard row's patterns come from hub config, so the engine settings renderer must be told the list.
