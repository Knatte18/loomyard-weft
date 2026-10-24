# Decision record — gate-runs-once

## Goal

Each test runs once per tree and in one tier.
Every producer that runs a test gate names its own `gate.yaml` key, which sets its tiers and its scope, save darn's tree-scoped sites, whose trees set the scope (D14).
`lyx gate test` carries a green tier forward over an unchanged tree, so a repeat on the same tree runs nothing.
Shed rows get an entry gate that replaces darn's recipe edge rewrite and the Command rows' switch.
A loom or darn run's gate time is measurable per gate key, launch latency included.

## Scope

In:

- `gate.yaml`'s `gates` reshaped to one key per producer, each with tiers and scope, and `lyx config reconcile` migrating `landing`, `final` and `round`.
- `lyx gate test --gate <key>`, the per-worktree green record, the `diff` scope's base and the run record's gate key.
- Disjoint tiers: build tags renamed to the tier names, `tier0` tagged, `guard_tier` retired, the guard step run per tier.
- Shed row entry gates, the check form, and their two consumers: batten's Command rows and darn's Describe row.
- Darn's `tree_scope_verify` retired for its rows' gate keys (D14).
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
`verifytree`'s landing failure record names the failing row's key, and a record naming the retired `landing` kind reads as the key of the row that reads it, Merge-In, Finalize or the merge fixer they spawn, since the record names no row and only those rows read it.
The template holds every key the shipped recipes read, each `{tiers: [0]}` with the scope below.
`lyx config reconcile` migrates a hub file: `landing`'s list becomes `merge-in`, `finalize`, `darn-write` and `code-burler-fix`, `final`'s becomes `webster`, `round`'s becomes `webster-fix`, and `card` and `batch` keep their lists.
Each migrated key takes the scope closest to today's call: `card` for `card` and `batch`, `all` for `webster`, `merge-in` and `finalize`, `diff` for `webster-fix`, `darn-write` and `code-burler-fix`.
`code-burler-fix`'s fallback without a recorded pass moves from today's `--all` to the parent merge base (D2).
Bound: a recipe row's `gate_key` selects only which key of the hub's `gate.yaml` it runs; the tiers and scopes stay hub-file only.
Rationale: operator decisions of 2026-10-23 and 2026-10-24; `gate_key` lets a recipe that keeps darn's row names, such as a piece recipe, take its own tiers.

### D2. Scope values

- `card`: the packages of the cards the site names.
  Only `card` and `batch` may set it; the config load refuses it on any other key, naming `lyx config gate`.
- `diff`: per listed tier, the diff from the newest commit that is an ancestor of HEAD and whose tree the green record (D4) holds whole-green for that tier under the tier's current fingerprint, with empty args and the same module directory.
  Without one, the diff from the merge base with the parent branch, which is today's computed form.
  Without a parent, the tier runs whole.
  Tiers that land on the same base share one derivation.
- `all`: the tier's `all_targets`.

Rationale: the board's two `diff` readings, since the last green run and since the fork from the parent, become one chain, the second as the fallback of the first; a narrowed green never becomes a base, so the chain stays sound.

### D3. `lyx gate test --gate <key>`

A site renders `lyx gate test --wait --gate <key>`; the `card` and `batch` sites always add `--card <id>` per card.
`lyx gate test` reads the key's tiers and scope from the config at run time and uses the card ids only under a `card` scope.
It refuses `--gate` alongside `--tier`, `--upto`, `--all` or `--base`, and a `card` scope without `--card`, naming the key.
Explicit targets beside `--gate` replace the key's scope; only darn's tree-scoped sites render them (D14).
Without `--gate` the verb keeps its manual flags as today.
The run record gains the gate key, empty for a manual run.
The fix rows' and writers' verify gate keeps its own comment-lint base: the site's latest recorded pass when it is an ancestor of HEAD, else the merge base with the parent, so a first fix round lints rather than skips.
The darn form keeps today's lint from the merge base on every attempt.
Rationale: one place resolves scope, and the record names the key the measurement groups by; per-site commands no longer share the pass of the told command, Webster's end gate, which today gives loom's fix round its lint base.

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
A tier whose carried part and passed remainder together cover every package its `all_targets` expand to writes a whole entry for the tree.
An entry with empty args covers a request with empty args and the guard step's own `-run` filter; any other args need an exact match, so `-- -count=1` reruns.
The guard step writes its entries under its own `-run` args, since its filter runs only the guard tests of its packages: an exact-args match carries a repeated guard group, and such an entry never covers an empty-args request.
Writers serialize through a lock in the same directory; pruning drops old trees but keeps each tier's 20 newest whole-green commits, so D2 finds an older ancestor when the newest is not one.
The carry sits under `verifytree`'s command-and-tree skip, which every site keeps.
Bound: a carried tier trusts a pass over the same tree, the same tier declaration and the same args on this worktree; a flaky test that passed once is not rerun, a Go toolchain change and inherited environment variables outside the tier's `env` are not in the fingerprint, ignored files such as `.dev-bin` are outside both the clean check and the fingerprint, so a change to ignored state a test reads over the same tree is carried, and the record never leaves the machine, so an adopted run tests afresh.
The record is a plain file: the standing deny refuses a Bash redirect into it, as into any project file outside `.scratch/`, but Edit, Write and Bash writers such as `cp` or `tee` reach it.
A forged or hand-edited whole entry for a tree makes every tier of every later gate on that tree carry, Merge-In and Finalize included.
This is accepted and stated in `internal/gatecli`'s doc beside the slot variable's bound: a session that can write the worktree can equally change the code and tests under review, so the record guards only honest repeats, and no further deny is added because `cp` and `tee` would still reach it.
Rationale: after `loom-tail-review` no verify row exists, since the gates run inside Merge-In, Finalize, Darn-Write, Code-Burler-Fix and Webster, whose other work must still run; a row entry gate cannot skip the gate alone, so board item 3 lands in the gate, where every site, the card and batch gates and manual runs included, gets it.

