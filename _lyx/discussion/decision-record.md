# Decision record — gate-test-scope

## Goal

Every gate runs the test tier its position needs and no more, and the hub as a whole never runs more gate builds and tests at once than its configured slot count allows.
Concretely: the card gate drops to tier 1 of the card's own packages, tier 2 moves to one pass per batch plus the full plan verify, tier 3 stays at Publish and reaches the round gate only to confirm a fix of a Publish failure, the `llm` tier stays by hand as a recorded decision, a hub-wide slot lock bounds every gate test or build run and caps `-p`, spawned agents cannot start a raw module-wide `go` run, no session spawned in a pair can weaken `publish_verify` through its own tree, and `lyx config` refuses its hub-wide writes; that refusal guards against a mistaken write, not a determined agent, and D10's Bound names the routes that remain.
Alongside: the comment lint stops passing every comma-plus-conjunction break, and a Burler round after a goto from a failed Publish reviews the failure and the merge that brought it.

## Scope

In:

- The card gate command (`internal/websterengine/cardgate.go`) and a new per-batch gate command, both rendered into the implementer prompt.
- A new slot engine package and a new `lyx gate` CLI module whose `test` verb is the one sanctioned route for an agent's slotted `go test`.
- Slot acquisition at every Go-side gate run: the `verifytree.Verify` sites (webster gate, Webster-Burler round gate, Publish's plan verify and `publish_verify`, Finalize, `lyx webster verify`), the in-process card `**Verify:**` rerun, and the plan gate's `caller-uncovered` `go list` with export data, at both its gate site and `lyx loom validate-plan`.
- A hub-wide `gate` config module (slot count, `-p` cap, CLI slot wait).
- `landing` becomes a hub-wide config module, and `lyx config` refuses a hub-wide write from a task pair or under a slug-bearing strand name (D10).
- A standing PreToolUse deny in `claudeengine` against raw module-wide `go test`/`go build`/`go vet`, raw `tmux`/`llm`-tagged `go test`, and any command naming the slot-inheritance variable or the strand-name variable.
- A plan-gate check refusing a card `**Verify:**` command that runs a module-wide `go` command.
- A persisted Publish failure record, read by the Webster-Burler round gate and rendered into the Webster-Review prompt.
- The comment lint's comma-plus-conjunction rule (`internal/commentlint`).
- Webster implementer and verify-fix stencils, the plan template's card `**Verify:**` guidance, the Webster-Review rubric.
- Docs in the same commit: `pattern/PATTERN-test-speed.md` (gate table, `llm` decision), `pattern/PATTERN-comment-line-breaks.md` (lint bound), `pattern/PATTERN-verified-tree.md` (slot and failure record), a new `PATTERN.md` entry for the slot, `CLAUDE.md`'s push rule, the touched packages' `doc.go`, `docs/overview.md`'s module table, `contracts/specs/refusal-spec.md` rows for new refusals.

Out:

- Measuring tiers, tmux fixture cost, building `lyx` once per test run, fixture auto-gc: `test-suite-weight`, which depends on this task.
- A new route that hands a blocked Publish's finding to a fixer (`lyx loom reject` at Publish and the like): `publish-fix-route`.
  This task only makes the round after an existing goto see the failure.
- Slotting an agent's raw package-scoped `go test` while it iterates (see D7's bound).
- Any gate that runs the `llm` tier.

## Decisions

### D1 — Card gate: tier 1 of the card's own packages

Decision: `cardGateCommand` emits `lyx gate test <dirs>` over the card's surviving target directories (same Delete-only and no-Go-source filters as today), one step per module in the existing order: the root module first, then each nested module as `lyx gate test -C <module> <module-relative dirs>`, then `lyx loom lint-comments`.
Each step is rendered on its own line, not chained with `&&`, and the fork runs each as its own Bash call (D8), so one call holds at most one slot wait.
No `go build ./...`, no module-wide `go test ./...`, no `-tags integration`.
A card with no surviving directory runs only the lint.
`cardgate_test.go` is updated to the new shape.
Rationale: the module-wide build and test per card was the crash's load; a card's own compile and unit tests are what its commit must hold, and every wider check runs later at the batch gate (D2) and the plan verify.
Bound: a card that breaks a package outside its targets passes its card gate; the batch gate catches it when that package is in the batch, and Webster's gate (full plan verify) catches it otherwise, before Webster-Review.

