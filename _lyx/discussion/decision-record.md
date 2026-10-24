# Decision record — gate-runs-once

## Goal

Each test runs once per tree and in one tier.
Every producer that runs a test gate names its own `gate.yaml` key, which sets its tiers and its scope.
`lyx gate test` carries a green tier forward over an unchanged tree, so a repeat on the same tree runs nothing.
Shed rows get an entry gate that replaces darn's recipe edge rewrite and the Command rows' switch.
A loom or darn run's gate time is measurable per gate key, launch latency included.

## Scope

In:

- `gate.yaml`'s `gates` reshaped to one key per producer, each with tiers and scope, and `lyx config reconcile` migrating `landing`, `final` and `round`.
- `lyx gate test --gate <key>`, the per-worktree green record, the `diff` scope's base and the run record's gate key.
- Disjoint tiers: build tags renamed to the tier names, `tier0` tagged, `guard_tier` retired, the guard step run per tier.
- Shed row entry gates, the check form, and their two consumers: batten's Command rows and darn's Describe row.
- `lyx gate report` over the machine's gate run records.
- The tier plan per recipe, stated in code docs.
- Filing the board entry that removes the guard files' transitional tag term after deploy (D13).

Out:

- Speeding up `internal/fabricengine`'s tests.
- The regression perch (`regression-baseline-verify`), the piece runs' gate keys (`loom-beam-darns`) and `adopt-runs-tests`' gate; each uses what this task builds.
- Loom's `zero_rounds` Bouncer setting for a budget of 0, which stays as it is.
- The hub's IDE seed and gopls settings.
- Item 4 of the board entry, `tmux` in the final gate, which hub config already did on 2026-10-23; the migration keeps it.

## Decisions

### D1. One gate key per producer, with tiers and scope

`gate.yaml`'s `gates` maps each key to `{tiers: [<n>...], scope: card|diff|all}`.
Webster's keys are `card`, `batch`, `webster` (its end gate, formerly `final`) and `webster-fix` (its two verify fixers, formerly `round`).
A recipe row that runs a test gate reads the key its config's `gate_key` names, else its row name in lower case: `merge-in`, `finalize`, `darn-write` and `code-burler-fix` for the shipped recipes.
The merge fixer runs the key of the row that spawned it.
`landing`, `final` and `round` are retired.
The template holds every key the shipped recipes read, each `{tiers: [0]}` with the scope below.
`lyx config reconcile` migrates a hub file: `landing`'s list becomes `merge-in`, `finalize`, `darn-write` and `code-burler-fix`, `final`'s becomes `webster`, `round`'s becomes `webster-fix`, and `card` and `batch` keep their lists.
Each migrated key takes the scope closest to today's call: `card` for `card` and `batch`, `all` for `webster`, `merge-in` and `finalize`, `diff` for `webster-fix`, `darn-write` and `code-burler-fix`.
`code-burler-fix`'s fallback without a recorded pass moves from today's `--all` to the parent merge base (D2).
Bound: a recipe row's `gate_key` selects only which key of the hub's `gate.yaml` it runs; the tiers and scopes stay hub-file only.
Rationale: operator decisions of 2026-10-23 and 2026-10-24; `gate_key` lets a recipe that keeps darn's row names, such as a piece recipe, take its own tiers.

### D2. Scope values

- `card`: the packages of the cards the site names.
  Only `card` and `batch` may set it; the config load refuses it on any other key, naming `lyx config gate`.
- `diff`: per listed tier, the diff from the newest commit that is an ancestor of HEAD and whose tree the green record (D4) holds whole-green for that tier.
  Without one, the diff from the merge base with the parent branch, which is today's computed form.
  Without a parent, the tier runs whole.
  Tiers that land on the same base share one derivation.
- `all`: the tier's `all_targets`.

Rationale: the board's two `diff` readings, since the last green run and since the fork from the parent, become one chain, the second as the fallback of the first; a narrowed green never becomes a base, so the chain stays sound.

### D3. `lyx gate test --gate <key>`