### D5. Disjoint tiers through tier-named tags

Every test file carries exactly one build tag `tierN`, the tier that runs it: `integration`, `tmux` and `llm` become `tier1`, `tier2` and `tier3`, and today's untagged test files become `tier0`.
An untagged `_test.go` file holds no `Test`, `Benchmark`, `Fuzz` or `Example` function; it holds only support such as `TestMain` and helpers.
A test file whose constraint admits two tiers, such as `tier2 || tier3`, holds no test.
The helper rules of `PATTERN-test-speed` carry over under the tier names, and a file whose helpers another tier's tests use is split.
`cmd/lyx/tierpurity_test.go` and `cmd/lyx/llmtier_test.go` are restated with the tier names; untagged becomes `tier0`.
A repository scan test enforces the one-tag rule, allowing D13's transitional term on guard-marked files only.
Every Go package load that lists test files under `integration,tmux,llm` moves to `tier0,tier1,tier2,tier3,integration,tmux,llm`, so a scope sees `tier0` test imports and, until D13's follow-up drops the old spellings, a branch not yet holding the retag still has its old-tagged test files seen: `internal/impactset`'s reverse-dependency graph and guard-package listing, `internal/planglyph`'s type load and `internal/planparser`'s module-wide check.
`cmd/testtiming`'s untagged default mode runs `-tags tier0`, its `-full` maps to `tier1`, and its spawn scan's tag classification reads `tier1`, `tier2` and `tier3` beside the old spellings.
The guards keyed on the `tmux` and `llm` tags, `internal/planparser`'s `namesGateOnlyTier` card Verify check and `internal/shellguard`'s raw go row's `-tags` alternative, match `tier2` and `tier3` beside `tmux` and `llm`.
`covers` stays a generic tier key; this hub stops using it.
Card order: D6 and D7 land before or with the first retag card, and the one-tag scan test lands with the last retag card, since the scan passes the repository only once every test file is retagged.
Every card from the first retag card on tags each test file it adds with its `tierN` tag, so the last retag card's scan meets none untagged.
Rationale: operator decision of 2026-10-22 ("every tier tagged by its own name, `tier0` included"); with tag equal to tier name, Go code maps a file to its tier with no config.

### D6. The guard step runs per tier

Under the `go` scoper a test file's tier is the `tierN` tag in its constraint, and a file with none counts as `tier0`.
The guard step groups the guard tests outside their own tier's scope by their file's tier and runs each group under that tier's command, slots, timeout and `prebuilt_lyx`.
A group whose tier the call already ran whole, or carried whole, is skipped, and a group the green record holds under its own `-run` args carries (D4).
The guard scan's `integration` rule is replaced: a `//lyx:guard` marker is allowed in `tier0` and `tier1` files and refused in `tier2` and `tier3` files.
`guard_tier` is retired: reconcile prunes it from a hub file and the load drops it.
Rationale: disjoint tiers put most guard tests in `tier0` files, which `guard_tier: 1` no longer compiles.

### D7. A tier that silently runs no tagged test fails

Under the `go` scoper, a tier run fails when a package runs zero tests while it holds a test function in a file whose constraint holds under the tier's tag on the host's GOOS and GOARCH.
The count comes from the go-json events, so a package whose untagged `TestMain` still runs counts as zero.
The failure names the tier, the package and `lyx config gate`, since the tier's command lacks its tag.
A run whose args after `--` carry `-run`, `-skip`, `-list` or `-bench` is exempt, since its filter may match nothing in a package.
The guard step's runs are not exempt: each package of a guard group holds a guard test its filter names, so a zero count there means the command lacks its tag.
Bound: a filtered run under a command that lacks its tag passes vacuously; the site gates, which pass no args, catch it.
Rationale: a command that drops its tag would otherwise pass vacuously; a hub without tier tags never meets the check.

### D8. Row entry gates in shed

