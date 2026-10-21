# Support log — test-tiers

## Interview

### Turn 1 — exploration (autonomous, no operator)

Read the board entry and the entries it names (`dotnet-verify-protocol`, `regression-baseline-verify`, `dotnet-scoped-verify`, `gate-status`) plus `loom-gates-xdr-stacking`, `verify-watchdog`, `tuned-defaults-template`, `darn-verify-fixes`, `run-process-ownership`, `spawn-audit-log` and `lenses-language-sets`.
Read the gate, slot, impactset, verifytree, languageconfig, loomshed and webster package docs, the gate config and its template, the shell guard's raw-go row, the configengine `Spec`, the plan spec's verify model and the two pattern backgrounds that describe gates.
Found: every Go verify site already calls `lyx gate test` inside its command, but the commands are spelled by the plan writer (`## verify:`), by `darn.yaml`, by `landing.yaml`'s `publish_verify` and by impactset; recent plans spell a long package list; `verifytree` holds a slot and the inner gate inherits it; the template's `go_parallel` became a machine key in a recent commit.

### Turn 2 — design batch 1: where tiers live and how sites name them (auto-picks)

- Q: tiers in `gate.yaml` or `language.yaml`? Recommended `gate.yaml`, since it owns slots and has the machine layer. Picked.
- Q: a `--gate <name>` flag, or sites rendering resolved `--tier` lists? Recommended rendering lists, so two sites with the same list share one verified-tree entry. Picked.
- Q: keep the plan's `## verify:`? Recommended retiring it as a gate input while the parser still accepts it. Picked.

### Turn 3 — design batch 2: slots, timeouts, counts (auto-picks)

- Q: keep `verifytree`'s slot with inheritance, or move slots into the gate per tier? Recommended per tier, since a multi-slot tier inside an inherited single slot cannot be sequenced. Picked.
- Q: how to stop a heavy tier being starved? Recommended a drain lock that single-slot acquires wait behind, with ordered acquisition for multi-slot tiers. Picked.
- Q: how to count tests without hard-coding a command? Recommended a closed set of named counters. Picked.
- Q: zero tests on a scoped Go run? Recommended failing only when a target held test files, so test-less and tag-only packages do not fail card gates. Picked.

### Turn 4 — design batch 3: scope, guards, refusal, logs, hubs (auto-picks)

- Q: diff to HEAD or to the working tree? Recommended the working tree, since card gates run before the commit. Picked.
- Q: where do guard tests run? Recommended a `guard_tier` under the `go` scoper. Picked.
- Q: a static raw-test row or a hub list? Recommended the hub list `refuse_raw`. Picked.
- Q: where do run logs survive? Recommended the board's ephemeral gate directory with age pruning. Picked.
- Q: who writes hub values? Recommended the orch after deploy, with loomyard's block as the doc example. Picked.

### Turn 5 — lens review (seven forks, all returned)

- template fails its own validation (walkthrough, bound-breaker, interactions; BLOCKING) — fixed: cross-references checked at use, D1.
- `final` scoped against an operator decision (goal-scope, board-fit; BLOCKING) — fixed: `final` and `landing` run `--all`, D8.
- filtered whole run fails the floor (bound-breaker; BLOCKING) — fixed: whole means unfiltered, D7.
- `TestMain`-only packages fail narrower runs (walkthrough) — fixed: narrower runs warn on zero, D7.
- zero-test rule weakened without naming the departure (goal-scope) — fixed: departure and bound stated, D7.
- narrower `vstest` run undefined (interactions, bound-breaker) — fixed by the same D7 rule.
- round base is `landing` though Webster passes `final` (walkthrough) — fixed: loom's base is `final`, with fallback, D8.
- empty gate lists verify nothing (interactions, bound-breaker) — fixed: only `publish` may be empty, D1.
- `tree_scope_verify` unslotted (interactions, bound-breaker, leanness) — fixed: must start with `lyx gate test`; doc example moves to `--tier`, D9.
- card `**Verify:**` runner keeps its own slot and `Lease.Env` reads `go_parallel` (existing-paths) — fixed: runner drops its slot, `Lease.Env` carries only inheritance, plan gate refuses raw prefixes in `Verify:`, D6, D8, D10.
- pre-tier hangs unbounded once the 60-minute ceiling goes (bound-breaker) — fixed: fixed 10-minute ceiling before the first tier, D2.
- lifted-cap flag unbounded and missing from synopsis (bound-breaker) — fixed: `--wait` in synopsis with bound, D2.
- per-module timeout and slot scope undefined (walkthrough) — fixed, D2 and D3.
- `-C` vs `dir` (walkthrough, bound-breaker) — fixed, D2.
- `guard_tier` command placeholders unchecked (interactions) — fixed, D5.
- loomyard tier block incomplete (walkthrough, interactions, goal-scope) — fixed: per-tier common keys stated, timeouts and floors derived from measurement, D12.
- tier0 build and vet add card-gate cost (goal-scope) — fixed: tier0 is the `go test` alone, D12.
- prebuild cap (existing-paths) — fixed: tier's `parallel`, inside its timeout, D1.
- old Publish failure record kinds (existing-paths, interactions) — fixed: mapped, D8.
- plan-level `## verify:` checks still refuse (existing-paths) — fixed: retired, D9.
- wait record vs marker (interactions) — fixed: loom status reads the record first, D6.
- `refuse_raw` unvalidated and spawn-time (bound-breaker, existing-paths) — fixed: word shape validated, bound stated, D1 and D10.
- call-time vs render-time wording (bound-breaker) — fixed, D8.
- inherited slot bypasses heavy-tier sequencing unstated (leanness) — fixed: bound stated, D6.
- guard step cost (bound-breaker) — fixed: stated, D5.
- dependencies on `darn-verify-fixes`, `tender-pause-holds-reprompt`, `gate-status`, Wanted 5, `spawn-audit-log`, `tuned-defaults-template`, `origin-reconcile` (board-fit) — fixed: named in Scope or Open risks, tier added to the holder record.
- Required 3 wall-time bullet homeless (goal-scope) — fixed: stays in note `dotnet-verify-protocol`.
- Scope, Constraints, Auto-mode, D2 and D7 restate other text (leanness) — fixed: trimmed.
- garbled rejected-alternative bullet (leanness) — fixed.
- move NORCE Models values out of the record (leanness) — rejected: the board names both hubs, and the orch applies them from this record.
- `about` unlicensed (goal-scope) — rejected: it replaces the rendering `verify_tiers` gave the prompts.