### D2 — Batch gate: tier 2 once per batch for the batch's packages

Decision: begin-batch and recover-batch render a batch gate command beside the card gates: `lyx gate test --tags integration` over the union of the batch's cards' surviving directories, one step per module as in D1.
The fork runs it once, after the last card's commit and that card's own `**Verify:**`, before writing its report.
A failure counts as a gate failure under the existing self-fix cap and makes the report `FAILED`.
D7's slot-busy exit is not a failure: it costs no self-fix attempt, does not make the report `FAILED`, and the fork re-runs the step; the same holds for a card gate step, a card's `**Verify:**` line and a verify-fix run (D8).
Go never runs it, as with the card gate; the plan-level verify gate catches a fork that skips it.
Rationale: `-tags integration` runs the untagged files too, so one pass gives tier 1 and tier 2 for the batch's packages; once per batch instead of once per card removes the repeated tier 2 load.

### D3 — Plan verify, Publish and Finalize unchanged in scope

Decision: the plan's `## verify:` keeps running in full (build, vet under each tag, tier 1, tier 2) at Webster's gate, Publish and Finalize, and the Webster-Burler round gate keeps its impacted-set command.
Only D4's slot and D5's failure-driven tier 3 pass change these sites.
Rationale: these are the sites that guard the tree that lands; narrowing them would move defects onto `main`.

### D4 — Hub-wide slot lock for gate runs

Decision: a new engine package (working name `internal/gateslot`) owns a hub-wide pool of slots.
Each slot is one OS file lock (`internal/lock`) under the hub's never-tracked `<BoardDir>/.lyx/` mirror; a holder writes a holder record (worktree, site or command, pid, start time) beside its lock after acquiring it, and a reader trusts a holder record only while its lock is held.
The engine is told the slot directory, the slot count and the `-p` cap, and derives no path (`PATTERN-told-geometry`).
Every Go-side gate run listed in Scope acquires a slot before spawning and releases it when the spawn ends.
For a shell command, `verifyrun`'s callers (`verifytree.Verify` and `cardverify`) acquire the slot and compose the environment: `-p=<cap>` appended to any existing `GOFLAGS`, plus the inheritance variable.
`verifyrun.Run` gains an environment parameter and stays free of the slot engine; `lyx gate test` passes `-p <cap>` itself.
The spawned shell carries an inheritance variable naming the held slot; a `lyx gate test` that sees it and finds that slot's lock held runs inside the parent's slot without acquiring, so a verify command that calls `lyx gate test` cannot deadlock.
Go-side sites wait for a slot with no deadline of their own, cancellable by the caller's context; the site's verify timeout starts once the slot is held.
A waiter polls the slots in an injectable interval and logs the start and end of its wait through `internal/logger`.
Rationale: an OS file lock releases on process death, so a crashed holder never leaks a slot; polling a small pool is simpler than a queue and needs no daemon.
Bound: the slot bounds concurrent gate runs, not their size; one slot's run is still capped only by `-p`.
A command that passes its own `-p` overrides the `GOFLAGS` cap.
The inheritance variable can be forged to skip acquisition while some slot is held; D7's deny refuses any agent command naming it, and an operator shell is outside the bound.
Polling gives no FIFO order, so a waiter can be overtaken; the wait is visible (D6), not ordered.

### D5 — `gate` hub-wide config module