A site renders `lyx gate test --wait --gate <key>`; the `card` and `batch` sites always add `--card <id>` per card.
`lyx gate test` reads the key's tiers and scope from the config at run time and uses the card ids only under a `card` scope.
It refuses `--gate` alongside `--tier`, `--upto`, `--all`, `--base` or explicit targets, and a `card` scope without `--card`, naming the key.
Without `--gate` the verb keeps its manual flags as today.
The run record gains the gate key, empty for a manual run.
The fix rows' and writers' verify gate keeps its own comment-lint base: the site's latest recorded pass when it is an ancestor of HEAD, else the merge base with the parent, so a first fix round lints rather than skips.
Rationale: one place resolves scope, and the record names the key the measurement groups by; per-site commands no longer share Merge-In's recorded pass, which today gives the fix round its lint base.

### D4. The green record carries a tier forward

`lyx gate test` keeps a per-worktree green record under the worktree's never-tracked `.lyx` gate directory, declared through `internal/lyxdirs`.
An entry names a tier fingerprint, the tree, HEAD's commit, the module directory, the args after `--`, and either whole or the green package set.
The fingerprint is the tier's declaration without `parallel`, `timeout_min` and `slots`, so a changed command or env never carries.
The tree is HEAD's tree, and only on a worktree clean by `verifytree.DirtyPaths`'s rule; a dirty worktree neither reads nor writes the record.
A tier entry is written only after the tier passed and only when HEAD's tree is unchanged and still clean at that point.
Before acquiring slots, each listed tier subtracts what the record already holds green for the current tree:
a whole entry covers every target, and a package entry covers those packages under the `go` scoper.
A tier with nothing left is carried: it takes no slot, runs no command, writes a run record marked carried, and prints its summary line in the existing shape with exit 0, marked carried.
A tier with a remainder runs the remainder; under the `none` scoper only a whole entry carries.
An entry with empty args covers a request with empty args and the guard step's own `-run` filter; any other args need an exact match, so `-- -count=1` reruns.
Writers serialize through a lock in the same directory; pruning drops old trees but keeps each tier's recent whole-green commits, so D2 finds an older ancestor when the newest is not one.
The carry sits under `verifytree`'s command-and-tree skip, which every site keeps.
Bound: a carried tier trusts a pass over the same tree, the same tier declaration and the same args on this worktree; a flaky test that passed once is not rerun, a Go toolchain change is not in the fingerprint, and the record never leaves the machine, so an adopted run tests afresh.
Rationale: after `loom-tail-review` no verify row exists, since the gates run inside Merge-In, Finalize, Darn-Write, Code-Burler-Fix and Webster, whose other work must still run; a row entry gate cannot skip the gate alone, so board item 3 lands in the gate, where every site, the card and batch gates and manual runs included, gets it.

### D5. Disjoint tiers through tier-named tags

Every test file carries exactly one build tag `tierN`, the tier that runs it: `integration`, `tmux` and `llm` become `tier1`, `tier2` and `tier3`, and today's untagged test files become `tier0`.
An untagged `_test.go` file holds no `Test`, `Benchmark`, `Fuzz` or `Example` function; it holds only support such as `TestMain` and helpers.
A test file whose constraint admits two tiers, such as `tier2 || tier3`, holds no test.
The helper rules of `PATTERN-test-speed` carry over under the tier names, and a file whose helpers another tier's tests use is split.
`cmd/lyx/tierpurity_test.go` and `cmd/lyx/llmtier_test.go` are restated with the tier names; untagged becomes `tier0`.
A repository scan test enforces the one-tag rule, allowing D13's transitional term on guard-marked files only.
`cmd/testtiming` and `internal/planparser`'s module-wide check use the tier tags.
`covers` stays a generic tier key; this hub stops using it.
Rationale: operator decision of 2026-10-22 ("every tier tagged by its own name, `tier0` included"); with tag equal to tier name, Go code maps a file to its tier with no config.
The rename adds the already-tagged files to the edit; the measured gain of disjointness is about 20 s of package time per higher-tier run, which the operator decided for.

### D6. The guard step runs per tier

Under the `go` scoper a test file's tier is the `tierN` tag in its constraint, and a file with none counts as `tier0`.
The guard step groups the guard tests outside their own tier's scope by their file's tier and runs each group under that tier's command, slots, timeout and `prebuilt_lyx`.
A group whose tier the call already ran whole, or carried whole, is skipped.
The guard scan's `integration` rule is replaced: a `//lyx:guard` marker is allowed in `tier0` and `tier1` files and refused in `tier2` and `tier3` files.
`guard_tier` is retired: reconcile prunes it from a hub file and the load drops it.
Rationale: disjoint tiers put most guard tests in `tier0` files, which `guard_tier: 1` no longer compiles.