A recipe row may carry `entry_gate`, beside its routing.
`shedbuild` builds it into a `ProducerDef` entry-gate function over the row's checks; `shedengine` keeps its stdlib, `state` and `lock` imports.
Shed evaluates the gate when the run enters the row: after a verdict or the run's start routes to it, before the first call.
The status names the row it entered; a resume re-calls the current row without evaluating again when the names match, and a status without the field, or naming another row, reads as not entered.
Every routed persist and every skip clears the entered row, so a row is entered only between its own entry evaluation and its verdict, and a route back into the entered row, through skipped rows or not, evaluates its gate.
Per path:
routing leaves the named row behind, so the new current row is evaluated;
a relaunch without a target, the tackler's and the arbiter's alike, writes the status unchanged, so it re-calls the entered row without evaluation, as a resume does;
a relaunch with a target, the tackler's and the arbiter's alike, moves the run without naming the target as entered, so the target is evaluated, except when Admit reports the target already in effect: the status is then written unchanged and the entered row is re-called without evaluation;
every goto through the shed goto verb, whoever calls it, the operator, the orch or the Go driver's goto seam, names its target as entered, so it runs without evaluation;
a row rename renames the entered row.
A `pause_before` naming a row pauses the run before that row's entry gate is evaluated, so a skip chain stops at a paused row and a setting changed during the pause counts;
the row is marked entered only once the pause check has passed and its gate holds.
A `pause_after` naming a row its entry gate skips fires as for a Done: the run pauses at the row the skip routes to, with the reason `paused after <row>, as requested`, and the persist clears both pause conditions as a fired condition does.
A false gate appends a `skipped` history entry with the check's text and the values it read, and follows the row's `on_done`, spending no bounce budget and ending the segment's episode as a Done does.
Every history reader treats `skipped` as a row passed through, never as a stop or a call; the caller label stays on the verdict that led to the skip.
Consecutive skips chain within one step; a chain longer than the producer list stops the run with an error naming the rows.
An evaluation error leaves the row not entered and stops the step as a producer error does, status non-running included, so a resume evaluates again.
Bound: a gate skips only its own row; it reads only settings through the told lookup and resolved segment budgets, and the shed goto verb is the one bypass.
Rationale: operator view of 2026-10-17, one gate for every row of every recipe with no producer implementing it.

### D9. The check form

`entry_gate` is one check or a list of checks, all of which must hold.
A check is `<operand>`, `<operand> == <literal>` or `<operand> != <literal>`, with a YAML scalar literal.
An operand is `module.key`, read through `Env.Setting` over the pair's config stack, a task's `run_settings` included, or `budget.<Segment>`, the segment's resolved soft cap from `Env.Segments`.
A bare `module.key` operand reads a bool and holds when it is `true`; a bare `budget.<Segment>` operand reads a number and holds when it is non-zero.
Any other type, such as the string `no` or the number `1` for a `module.key`, is an evaluation error naming the row, the operand, the value and the setting's `lyx config` command, so the Command producer's refusal of a non-bool switch survives the move.
Recipe build refuses an unknown module or key, a segment with no `Env.Segments` entry, and a malformed check, each naming the row and the way forward.
The build checks a `module.key` operand against the config registry's module templates, the known-key check `configreg.SettingLookup` makes, never through `Env.Setting`, so batten's build needs no task worktree;
the caller tells that check as a recipe-wide `Env` function beside `Env.Setting`, so `shedrecipe` and `shedbuild` import nothing new; `Env.Setting` serves the evaluation when the row is entered.
No or, no arithmetic, no free code.
Rationale: orch recommendation; `budget.<Segment>` exists because a per-perch budget of null takes `review_max_bounces`, which a single setting cannot express.

### D10. The entry gate's consumers

- Batten's Open-Window, Seed-IDE and Open-IDE rows move their `switch:` to `entry_gate:`, and the Command producer loses its switch read and `CommandDeps.Forced`.
  `lyx batten run --window` opens a window running `lyx batten run --open-terminal`, and that inner run's hidden `--open-terminal` flag is what battencli's `Env.Setting` answers `batten.open_window` true for, for that run.
- Darn's edge rewrite for a Code-Review budget of 0 goes: the darn perch takes the same budget stamping as loom's, `zero_rounds` included, so its Bouncer settles approved at once.
  Darn's Describe row gets `entry_gate: budget.Code-Review`, so with a budget of 0 the writer's own change description stands, as today.

Rationale: board item 2 moves both onto the gate.
Its operator view put the darn gate on the Bouncer; this record departs from that on purpose: with `zero_rounds` darn's Code-Bouncer behaves as loom's at a budget of 0, and only Describe needs leaving out, because darn's writer writes its own description.

### D11. `lyx gate report`

`lyx gate report [--slug <slug>] [--since <date>]` reads this machine's gate run records and prints, per slug and per gate key, the tier runs, carried tiers, slot wait, wall time and launch time with its share.
`lyx gate test` keeps, per go-json tier run, a per-package launch list in the run record: for each package whose test binary ran, so neither from the cache nor without test files, the gap from `go test -json`'s `start` event to the package's next event, and the package's elapsed time from its closing event.
The go-json counter reads both from the package-level events, decoding their `Package`, `Time` and `Elapsed` fields beside the `Action` it decodes today, and a package whose output reports `(cached)` or `[no test files]` is left out; the rendered log stays the events' output only.
Launch time is the sum of a run's gaps; its share is that sum over the summed package time.
The gap holds the binary's exec, any host-scanner hold such as Cortex XDR's, its init and its `TestMain` setup; the same runs on Neve, which has no XDR, give the reference to subtract.
A record from before D3 has no key and is grouped by its tier list and scope form.
A record written before this task carries no launch list, so the report shows launch time only for the records that hold one.
Rationale: board item 5; the records hold the slot wait start, the start, the end and the counts per tier run, and this task adds the gate key and the launch list.
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
| darn | Darn-Write, Code-Burler-Fix and Finalize, diff wholly under `tree_scope` | that row's key | the touched trees' targets (D14) |