## Rejected alternatives

- Tiers in `language.yaml`: it has no machine layer for per-machine parallelism and does not own the slots.
- A `--gate <name>` CLI flag: two sites with identical lists would render different commands and miss each other's verified-tree entry.
- Keeping `verifytree`'s slot and having the inner gate inherit it: a tier asking for more slots or the whole machine could not acquire them inside an inherited slot without deadlock.
- Regex counters in config: Go prints no counts without `-v` or `-json`, so a Go hub needs a parser that also renders the output; a named counter is simpler and honest about being format knowledge.
- Failing every zero-test run: a card targeting a test-less or integration-only package would fail its untagged card gate.
- A Go fallback when no tier is declared: it would keep `go test` hard-coded in lyx.
- Keeping `--tags`: it is Go syntax on a language-neutral verb, and `--tier` covers it.
- Logs under the weft, tracked: full test output would bloat the records repository.
- Listing wall times in the change description: Describe runs before the landing gates.
- Keeping darn's `verify` as a hand-spelled command: the `landing` list now states it.
- Scoping `final` and loom's `landing` to the diff against the parent: it overrides the operator decision in note `loom-gates-xdr-stacking` that Webster's plan-level gate runs the whole suite.
- Letting a narrower run that discovers zero pass with a warning: the parent rejected it in round 1; superseded by the operator's 2026-10-21 decision that every run discovering zero passes.
- Checking gate-list cross-references at config load: the template, whose gate lists name `tier0` while it declares no tier, would fail to load.
- Keeping a `verifytree` slot for non-gate commands: requiring every command it runs to be one `lyx gate test --wait` call, or such calls joined by `&&`, leaves it nothing to slot.
- Refusing every card `**Verify:**` line that is not a `lyx gate test` call: the plan gate cannot tell a build or test from another check in static text, and the line is the escape hatch for checks a tier cannot express.
- Reporting a zero-discovery confirm-the-fix run as skipped: the operator's decision leaves no skipped outcome.
- Moving the NORCE Models values out of the record: the board names both hubs, and the record is what the orch applies.

## Review rounds

### Parent review, round 1 (ly:orch, BLOCKING)

- F1 (BLOCKING, D7 and its acceptance criterion: a narrower run discovering zero passed with a warning) — fixed: every run that discovers zero fails, filtered runs included; the scoper drops targets with no tests for the tier, `TestMain` alone counting as none, using a new `go`-scoper tier key `build_tags`; an empty remaining scope is skipped with nothing to test, logged as its own outcome; the departure sentence and the Bound paragraph are gone; D1, D3, D11, D12 and the acceptance criterion follow.

### Discussion review, round 1 (burler, BLOCKING)