### D7. A tier that silently runs no tagged test fails

Under the `go` scoper, a tier run fails when a package runs zero tests while it holds a test function in a file whose constraint holds under the tier's tag on the host's GOOS and GOARCH.
The count comes from the go-json events, so a package whose untagged `TestMain` still runs counts as zero.
The failure names the tier, the package and `lyx config gate`, since the tier's command lacks its tag.
Rationale: a command that drops its tag would otherwise pass vacuously; a hub without tier tags never meets the check.

### D8. Row entry gates in shed

A recipe row may carry `entry_gate`, beside its routing.
`shedbuild` builds it into a `ProducerDef` entry-gate function over the row's checks; `shedengine` keeps its stdlib, `state` and `lock` imports.
Shed evaluates the gate when the run enters the row: after a verdict or the run's start routes to it, before the first call.
The status names the row it entered; a resume re-calls the current row without evaluating again when the names match, and a status without the field, or naming another row, reads as not entered.
Routing and the tackler's relaunch leave the named row behind, so the new current row is evaluated; `lyx shed goto` names its target as entered, so it runs without evaluation; a row rename renames it.
A false gate appends a `skipped` history entry with the check's text and the values it read, and follows the row's `on_done`, spending no bounce budget and ending the segment's episode as a Done does.
Every history reader treats `skipped` as a row passed through, never as a stop or a call; the caller label stays on the verdict that led to the skip.
Consecutive skips chain within one step; a chain longer than the producer list stops the run with an error naming the rows.
An evaluation error leaves the row not entered and stops the step as a producer error does, status non-running included, so a resume evaluates again.
Bound: a gate skips only its own row; it reads only settings through the told lookup and resolved segment budgets, and the operator's goto is the one bypass.
Rationale: operator view of 2026-10-17, one gate for every row of every recipe with no producer implementing it.

### D9. The check form

`entry_gate` is one check or a list of checks, all of which must hold.
A check is `<operand>`, `<operand> == <literal>` or `<operand> != <literal>`, with a YAML scalar literal.
An operand is `module.key`, read through `Env.Setting` over the pair's config stack, a task's `run_settings` included, or `budget.<Segment>`, the segment's resolved soft cap from `Env.Segments`.
A bare operand holds when its value is set and is not zero, `false`, an empty string or an empty list.
Recipe build refuses an unknown module or key, a segment with no `Env.Segments` entry, and a malformed check, each naming the row and the way forward.
No or, no arithmetic, no free code.
Rationale: orch recommendation; `budget.<Segment>` exists because a per-perch budget of null takes `review_max_bounces`, which a single setting cannot express.

### D10. The entry gate's consumers

- Batten's Open-Window, Seed-IDE and Open-IDE rows move their `switch:` to `entry_gate:`, and the Command producer loses its switch read and `CommandDeps.Forced`.
  `lyx batten run --window` becomes a value battencli's `Env.Setting` answers for `batten.open_window` for that run.
- Darn's edge rewrite for a Code-Review budget of 0 goes: the darn perch takes the same budget stamping as loom's, `zero_rounds` included, so its Bouncer settles approved at once.
  Darn's Describe row gets `entry_gate: budget.Code-Review`, so with a budget of 0 the writer's own change description stands, as today.

Rationale: board item 2 moves both onto the gate.
Its operator view put the darn gate on the Bouncer; this record departs from that on purpose: with `zero_rounds` darn's Code-Bouncer behaves as loom's at a budget of 0, and only Describe needs leaving out, because darn's writer writes its own description.

### D11. `lyx gate report`

`lyx gate report [--slug <slug>] [--since <date>]` reads this machine's gate run records and logs and prints, per slug and per gate key, the tier runs, carried tiers, slot wait, wall time and launch time with its share.
Launch time per package is the gap from `go test -json`'s `start` event to the package's next event, summed over the packages that ran rather than came from the cache; its share is that sum over the summed package time.
The gap holds the binary's exec, any host-scanner hold such as Cortex XDR's, its init and its `TestMain` setup; the same runs on Neve, which has no XDR, give the reference to subtract.
A record from before D3 has no key and is grouped by its tier list and scope form.
Rationale: board item 5; the records and logs already hold every figure but the key.
Evidence: `go doc cmd/test2json` at go1.26.8 defines `start` as "the test binary is about to be executed".

### D12. The tier plan per recipe