Decision: a new config module `gate`, marked `HubWide` in `configreg` and read and written only at `<BoardDir>/_lyx/config/gate.yaml` (`PATTERN-hub-wide-config`), loaded strictly, with keys `slots` (default 2), `go_parallel` (the `-p` cap, default 4) and `cli_wait_sec` (how long `lyx gate test` waits for a slot before its slot-busy exit, default 300, so a stalled holder reaches the caller as a named error instead of an open-ended wait).
Neither the wait nor the run sits under the Bash tool's timeout, because every agent runs every `lyx gate test`, and every card `**Verify:**` line, as a background Bash call (D8).
A value below 1 fails the load naming the key.
The count is read at each acquire; lowering it leaves a running holder of a higher slot to finish.
Rationale: slot count and parallelism describe the machine the hub runs on, which no pair may change; D10's pair-side refusal covers `gate` as it covers `landing`.

### D6 — Wait shown in status

Decision: a Go-side site's wait is recorded in the `verifytree` running marker as a waiting state with its start time, and flips to running when the slot is held.
`lyx gate test` writes a wait record under its worktree's `.lyx/` while it waits and removes it on acquire or exit.
`lyx loom status` reports, for its worktree, each waiting gate (site or command) with its wait start, and the hub's current slot holders by worktree and site.
Rationale: a stall behind another run's Publish must be explainable from the stalled run's status without reading panes.
Bound: display only; no Go decision reads the wait record or the waiting state.

### D7 — `lyx gate test` and the agent deny

Decision: a new cobra CLI module `lyx gate` (`PATTERN-cli-cobra`) with one verb, `test`: `lyx gate test [-C <dir>] [--tags <tags>] <packages...> [-- <go test flags>]`.
It acquires a slot (D4) waiting at most `cli_wait_sec`, runs `go test -p <cap>` with the told arguments, streams the output, logs the spawn and its end, and exits with `go test`'s code.
It needs at least one package argument.
When every slot stays busy for the whole wait it exits with a distinct code and a JSON error naming the holders and the way forward: re-run the same command.
Hub membership is decided by the `-C` dir, else the cwd, never by the bare process cwd when `-C` is given, so `cd` elsewhere plus `-C <worktree>` is still slotted.
`lyx gate`'s own pre-run resolves it: `lyxcwd.ResolveWorktree` on that dir, which finds the containing worktree from any directory inside it, then `preflight.BoardLyxPresent` on the result.
It does not use `preflight.ResolveMode`, whose anchor gate refuses any directory but the worktree's anchor and so would refuse a nested module, and it does not go through `internal/cliwire`, because `lyx gate` has no `--target-dir`, standalone state, plan or stencils to wire.
The same `-C` dir is handed to `go test -C`, so one flag picks both the module and the hub; this is intended, because a nested module sits inside the worktree whose hub it shares.
When that dir sits in no git worktree, or in one with no hub board, it runs unslotted under the template's `-p` cap and logs that no hub bound applies.
`go test` runs in its own process group, so `lyx gate test` can kill the whole `go test` tree without signalling the shell that called it; it kills that group when it catches a terminating signal, including one sent to its own group, so a killed holder takes its run down with its slot.
On Linux the `go` child also carries a parent-death `SIGKILL` (`Pdeathsig`, set from a goroutine locked to its OS thread), so even a `SIGKILL` of `lyx gate test` ends `go` itself.
On Windows `go test` runs inside a job object created with kill-on-job-close, so any exit of `lyx gate test`, a hard kill included, ends the whole tree.
`claudeengine` adds one standing PreToolUse deny row, matched by a `grep -E` over the Bash payload like the python deny, refusing: a raw `go test`, `go build` or `go vet` with any package pattern holding `...` or equal to `all`, with or without `-C`; a raw `go test` whose tags name `tmux` or `llm`; and any command naming the slot-inheritance variable or `LYX_STRAND_NAME` (D10).
Its steer names `lyx gate test` as the route; `buildDenyNotice` announces it from the same table.
Every spawned session gets it, the orch included.
Rationale: the crash came from agents' module-wide runs; refusing those at the tool boundary and routing them through one slotted verb bounds them without slowing an agent's package-scoped iteration.
Bound: the deny sees the command's static text only; a module-wide run hidden behind a script, a `cd` plus a relative pattern the regex does not cover, or a shell variable passes, and so does a raw package-scoped `go test` (tiers 1 and 2) of any package list, which is unslotted and bounded only by the stencils' package list (D8).
`lyx gate test` accepts a module-wide pattern, which is slotted.
The unslotted path lets through only a run whose target is outside every hub, such as a copy of the module elsewhere on disk, capped by the template's `-p` alone.
An uncatchable kill (`SIGKILL`) of `lyx gate test` frees its slot; on Linux `go` dies with it, but test binaries and compilers `go` had already started run on unslotted until they end, and on another Unix the whole `go test` group runs on.