Darn and loom share the rows they share, so they share keys; the writer's end gate differs by operator decision: Webster runs whole, darn's writer runs the diff.
Rationale: the operator decision of 2026-10-23 asks for the tier plan stated plainly per recipe, and `loom-beam-darns` defers its piece tiers to it.

### D13. The transition while the old `lyx` runs the gates

This run's gates, and `main`'s until the operator deploys after landing, run the installed `lyx`, whose guard scan accepts a `//lyx:guard` marker only in a file that compiles with no tag or with `integration` alone.
So a guard-marked file takes `//go:build tierN || integration` instead of `tierN`, which that scan accepts; under the new tier commands it compiles only in `tierN` once the hub drops `integration`.
This task files a board entry, depending on this one, that removes the `|| integration` term, the scan test's allowance for it, and the old tag spellings in the hub commands, in D5's package loads, in `cmd/testtiming`'s spawn scan and in the two guards D5 extends after the deploy.
The installed `lyx` also lists packages under `integration,tmux,llm`, so a retagged `tier0` test file drops out of its impact graph:
its narrowed gates, `--base` and computed alike, miss a package reached only through a `tier0` test file's import.
Bound: this run's whole gates, Webster's end gate and Merge-In's landing gate, both over `--all`, and `main`'s `--upto 2`, where no parent makes the scope whole, run every package and catch it; after the deploy D5's loads, which list both spellings, close the gap.
The plan's first retagging card carries a `**Verify:**` line, which `lyx gate card-verify` runs before the card lands, that fails unless each of the hub's tier commands in `lyx config gate --resolved` names its `tierN` tag, and fails when any tier there still carries a `covers` entry, since D7 does not exist in the installed `lyx`.
The `covers` check is needed because the installed `lyx` drops a covered tier from a call, and `tier1`'s `-tags tier1,integration` no longer compiles `tier0`-tagged files, so a kept `covers: [0]` leaves the retagged `tier0` tests in no gate.
When it fails, the way forward is the Constraints' hub change, applied by the orch, and then a rerun of the card.
Rationale: the installed scan refuses every narrowed gate, `main`'s `--upto 2` included, from the first disjointly tagged guard file on, and only a deploy from a green `main` replaces it.

### D14. Darn's tree-scoped verify takes its row's key

`darn.yaml`'s `tree_scope_verify` is retired; reconcile prunes it from a hub file, and the load drops it.
Tree scope is then on exactly when `tree_scope` holds a tree, and removing every tree turns it off, and `ErrTreeScopeWithoutTrees` goes.
The migration fires only while the hub file still holds the `tree_scope_verify` key: held empty, reconcile empties `tree_scope` and prunes the key, keeping an off hub off; held set, it prunes the key and keeps `tree_scope`; absent, it changes nothing, so a later reconcile never touches trees the operator adds.
A hub whose file never wrote `tree_scope_verify` but holds trees turns tree scope on after the migration.
For a diff wholly under `tree_scope`'s trees, Darn-Write, darn's Code-Burler-Fix and darn's Finalize render `lyx gate test --wait --gate <key> <targets>`, with the row's key and the touched trees' targets from `tree_scope` in place of the key's scope.
When no touched tree contributes a target, the site renders no call and runs nothing, as today's empty call does.
The `lyx config darn` refusal that asks to clear `tree_scope_verify` before emptying `tree_scope` goes with the key.
Bound: the tiers of a tree-scoped run are the row key's, not an operator-written command's.
Rationale: a tree-scoped run then carries its site's key into the run record and the report, and `tree_scope` keeps narrowing a diff to the trees it touched.

### Added after Discussion (parent, 2026-10-24): Shorten test-tiers for PATTERN headroom

Decision: Shorten PATTERN-test-tiers to: Running hub's tests: only via `lyx gate test`; a site names its gate key, never tiers or scope, bar darn's tree-scoped targets.

Rationale: PATTERN.md is already over its headroom line and this task's entries add 510 bytes; trimming detail the background files and code already hold makes room until a PATTERN cleanup (operator, 2026-10-24).

### Added after Discussion (parent, 2026-10-24): Shorten command-row for PATTERN headroom

Decision: Shorten PATTERN-command-row to: Adding side-action row to a recipe: a `Command` row over config keys, its command from template and board default only.

Rationale: PATTERN.md is already over its headroom line and this task's entries add 510 bytes; trimming detail the background files and code already hold makes room until a PATTERN cleanup (operator, 2026-10-24).

### Added after Discussion (parent, 2026-10-24): Shorten shed-recipe-registry for PATTERN headroom

Decision: Shorten PATTERN-shed-recipe-registry to: Adding producer: one `map[string]Constructor` in `internal/shedrecipe` behind `Lookup`, bar `Implementer`'s.

Rationale: PATTERN.md is already over its headroom line and this task's entries add 510 bytes; trimming detail the background files and code already hold makes room until a PATTERN cleanup (operator, 2026-10-24).