`internal/gateslot`'s doc states, per recipe, which key runs at which row with which default scope:

| Recipe | Row | Key | Scope |
|---|---|---|---|
| loom | Webster, per card | `card` | `card` |
| loom | Webster, per batch | `batch` | `card` |
| loom | Webster, verify fixers | `webster-fix` | `diff` |
| loom | Webster, end gate | `webster` | `all` |
| loom | Merge-In | `merge-in` | `all` |
| loom, darn | Code-Burler-Fix | `code-burler-fix` | `diff` |
| loom, darn | Finalize | `finalize` | `all` |
| darn | Darn-Write | `darn-write` | `diff` |

Darn and loom share the rows they share, so they share keys; the writer's end gate differs by operator decision: Webster runs whole, darn's writer runs the diff.
Rationale: the operator decision of 2026-10-23 asks for the tier plan stated plainly per recipe, and `loom-beam-darns` defers its piece tiers to it.

### D13. The transition while the old `lyx` runs the gates

This run's gates, and `main`'s until the operator deploys after landing, run the installed `lyx`, whose guard scan accepts a `//lyx:guard` marker only in a file that compiles with no tag or with `integration` alone.
So a guard-marked file takes `//go:build tierN || integration` instead of `tierN`, which that scan accepts; under the new tier commands it compiles only in `tierN` once the hub drops `integration`.
This task files a board entry, depending on this one, that removes the `|| integration` term, the scan test's allowance for it and the hub commands' old tag spellings after the deploy.
The first retagging card's gate must report the retagged packages' tests as discovered, since D7 does not exist in the installed `lyx`.
Rationale: the installed scan refuses every narrowed gate, `main`'s `--upto 2` included, from the first disjointly tagged guard file on, and only a deploy from a green `main` replaces it.

## Constraints

- The hub's tier commands must carry both tag spellings, and this hub must stop using `covers`, before Webster gates any card that retags a file; the orch applies it from the prime with `lyx config gate --set`:
  `tier0` adds `-tags tier0`, `tier1` uses `-tags tier1,integration` and vets under `tier0`, `tier1,integration`, `tier2,tmux` and `tier3,llm`, `tier2` uses `-tags tier2,tmux`, `tier3` uses `-tags tier3,llm`, and `covers` goes from every tier.
  Untagged files compile under any tag set, so the change is right for old and new trees alike; dropping `covers` costs an old tree one extra `tier0` run per gate until it merges the parent.
  Nothing in the installed `lyx` checks it; the discussion's delivery to the orch is the hand-off.
- `PATTERN-shed-producer-seam`: the entry gate reaches `shedengine` as a function on `ProducerDef`; shedengine imports nothing new.
- `PATTERN-recipe-setting-lookup`: the check reads through `Env.Setting` and `Env.Segments`; no row gets an `Env` field, and the `--window` force is a value of the lookup.
- `PATTERN-tolerant-records`: the green record, the status's entered row and the `skipped` history outcome are read tolerantly; an old status without the entered row reads as not entered.
- `PATTERN-durable-vs-ephemeral-state` and `PATTERN-lyxdirs-single-declarer`: the green record is never tracked, under `.lyx`, named through `internal/lyxdirs`.
- `PATTERN-binary-change-reconcile` and `PATTERN-hub-wide-config`: the `gates` reshape and the `guard_tier` prune run through `gateslot.MigrateConfig` in reconcile; `gate.yaml` stays hub-wide.
- `PATTERN-gate-slot`: a carried tier takes no slot because it runs nothing; every tier that runs takes its slots.
- `PATTERN-refusal-way-forward`: every new refusal names its way forward.
- `PATTERN-sole-parsers`: the recipe's `entry_gate` is parsed only in `shedbuild`.
- `PATTERN-failure-stop-visible`: an entry-gate evaluation error leaves the status non-running before an `--until-stop` waiter returns.

## Auto-mode assumptions

- "Every tier tagged by its own name" means the tag is the tier name (D5).
- Board item 3, written as a verify row's entry check, maps onto the gate's green record (D4).
- The migrated scopes are the operator's to change later in config.
- The measurement on OSL-1046 is the operator's run of `lyx gate report` there after deploy, within the records' 14-day retention, which keeps runs from before the landing; gate records are machine-local and this run executes on Neve.
- `lyx shed goto` bypassing the entry gate is the operator's force.

## Open risks