### D8 — Stencils name each fork's packages and the gate route

Decision: the implementer prompt renders the batch's package directories (D2's union) and states that the fork may run raw `go test` on those only, that module-wide and `tmux`-tier runs go through `lyx gate test`, that each gate step and each card's `**Verify:**` line runs as its own background Bash call (`run_in_background`) whose exit the fork waits for, so neither the slot wait nor the run is killed by the Bash tool's timeout, and that a slot-busy exit is handled as D2 states.
The verify-fix fork's stencil states the same route for the packages its findings name, the same background call for each `lyx gate test`, and the same slot-busy handling.
The plan template states that a card's `**Verify:**` is package-scoped and uses `lyx gate test` for a module-wide or `tmux` run.
`CLAUDE.md`'s push rule runs `lyx gate test ./...` and `lyx gate test --tags integration ./...`, each as its own background Bash call; no orch stencil carries a test step, and the orch reads the push rule from `CLAUDE.md`.
Rationale: the deny (D7) refuses; the stencil tells the agent beforehand what it may run, so a fork never learns the rule by refusal.

### D9 — Plan-gate check on card `**Verify:**`

Decision: a new blocking plan-gate check refuses a card whose `**Verify:**` command contains a raw command D7's deny would refuse, naming the card and the `lyx gate test` form.
It lives in the plan gate's shared validation, so the gate and `lyx loom validate-plan` call the same function (`PATTERN-gate-self-check-parity`), and `lyx loom validate-plan` reaches it in a test.
Rationale: a fork cannot run such a Verify under the deny, so a plan holding one fails its batch by construction; the plan gate is where that is cheapest to catch.
Bound: it matches the same static patterns as D7 and inherits that bound.

### D10 — `landing` becomes hub-wide