### Added after Discussion (parent, 2026-10-24): Shorten trimpath-safe-root for PATTERN headroom

Decision: Shorten PATTERN-trimpath-safe-root to: Locating module root in gate-built code: never from a `runtime.Caller` path alone.

Rationale: PATTERN.md is already over its headroom line and this task's entries add 510 bytes; trimming detail the background files and code already hold makes room until a PATTERN cleanup (operator, 2026-10-24).

### Added after Discussion (parent, 2026-10-24): Shorten role-skills-typed for PATTERN headroom

Decision: Shorten PATTERN-role-skills-typed to: Loading spawned session's skill: spawner names it on launch spec and lyx delivers it; no stencil asks. (test)

Rationale: PATTERN.md is already over its headroom line and this task's entries add 510 bytes; trimming detail the background files and code already hold makes room until a PATTERN cleanup (operator, 2026-10-24).

### Added after Discussion (parent, 2026-10-24): Shorten template-overlay for PATTERN headroom

Decision: Shorten PATTERN-template-overlay to: Judging config document against its template: only `configengine`'s effective template.

Rationale: PATTERN.md is already over its headroom line and this task's entries add 510 bytes; trimming detail the background files and code already hold makes room until a PATTERN cleanup (operator, 2026-10-24).

### Added after Discussion (parent, 2026-10-24): Shorten shell-guard-table for PATTERN headroom

Decision: Shorten PATTERN-shell-guard-table to: Guarding shell command: only in `internal/shellguard`, every engine rendering every row.

Rationale: PATTERN.md is already over its headroom line and this task's entries add 510 bytes; trimming detail the background files and code already hold makes room until a PATTERN cleanup (operator, 2026-10-24).

### Added after Discussion (parent, 2026-10-24): Shorten lyxdirs-single-declarer for PATTERN headroom

Decision: Shorten PATTERN-lyxdirs-single-declarer to: Naming `_lyx` or `.lyx` in a path: only `internal/lyxdirs`, bar a `tools/` script's variable block.

Rationale: PATTERN.md is already over its headroom line and this task's entries add 510 bytes; trimming detail the background files and code already hold makes room until a PATTERN cleanup (operator, 2026-10-24).

### Added after Discussion (parent, 2026-10-24): Shorten treadle-runner-seam for PATTERN headroom

Decision: Shorten PATTERN-treadle-runner-seam to: Importing into `internal/treadleengine`: allowlist only, never `burlerengine`, `lyxcwd` or a CLI package.

Rationale: PATTERN.md is already over its headroom line and this task's entries add 510 bytes; trimming detail the background files and code already hold makes room until a PATTERN cleanup (operator, 2026-10-24).

### Added after Discussion (parent, 2026-10-24): Shorten binary-change-reconcile for PATTERN headroom

Decision: Shorten PATTERN-binary-change-reconcile to: Reconciling a hub's config after a binary change: only via `hubreconcile.Ensure`.

Rationale: PATTERN.md is already over its headroom line and this task's entries add 510 bytes; trimming detail the background files and code already hold makes room until a PATTERN cleanup (operator, 2026-10-24).

### Added after Discussion (parent, 2026-10-24): Shorten shed-producer-seam for PATTERN headroom

Decision: Shorten PATTERN-shed-producer-seam to: Importing into `internal/shedengine`: only stdlib, `state` and `lock`; paths are told.

Rationale: PATTERN.md is already over its headroom line and this task's entries add 510 bytes; trimming detail the background files and code already hold makes room until a PATTERN cleanup (operator, 2026-10-24).

### Added after Discussion (parent, 2026-10-24): Shorten parent-lock for PATTERN headroom

Decision: Shorten PATTERN-parent-lock to: Moving, merging into or landing on a local parent branch: only in Finalize, under its parent lock.

Rationale: PATTERN.md is already over its headroom line and this task's entries add 510 bytes; trimming detail the background files and code already hold makes room until a PATTERN cleanup (operator, 2026-10-24).

### Added after Discussion (parent, 2026-10-24): Shorten agent-filed-issues for PATTERN headroom

Decision: Shorten PATTERN-agent-filed-issues to: Filing GitHub issue: only `lyx selfreport create` in production. (test)

Rationale: PATTERN.md is already over its headroom line and this task's entries add 510 bytes; trimming detail the background files and code already hold makes room until a PATTERN cleanup (operator, 2026-10-24).

### Added after Discussion (parent, 2026-10-24): Shorten fabric-write-containment for PATTERN headroom

Decision: Shorten PATTERN-fabric-write-containment to: Writing `_launchers` or workspace file in `fabricengine`: via a hub `os.Root` only.

Rationale: PATTERN.md is already over its headroom line and this task's entries add 510 bytes; trimming detail the background files and code already hold makes room until a PATTERN cleanup (operator, 2026-10-24).

### Added after Discussion (parent, 2026-10-24): Shorten origin-reconcile for PATTERN headroom

Decision: Shorten PATTERN-origin-reconcile to: Acting on a run or hub after an absence: `fabricengine`'s origin reconcile first.