- Untagged test files leave the default gopls view once tagged `tier0`; the operator's IDE seed needs `-tags=tier0,tier1,tier2,tier3` in gopls's build flags.
- If the orch has not changed the hub's tier commands when Webster retags files, the installed `lyx`'s gates pass vacuously; D13's discovered-count check on the first retagging card is the only catch.
- Another branch landing on `main` during this run with new untagged or `integration`-tagged tests fails the one-tag scan at this run's Merge-In or Finalize gate, and the merge fixer retags them within its caps.
- A branch landing after this one with such tests fails the scan after it merges the parent, and its own fixer retags them.
- An old `lyx` on another machine refuses the reshaped `gate.yaml` until that machine deploys, since reconcile on the first deployed machine commits it hub-wide.
- Retagging touches over a thousand test files by Edit; the plan's card split decides how that cost spreads.
- `docs/benchmarks/test-suite-timing.md`'s tier figures before and after the change are not comparable, since tier1 stops running tier0's tests.
- Board entry `adopt-runs-tests` names the retired `landing` list.

## Acceptance criteria

- A hub file holding `landing`, `final`, `round`, `card`, `batch` and `guard_tier` reconciles to the D1 keys with the D1 scopes, and a second reconcile changes nothing.
- The config load refuses a `card` scope on a key other than `card` and `batch`, naming `lyx config gate`.
- `lyx gate test --gate <key>` runs the key's tiers over its scope; it refuses `--gate` with `--tier`, `--upto`, `--all`, `--base` or targets, and a `card` scope without `--card`.
- Every site named in D12 renders `--gate` with its key, a row's `gate_key` overrides its row name, and the merge fixer runs its row's key.
- A second `--gate` run over the same clean tree carries every tier, takes no slot, writes carried run records and prints carried summary lines that `gateslot`'s summary readers parse; a dirty worktree carries nothing and writes nothing.
- An `all` run after a green `diff` run over the same tree runs only the packages the `diff` run did not cover.
- A changed tier command, a different tree or args other than a `-run` filter break the carry, and a tree that changes during a run records nothing.
- A `diff` run takes its base from the newest ancestor whole-green commit, falls back to the parent merge base, and runs whole without a parent.
- The one-tag scan fails a test function in an untagged file and in a file admitting two tiers, accepts `|| integration` only on a guard-marked file, and passes the repository.
- Guard tests in `tier0` and `tier1` files run under their own tier's command; reconcile prunes `guard_tier`.
- A tier run whose command lacks its tag fails on a package holding a test for that tier, a package with an untagged `TestMain` included, and a platform-constrained file off its platform does not fail it.
- A row whose entry gate is false is recorded `skipped` with its reason, follows `on_done`, spends no budget, and a chain of skips is followed in one step.
- A resume of an entered row re-calls it without evaluating its gate, a tackler relaunch evaluates it, and `lyx shed goto` runs a gated-off row.
- The recipe build refuses an unknown setting, an unknown segment and a malformed check.
- Batten's Open-Window, Seed-IDE and Open-IDE rows skip when their setting is off, and `--window` runs Open-Window with its setting off.
- A darn run with a Code-Review budget of 0 settles its Bouncer approved at once, skips Describe and keeps the writer's description; with a budget above 0 it runs as today.
- `lyx gate report` groups old and new records per slug and key, counts carried tiers, and reports launch time from the logs' `start` events, leaving cached packages out.
- The board holds the D13 follow-up entry, depending on this task.

## PATTERN changes

- `PATTERN-test-tiers` — changes: Running hub's tests: only through `lyx gate test`; a site names its gate key, never tiers or scope.
- `PATTERN-tier-tags` — adds: Tagging a test file: one `tierN` build tag, the tier that runs it; an untagged test file holds no test. (test)
- `PATTERN-gate-carry` — adds: Skipping a test tier as already green: only `lyx gate test`'s green record, over a clean tree.
- `PATTERN-row-entry-gate` — adds: Leaving a recipe row out by config: its `entry_gate`, never a recipe rewrite or producer-side switch.

## Notes for the plan writer

- Order the cards so the one-tag scan, D6 and D7 land before or with the first retag, and plan on the Constraints' hub change being in place.
- `verifytree`'s landing failure record names its gate as a `FailureKind`; the retired kinds map onto the new keys on read.
- `gateslot.Config.TiersValue` renders the gate lists into prompts and changes with the shape.