Decision: the `landing` config module is marked `HubWide`, so `publish_verify` and every other landing key are read only at `<BoardDir>/_lyx/config/landing.yaml`; the existing hub-wide reconcile seeds the hub file from the prime's copy when absent (`SeedPrime`), keeping this repo's `go test -tags tmux ./...`, and a pair's leftover copy is reported retired or divergent by the existing reconcile.
Publish and Finalize load landing config from the board dir.
`lyx config` refuses a write (`--set` or edit) to any `HubWide` module when the worktree it resolves is a task pair, or when `LYX_STRAND_NAME` (`agentname.StrandNameEnv`, which reed exports into every spawned session) parses through `internal/agentname` as a slug-bearing strand name, wherever the worktree resolves.
The refusal names the way forward: the operator or the hub orch runs it from the prime.
Rationale: a pair's tracked `landing.yaml` was the hole that let a task empty its own `publish_verify`, and its other keys (`require_pr_to_base`, `squash`) are the same kind of landing policy a task must not set for itself; the hub-wide machinery already exists.
Without the refusal, `lyx config <module> --set` on a `HubWide` module from a pair writes and commits the hub file, so moving `landing` hub-wide would turn a pair-local hole into a hub-wide one.
Bound: this removes per-pair landing overrides for every key; a landing change is now an operator or hub-orch `lyx config landing` from the prime, committed in `_board`, effective for every pair at once.
`lyx config` has no `-C` flag, so the route into the prime is a `cd`, and the strand-name key refuses it while the variable is inherited; D7's deny refuses any agent command naming `LYX_STRAND_NAME`.
What still passes, each a deliberate act rather than a mistaken write: a direct file write to `<BoardDir>/_lyx/config/landing.yaml` outside `lyx config`; a command that clears the environment without naming the variable (`env -i`, `exec -c`, an `unset` over a prefix expansion) before `lyx config` from the prime; and an unset hidden behind a script (D7's Bound).
The hub orch, whose strand name carries no slug, keeps its writes.

### D11 — Tier 3 at the round gate only to confirm a Publish fix

Decision: when Publish's plan verify or `publish_verify` fails, Publish persists a failure record under the worktree's `.lyx/` (which verify failed, as a kind — plan verify or `publish_verify` — never a command string, the failing test identities parsed by the parser webster's verify gate uses, the log path, HEAD, and the merge-in commit Publish made, if any), and a later Publish pass deletes it.
While a record is present, the Webster-Burler round gate adds to its impacted-set command: each failing identity run by name in its package under the failing verify's tier, and, when the failing verify was `publish_verify`, `go test -tags tmux` over the impacted set.
The round gate builds that command from the record's kind and the identities that pass D12's identity check, quoting each as an argument, and never runs text from the record as a shell command.
With no record, the round gate runs no tier 3, as today.
Rationale: an impacted-set tier 3 pass every round costs tmux servers and minutes for a tier that rarely breaks; after a Publish failure it is the only evidence a fix round has that the failing tier now passes, which `shuttle-send`'s three repeated Publish blocks lacked.
Bound: with no record, a tier 3 defect still surfaces first at Publish; identities the parser cannot extract (a build failure, a panic before any test line) fall back to the impacted-set pass alone.
The record sits in the pair's own `.lyx/`, so deleting or editing it can skip, narrow or redirect the round pass and D12's review widening.
After D12's shape checks, an edited record carries no shell text or free prose into the reviewer's prompt or shell: what still passes is a well-formed test name that never failed, which forces a BLOCKING review finding for it, and another real merge commit of the worktree, which points the reviewer at that commit's diff.
Publish reruns both verifies in full regardless, and stays the guard.

### D12 — Burler range after a goto from a failed Publish

Decision: while D11's record is present, the Webster-Review prompt renders it through a stencil marker: the failing verify, identities and log path, and the merge-in commit with the command that reads what it brought in (`git diff <merge>^1 <merge>`).
Each field passes a shape check before it renders: the merge-in commit is a full hex object name that resolves, through `internal/gitrepo`'s go-git reads, to a merge commit in the worktree; each identity is a Go test name, a top-level `Test` identifier with optional `/`-separated subtest segments holding no whitespace, quote or shell metacharacter; the log path lies under the directory Publish writes its verify logs to.
A field that fails its check does not render, and the failure is logged; the verify kind renders from the record's closed set, never as text.
The Webster-Review rubric tells the reviewer to raise each failing identity as a BLOCKING finding, reading the merge's content and the log for its cause, in addition to the branch's own range.
Rationale: after Publish merges the parent in, `git merge-base` moves to the parent's tip, so the branch-only range excludes the merged content that carried the defect.
Bound: the rendered record is the only widening; with no record the range rules stay as they are.

### D13 — Comment lint: comma-plus-conjunction passes only on its own clause

Decision: a checked break ending in a comma whose next line starts with a coordinating conjunction passes only when the words after the conjunction, up to the next `,`, `;`, `:` or sentence end, open with a subject (a subject or demonstrative pronoun, a determiner followed by a word, or a backticked identifier) and hold a finite-verb candidate after it (a closed set of auxiliaries and modals, or a word ending in `s` or `ed` after the subject's head word).
Any other comma-plus-conjunction break is a finding.
The lint stays diff-scoped: only new breaks are checked.
Rationale: an uncertain case errs toward flagging, because joining the two lines always passes the lint, so a false finding costs one join and a false pass costs the rule.
Bound: a plural noun read as a verb (`, and the gate tests`) passes; a clause whose subject the opener set misses (`, and nothing runs` with a pronoun outside the set) is flagged and joined.

### D14 — `llm` tier stays by hand

Decision: `pattern/PATTERN-test-speed.md` records as a decision that no gate runs the `llm` tier: every gate compiles it through `go vet -tags llm`, and running it is the operator's call.
Rationale: an `llm` run is billed, nondeterministic and needs a live session, so a gate on it would cost money per round and fail on the model rather than the code.

### Added after Discussion (parent, 2026-10-10): lyx gate child lifetime stays in internal/gatecli

Decision: The process-group, Pdeathsig and job-object code for lyx gate test lives in new internal/gatecli files only; no card touches internal/proc, ConfigureGroupKill, ErrNoProcessGroup or internal/verifyrun's procgroup files.

Rationale: The parallel run git-robustness creates those internal/proc identifiers and deletes verifyrun's procgroup files; keeping D7's child code apart lets the two branches merge without conflict, and moving it into internal/proc is a later board note.

### Added after Discussion (parent, 2026-10-10): CLAUDE.md push rule keeps the text-only line

Decision: The rewritten CLAUDE.md push bullet keeps main's second line: a commit that changes only text no build or test reads, such as docs/ and CLAUDE.md, is pushed without the test run.

Rationale: main gained that line in 0cdd14ac9 after this branch forked; dropping it in the rewrite would revert it at landing.

### Added after Discussion (parent, 2026-10-10): verifyrun keeps main's proc group kill

Decision: main's internal/verifyrun/run.go now calls proc.ConfigureGroupKill and its package doc names internal/proc (git-robustness, 6741f7157); card 3's re-signing of Run and run keeps that call and that import line when the branch meets main, and adds no process-group code to verifyrun.

Rationale: git-robustness landed while this plan was written; the two branches edit run.go for different reasons and must both survive the merge.

## Constraints

- `PATTERN-told-geometry`: the slot engine is told its directory, count and cap; only a CLI layer resolves the board dir, `lyx gate`'s through its own pre-run (D7).
- `PATTERN-lyxdirs-single-declarer` and `PATTERN-durable-vs-ephemeral-state`: slot locks, holder records, wait records and the Publish failure record are never tracked and live under `.lyx` mirrors whose segments come from `internal/lyxdirs`.
- `PATTERN-hub-wide-config`: `gate` and `landing` are `HubWide`, read and written only at the board dir.
- `PATTERN-config-strictness`: `gate` loads strictly, as `landing` does.
- `PATTERN-verified-tree`: every plan-verify site still calls `verifytree.Verify`; the slot is acquired inside that path, so no site gains a second runner, and `cardverify` stays a non-plan-verify site that also takes a slot.
- `PATTERN-spawn-observability`: `lyx gate test`'s `go` spawn and every slot wait are logged.
- `PATTERN-cli-cobra`, `PATTERN-sandbox-coverage`: `lyx gate` is a cobra subtree, exercised by the sandbox suite or excluded with a reason; it is not a standalone-capable CLI, so `PATTERN-cliwire-sole-wiring` does not apply.
- `PATTERN-shuttle-provider-seam`: the deny row lives in `claudeengine` only.
- `PATTERN-no-denied-recovery`: no refusal, stencil or spec names a raw module-wide `go` run as a way forward once D7 denies it; each names `lyx gate test`.
- `PATTERN-refusal-way-forward`: D9's plan-gate refusal, D10's pair-side `lyx config` refusal and any new loom or webster refusal get a spec row in the same commit and a test reaching them.
- `PATTERN-gate-self-check-parity`: D9 sits in the shared plan validation; the comment lint change stays inside `commentlint.Lint`.
- `PATTERN-test-speed`: slot tests sit in tier 1 where they spawn nothing (file locks are not spawns), with an injectable poll interval; `lyx gate test` spawn tests sit in tier 2.
- `PATTERN-comment-line-breaks`: its lint-bound section is rewritten to D13.
- `verifyrun`'s package doc pins its imports to the standard library and `internal/logger`; D4 keeps that set.
- Windows: the slot lock uses `internal/lock` (gofrs/flock), which works there; `GOFLAGS` composition and the deny regex hold under git-bash; `lyx gate test`'s child lifetime uses a kill-on-close job object there (D7).

## Auto-mode assumptions

- Defaults: `slots` 2, `go_parallel` 4, `cli_wait_sec` 300; the operator tunes them in `gate.yaml`.
- The slot directory is a `gate` segment under `<BoardDir>/.lyx/`.
- The CLI verb is `lyx gate test`; the module name `gate` does not collide with an existing CLI module.
- Raw package-scoped `go test` by agents stays unslotted.
- `landing` moving hub-wide is in scope as the means for "a pair cannot empty `publish_verify`", and closes the same hole for its other keys.
- The tier 3 round pass is conditional on a Publish failure record, not run every round.
- Webster-Review is the round that reads the failure record; Discussion- and Plan-Review do not.

## Open risks

- The deny regex can refuse a legitimate command whose text contains a matching pattern (a `grep` for `go test ./...`, a commit message) and miss a disguised one; its tests must pin both a refusal set and a pass set.
- With 2 slots and four runs, a Publish holding a slot for many minutes queues forks' gate runs; the CLI wait bound turns that into re-runs, which costs fork turns.
- The comment lint heuristic will flag some valid clause breaks; the cost is a join per flag, but a fork's self-fix cap counts lint failures.
- Making `landing` hub-wide changes how existing pairs load it; a pair started before the change keeps a now-retired copy until reconcile, and Publish must not read it.
- A verify command run inside a slot that itself starts nested `lyx` gates under a different worktree would not inherit; none is known.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - card-verify-and-verify-fix-miss-background-rule (class design, severity MEDIUM, open in rounds 3)
  - cli-wait-sec-vs-bash-timeout-chain (class design, severity MEDIUM, open in rounds 1, 2, 3)
  - d10-pair-can-write-hubwide-landing (class design, severity BLOCKING, open in rounds 1, 2, 3)
  - d7-cliwire-preflight-import-barred (class design, severity BLOCKING, open in rounds 3)
  - d7-outside-hub-unslotted-unbounded (class design, severity BLOCKING, open in rounds 1, 2, 3)
  - failure-record-tamper-bound-unstated (class design, severity BLOCKING, open in rounds 1, 2, 3)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- The card gate for a card with targets in the root module and a nested module is `lyx gate test` over the root directories, then `lyx gate test -C <module>` over module-relative directories, then `lyx loom lint-comments`, each step on its own line with no `&&` chain, with no `go build ./...` and no `-tags integration`; a card with only stencil or doc targets runs only the lint.
- The implementer prompt carries a batch gate running `lyx gate test --tags integration` over the union of the batch's directories, per module, and the batch's package list.
- The implementer prompt states that each gate step and each card `**Verify:**` line is its own background Bash call and that a slot-busy exit from either is re-run without a self-fix attempt or a `FAILED` report; the verify-fix stencil names `lyx gate test` for its findings' packages, run as a background Bash call with the same slot-busy handling; the plan template's `**Verify:**` guidance names `lyx gate test` for module-wide and `tmux` runs; `CLAUDE.md`'s push rule runs both `lyx gate test` commands, each as a background Bash call.
- With `slots: 1`, a second slot acquirer waits while the first holds, logs the wait start and end, and proceeds when the first releases; a holder that exits without releasing frees its slot.
- A Go-side verify while all slots are held shows a waiting state in `lyx loom status`, with its start time and the holders' worktrees and sites, and its verify timeout starts only once the slot is held.
- `lyx gate test` with all slots busy past `cli_wait_sec` exits with its distinct code and a JSON error naming the holders and re-running as the way forward; with a free slot it runs `go test -p <cap>` and exits with its code; terminated by a catchable signal mid-run, it kills its `go test` process group and its slot frees; on Linux, a `SIGKILL` of `lyx gate test` mid-run ends its `go` child too.
- `lyx gate test` run from inside a slot-holding verify command runs without acquiring a second slot.
- `lyx gate test -C <worktree>` run from a cwd outside the hub takes a slot and runs `go test` in that dir; `lyx gate test -C <nested module dir>` inside a hub worktree takes a slot and runs `go test` in the nested module; with a target outside every hub it runs unslotted under the template's `-p` cap and logs that no hub bound applies.
- A shell command run by a Go-side gate sees `-p=<cap>` in `GOFLAGS`, appended to any existing value.
- The deny refuses `go test ./...`, `go build ./...`, `go vet ./...`, `go test ./internal/...`, `go -C x test ./...`, `go test all`, `go test -tags tmux ./internal/x` a command naming the inheritance variable and one naming `LYX_STRAND_NAME`, and passes `go test ./internal/x`, `go test -tags integration ./internal/x -run Foo` and `lyx gate test ./...`; the deny notice announces it.
- `lyx loom validate-plan` refuses a card whose `**Verify:**` is `go test ./...`, naming the card and the `lyx gate test` form, and passes one whose Verify is `go test ./internal/x`.
- Publish reads `publish_verify` from the board dir's `landing.yaml`; a pair whose own `landing.yaml` sets `publish_verify: ""` still runs the hub's command.
- Hub reconcile with no hub `landing.yaml` seeds it from the prime's copy, keeping its `publish_verify`.
- `lyx config landing --set publish_verify=` run in a task pair is refused naming the prime as the way forward, and the hub file is unchanged; the same write run from the prime's directory under a slug-bearing `LYX_STRAND_NAME` is refused too; the same write from the prime with no strand name, or the hub orch's, succeeds.
- A `gate.yaml` with `slots: 0` fails the load naming `slots`.
- A failing `publish_verify` leaves a failure record; the next Webster-Burler round gate runs the failing identities by name and the impacted-set `tmux` pass; a passing Publish removes the record, and a round with no record runs no `tmux` test.
- A failure record with no extracted identity (a build failure) makes the round gate run the impacted-set pass alone.
- With a failure record present, the Webster-Review prompt carries the failing identities, log path and the merge-in commit's diff command; a record whose merge field is not a merge commit's full hex name (`HEAD --output=x`, a non-merge commit), whose identity holds a shell metacharacter, or whose log path lies outside Publish's log directory does not render that field, and the round gate runs no such identity.
- The comment lint flags a new break before `, and returns the value` (compound predicate) and `, and \`Foo\`` (list item), and passes one before `, and it returns the value` and `, but the gate refuses it`; untouched legacy breaks stay unchecked.
- `pattern/PATTERN-test-speed.md`'s gate table states D1, D2, D3, D11 and D14; `CLAUDE.md`'s push rule names `lyx gate test`; `PATTERN.md` carries the slot entry; `go test ./...` and `go test -tags integration ./...` stay green.

## Notes for the plan writer

- The python deny in `internal/shuttleengine/claudeengine/settings.go` (`standingDenies`, `buildDenyNotice`) is the model for D7's row.
- The verify-gate failure parser webster already uses for its flaky rerun is the source for D11's identities.
- `configsync.ReconcileHubWideAt` and its `SeedPrime` path already cover D10's migration; check `configcli`'s hub-wide set and edit paths pick `landing` up from the flag alone.
- `cmd/lyx/testdata/test-budget.yaml` likely needs rows for the new packages.