Rationale: PATTERN.md is already over its headroom line and this task's entries add 510 bytes; trimming detail the background files and code already hold makes room until a PATTERN cleanup (operator, 2026-10-24).

### Added after Discussion (parent, 2026-10-24): Shorten never-force-add for PATTERN headroom

Decision: Shorten PATTERN-never-force-add to: Keeping transients unindexed: `.git/info/exclude`; never `git add -f`.

Rationale: PATTERN.md is already over its headroom line and this task's entries add 510 bytes; trimming detail the background files and code already hold makes room until a PATTERN cleanup (operator, 2026-10-24).

### Added after Discussion (parent, 2026-10-24): Shorten recipe-owns-failures for PATTERN headroom

Decision: Shorten PATTERN-recipe-owns-failures to: A merge or test failure at Merge-In or Finalize: its fixer, a reroute or a halt to the orch.

Rationale: PATTERN.md is already over its headroom line and this task's entries add 510 bytes; trimming detail the background files and code already hold makes room until a PATTERN cleanup (operator, 2026-10-24).

### Added after Discussion (parent, 2026-10-24): Shorten hub-store-gc for PATTERN headroom

Decision: Shorten PATTERN-hub-store-gc to: Hub stores run no auto gc: only `fabricengine/housekeeping.go` sets it or runs `gc`.

Rationale: PATTERN.md is already over its headroom line and this task's entries add 510 bytes; trimming detail the background files and code already hold makes room until a PATTERN cleanup (operator, 2026-10-24).

### Added after Discussion (parent, 2026-10-24): Shorten owner-machine for PATTERN headroom

Decision: Shorten PATTERN-owner-machine to: Starting, resuming or landing a run or acting on wake: only where no other machine owns it.

Rationale: PATTERN.md is already over its headroom line and this task's entries add 510 bytes; trimming detail the background files and code already hold makes room until a PATTERN cleanup (operator, 2026-10-24).

### Added after Discussion (parent, 2026-10-24): Shorten pane-binary-resolution for PATTERN headroom

Decision: Shorten PATTERN-pane-binary-resolution to: Starting `lyx` in a reed pane or window: resolve it via `panebin.go` only. (test)

Rationale: PATTERN.md is already over its headroom line and this task's entries add 510 bytes; trimming detail the background files and code already hold makes room until a PATTERN cleanup (operator, 2026-10-24).

## Constraints

- The hub's tier commands must carry both tag spellings, and this hub must stop using `covers`, before Webster gates any card that retags a file; the orch applies it from the prime with `lyx config gate --set`:
  `tier0` adds `-tags tier0`, `tier1` uses `-tags tier1,integration` and vets under `tier0`, `tier1,integration`, `tier2,tmux` and `tier3,llm`, `tier2` uses `-tags tier2,tmux`, `tier3` uses `-tags tier3,llm`, and `covers` goes from every tier.
  Untagged files compile under any tag set, so these tier commands are right for old and new trees alike, though the installed `lyx`'s package loads are not (D13); dropping `covers` costs an old tree one extra `tier0` run per gate until it merges the parent.
  Nothing in the installed `lyx` checks it; the discussion's delivery to the orch is the hand-off, and D13's `**Verify:**` line on the first retagging card catches a hub that lacks a tag or keeps a `covers` entry.
- `PATTERN-shed-producer-seam`: the entry gate reaches `shedengine` as a function on `ProducerDef`; shedengine imports nothing new.
- `PATTERN-recipe-setting-lookup`: the check reads through `Env.Setting` and `Env.Segments`, and the build's known-key check through the told template check (D9); no row gets an `Env` field, and the `--window` force is a value of the lookup.
- `PATTERN-tolerant-records`: the green record, the run record's gate key and launch list, the status's entered row and the `skipped` history outcome are read tolerantly; an old status without the entered row reads as not entered.
- `PATTERN-durable-vs-ephemeral-state` and `PATTERN-lyxdirs-single-declarer`: the green record is never tracked, under `.lyx`, named through `internal/lyxdirs`.
- `PATTERN-binary-change-reconcile` and `PATTERN-hub-wide-config`: the `gates` reshape and the `guard_tier` prune run through `gateslot.MigrateConfig` in reconcile, and the `tree_scope_verify` prune through darn's config module's migration (D14); `gate.yaml` and `darn.yaml` stay hub-wide.
- `PATTERN-gate-slot`: a carried tier takes no slot because it runs nothing; every tier that runs takes its slots.
- `PATTERN-refusal-way-forward`: every new refusal names its way forward.
- `PATTERN-sole-parsers`: the recipe's `entry_gate` is parsed only in `shedbuild`.
- `PATTERN-failure-stop-visible`: an entry-gate evaluation error leaves the status non-running before an `--until-stop` waiter returns.

## Auto-mode assumptions

- "Every tier tagged by its own name" means the tag is the tier name (D5).
- Board item 3, written as a verify row's entry check, maps onto the gate's green record (D4).
- The migrated scopes are the operator's to change later in config.
- The measurement on OSL-1046 is the operator's run of `lyx gate report` there after deploy, within the records' 14-day retention, which keeps runs from before the landing; gate records are machine-local and this run executes on Neve.
  The before side, from pre-landing records, gives slot wait and wall time per tier list and scope form; launch time and its share come from runs after the deploy only.