- The operator's 2026-10-21 decision (zero discovered passes, no floor, no skipped outcome) folded into D7: `min_tests`, `build_tags`, the drop rule and the skipped outcome are gone from D1, D3, D7, D11, D12 and the acceptance criteria; this settles F2's floor half, F6, F7's drop half, F10 and F11.
- F1 — fixed: the card `**Verify:**` runner keeps one slot for a line that is not one `lyx gate test --wait` call, D8 and D13.
- F2 — fixed: under `none`, computed, `--base` and card scopes are whole and explicit targets pass verbatim, D3.
- F3 — fixed: darn's tree-scoped landing sites, D8 table.
- F4 — fixed: darn runs no command for an empty rendered `{targets}`, D9.
- F5 — fixed: `--wait` required in `tree_scope_verify` and in a `**Verify:**` gate call, and the caller timeouts no longer apply to a gate call, D8, D9, D10.
- F6 — fixed: a `{targets}`-less tier runs whole except under an empty scope, D1.
- F7 — fixed: D3's Bound names the empty-set build it leaves to `final` and `landing`.
- F8 — fixed: required tier keys, whole-tier writes through a flow-mapping `--set`, since the orch cannot drive the editor, and a provisional `timeout_min`, D1 and D12.
- Self-check forks: NORCE tiers need no `all_targets`, the guard step's prebuild cost, the Goal's exceptions, darn's doc example and template comment, the auto-mode line and the trimpath constraint followed the fixes.
- F9 — fixed: the guard step is a run of `guard_tier`, D2, D5, D11.
- F12 — fixed: `-p={parallel}` in every loomyard tier's GOFLAGS, D12.
- F13 — fixed: D13 for non-tier slot holders; PATTERN texts name the exceptions.
- F14 — fixed: Merge-In and Publish name the round's base command on Site, D8.
- F15 — fixed: `-C` only with explicit targets, D2.
- F16 — fixed: the placeholder check runs only for a guard step not skipped, D5.
- F17 — fixed: D2's Bound reworded.
- F18 — fixed: D10's Bound lists what the row lets through.
- F19 — fixed: `tree_scope_verify` may hold no shell control operator, D9.
- F20 — fixed: pre-deploy processes and the drain, Open risks.
- F21 — fixed: a stale machine-layer `parallel` is dropped with a warning, D1.
- F22 — fixed: the watchdog watches Go test binaries only, D2.
- F23 — fixed: the Webster prompts stop rendering the plan's verify section, D9.

### Discussion review, round 2 (burler, BLOCKING)

- F1 — fixed: the fork runs a card's `**Verify:**` through `lyx gate card-verify`, the same slotted runner as Go's re-run, D8 and D13; under `lyx gate` because forks are refused every `lyx webster` command.
- F2 — fixed: confirm-the-fix renders each step as its own `lyx gate test --wait` call joined by `&&`, D8.
- F3 — fixed: the `.md`-only acceptance criterion names the scoper.
- F4 — fixed: `cmd/testtiming` holds one slot in every mode, D13 and `PATTERN-test-tiers`.
- F5 — fixed: brace-holding flow-mapping values are double-quoted, D1 and D12.
- F6 — fixed: in-flight `--tags` `**Verify:**` lines, Open risks.

## Question ledger

- Resolved (auto-pick): tier config home → `gate.yaml`.
- Resolved (auto-pick): site naming → resolved `--tier` lists, no `--gate` flag.
- Resolved (auto-pick): plan `## verify:` → retired as input, still parsed, informational finding.
- Resolved (auto-pick): slot ownership → per tier in the gate; `verifytree` takes none.
- Resolved (auto-pick): heavy tiers → drain lock plus ordered acquisition.
- Resolved (auto-pick): counters → `go-json` and `vstest`.
- Resolved (operator, 2026-10-21, replacing the parent review's round-1 ruling): zero-test rule → a run that discovers zero passes; no floor, no skipped outcome, the count is logged.
- Resolved (auto-pick): `final` and `landing` scope → `--all`, per the operator decision in note `loom-gates-xdr-stacking`.
- Resolved (auto-pick): gate-list cross-references → checked at use, not at load.
- Resolved (auto-pick): computed scope diffs to the working tree.
- Resolved (auto-pick): guard tests → `guard_tier` under the `go` scoper.
- Resolved (auto-pick): raw-test refusal → hub list `refuse_raw`.
- Resolved (auto-pick): run log → board's ephemeral gate directory, 14-day pruning.
- Resolved (auto-pick): hub values → orch applies after deploy.
- Resolved (auto-pick): change-description wall times → out of scope, reason in the record.
- Resolved (auto-pick): `--tags` → retired with a refusal naming `--tier`.
- Resolved (auto-pick): the fork's run of a card `**Verify:**` → `lyx gate card-verify`, the slotted card runner, over refusing build and test lines at the plan gate.
- Resolved (auto-pick): `cmd/testtiming` → one slot in every mode, over exempting the timing and `-redundancy` modes from `PATTERN-gate-slot`.