- A goto through the shed goto verb bypassing the entry gate is a deliberate force, whoever calls it.

## Open risks

- Untagged test files leave the default gopls view once tagged `tier0`; the operator's IDE seed needs `-tags=tier0,tier1,tier2,tier3` in gopls's build flags.
- If the orch has not changed the hub's tier commands, or has left a `covers` entry, when Webster retags files, the installed `lyx`'s gates pass vacuously or run the retagged `tier0` tests in no gate; D13's `**Verify:**` line on the first retagging card, which checks both, is the only catch.
- Another branch landing on `main` during this run with new untagged or `integration`-tagged tests fails the one-tag scan at this run's Merge-In or Finalize gate, and the merge fixer retags them within its caps.
- A branch landing after this one with such tests fails the scan after it merges the parent, and its own fixer retags them.
- An old `lyx` on another machine refuses the reshaped `gate.yaml` until that machine deploys, since reconcile on the first deployed machine commits it hub-wide.
- Retagging touches over a thousand test files by Edit; the plan's card split decides how that cost spreads.
- `docs/benchmarks/test-suite-timing.md`'s tier figures before and after the change are not comparable, since tier1 stops running tier0's tests.
- Board entry `adopt-runs-tests` names the retired `landing` list.

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/plan/round-3-review.md; fixer report: _lyx/reviews/plan/round-3-fixer-report.md.
  - card-15-extends-should-be-edit (class consistency, severity MEDIUM, open in rounds 3)
  - card-17-extends-should-be-edit (class consistency, severity MEDIUM, open in rounds 3)
  - card-20-duplicate-forced-answer-test (class consistency, severity MEDIUM, open in rounds 2, 3)
  - hand-enumerated-affected-sets (class design, severity BLOCKING, open in rounds 1, 2, 3)
<!-- /lyx:carry-over Plan-Review -->

<!-- lyx:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/code/round-1-review.md; fixer report: _lyx/reviews/code/round-1-fixer-report.md.
  - relaunch-entered-row-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Code-Review -->

## Acceptance criteria

- A hub file holding `landing`, `final`, `round`, `card`, `batch` and `guard_tier` reconciles to the D1 keys with the D1 scopes, and a second reconcile changes nothing.
- The config load refuses a `card` scope on a key other than `card` and `batch`, naming `lyx config gate`.
- `lyx gate test --gate <key>` runs the key's tiers over its scope; it refuses `--gate` with `--tier`, `--upto`, `--all` or `--base`, and a `card` scope without `--card`, and explicit targets replace the key's scope.
- Every site named in D12 renders `--gate` with its key, a row's `gate_key` overrides its row name, and the merge fixer runs its row's key.
- A second `--gate` run over the same clean tree carries every tier, takes no slot, writes carried run records and prints carried summary lines that `gateslot`'s summary readers parse; a dirty worktree carries nothing and writes nothing.
- An `all` run after a green `diff` run over the same tree runs only the packages the `diff` run did not cover.
- A guard-step pass writes its entry under its own `-run` args, so an `all` run after it on the same tree runs the guard packages' tests, and a repeat of the guard group carries.
- A changed tier command or a different tree breaks the carry, any args other than the guard step's own `-run` filter break it unless they match the entry exactly, and a tree that changes during a run records nothing.
- A `diff` run takes its base from the newest ancestor whole-green commit, falls back to the parent merge base, and runs whole without a parent; a whole entry recorded under `-- -run X` or under a since-changed tier command is never taken as a base.
- A `diff` scope includes a package reached only through a `tier0` test file's import.
- Darn's tree-scoped sites render `--gate` with their row's key and the touched trees' targets.
- Reconcile of a hub file holding `tree_scope_verify` empty prunes it and empties `tree_scope`; holding it set, prunes it and keeps `tree_scope`; without it but with trees, keeps the trees; and a second reconcile of each changes nothing.
- The one-tag scan fails a test function in an untagged file and in a file admitting two tiers, accepts `|| integration` only on a guard-marked file, and passes the repository.
- Guard tests in `tier0` and `tier1` files run under their own tier's command; reconcile prunes `guard_tier`.
- A card Verify line and a raw `go test` naming `-tags tier2` or `-tags tier3` are refused, as with `tmux` and `llm`.
- A tier run whose command lacks its tag fails on a package holding a test for that tier, a package with an untagged `TestMain` included, and neither a platform-constrained file off its platform nor a `-run` filter that misses a package fails it.
- A row whose entry gate is false is recorded `skipped` with its reason, follows `on_done`, spends no budget, and a chain of skips is followed in one step.
- A resume of an entered row re-calls it without evaluating its gate, a tackler or arbiter relaunch with a goto target evaluates the target while one without a target, or with a target already in effect, re-calls the entered row, and `lyx shed goto` runs a gated-off row.
- A Stuck routed back to its own row through a skipped row evaluates that row's gate.
- A `pause_before` on a gated row halts the run before the gate is evaluated, and a setting changed during the pause decides the gate.
- A `pause_after` on a skipped row pauses the run at that row's `on_done` row.
- A bare `module.key` operand holding a non-bool, such as `no` or `1`, stops the step with an evaluation error.
- A recipe build with an entry gate on a `module.key` operand needs no task worktree.
- The recipe build refuses an unknown setting, an unknown segment and a malformed check.
- Batten's Open-Window, Seed-IDE and Open-IDE rows skip when their setting is off, and `--window` runs Open-Window with its setting off.
- A darn run with a Code-Review budget of 0 settles its Bouncer approved at once, skips Describe and keeps the writer's description; with a budget above 0 it runs as today.
- A go-json tier run's record holds a launch entry per package whose test binary ran, and none for a cached package or one without test files.
- `lyx gate report` groups old and new records per slug and key, counts carried tiers, reports launch time from the records' launch lists, and shows no launch time for a record without one.
- The board holds the D13 follow-up entry, depending on this task.

## PATTERN changes

- `PATTERN-test-tiers` — changes: Running hub's tests: only via `lyx gate test`; a site names its gate key, never tiers or scope, bar darn's tree-scoped targets. — added after Discussion (parent, 2026-10-24)
- `PATTERN-tier-tags` — adds: Tagging a test file: one `tierN` build tag, the tier that runs it; an untagged test file holds no test. (test)
- `PATTERN-gate-carry` — adds: Skipping a test tier as already green: only `lyx gate test`'s green record, over a clean tree.
- `PATTERN-row-entry-gate` — adds: Leaving a recipe row out by config: its `entry_gate`, never a recipe rewrite or producer-side switch.
- `PATTERN-command-row` — changes: Adding side-action row to a recipe: a `Command` row over config keys, its command from template and board default only. — added after Discussion (parent, 2026-10-24)
- `PATTERN-shed-recipe-registry` — changes: Adding producer: one `map[string]Constructor` in `internal/shedrecipe` behind `Lookup`, bar `Implementer`'s. — added after Discussion (parent, 2026-10-24)
- `PATTERN-trimpath-safe-root` — changes: Locating module root in gate-built code: never from a `runtime.Caller` path alone. — added after Discussion (parent, 2026-10-24)
- `PATTERN-role-skills-typed` — changes: Loading spawned session's skill: spawner names it on launch spec and lyx delivers it; no stencil asks. (test) — added after Discussion (parent, 2026-10-24)
- `PATTERN-template-overlay` — changes: Judging config document against its template: only `configengine`'s effective template. — added after Discussion (parent, 2026-10-24)
- `PATTERN-shell-guard-table` — changes: Guarding shell command: only in `internal/shellguard`, every engine rendering every row. — added after Discussion (parent, 2026-10-24)
- `PATTERN-lyxdirs-single-declarer` — changes: Naming `_lyx` or `.lyx` in a path: only `internal/lyxdirs`, bar a `tools/` script's variable block. — added after Discussion (parent, 2026-10-24)
- `PATTERN-treadle-runner-seam` — changes: Importing into `internal/treadleengine`: allowlist only, never `burlerengine`, `lyxcwd` or a CLI package. — added after Discussion (parent, 2026-10-24)
- `PATTERN-binary-change-reconcile` — changes: Reconciling a hub's config after a binary change: only via `hubreconcile.Ensure`. — added after Discussion (parent, 2026-10-24)
- `PATTERN-shed-producer-seam` — changes: Importing into `internal/shedengine`: only stdlib, `state` and `lock`; paths are told. — added after Discussion (parent, 2026-10-24)
- `PATTERN-parent-lock` — changes: Moving, merging into or landing on a local parent branch: only in Finalize, under its parent lock. — added after Discussion (parent, 2026-10-24)
- `PATTERN-agent-filed-issues` — changes: Filing GitHub issue: only `lyx selfreport create` in production. (test) — added after Discussion (parent, 2026-10-24)
- `PATTERN-fabric-write-containment` — changes: Writing `_launchers` or workspace file in `fabricengine`: via a hub `os.Root` only. — added after Discussion (parent, 2026-10-24)
- `PATTERN-origin-reconcile` — changes: Acting on a run or hub after an absence: `fabricengine`'s origin reconcile first. — added after Discussion (parent, 2026-10-24)
- `PATTERN-never-force-add` — changes: Keeping transients unindexed: `.git/info/exclude`; never `git add -f`. — added after Discussion (parent, 2026-10-24)
- `PATTERN-recipe-owns-failures` — changes: A merge or test failure at Merge-In or Finalize: its fixer, a reroute or a halt to the orch. — added after Discussion (parent, 2026-10-24)
- `PATTERN-hub-store-gc` — changes: Hub stores run no auto gc: only `fabricengine/housekeeping.go` sets it or runs `gc`. — added after Discussion (parent, 2026-10-24)
- `PATTERN-owner-machine` — changes: Starting, resuming or landing a run or acting on wake: only where no other machine owns it. — added after Discussion (parent, 2026-10-24)
- `PATTERN-pane-binary-resolution` — changes: Starting `lyx` in a reed pane or window: resolve it via `panebin.go` only. (test) — added after Discussion (parent, 2026-10-24)

## Notes for the plan writer

- Plan on the Constraints' hub change being in place.
- `gateslot.Config.TiersValue` renders the gate lists into prompts and changes with the shape.
