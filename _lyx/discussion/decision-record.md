# Decision record — verify-publish-hardening

## Goal

A failing test stops on the card that caused it, the plan-level verify runs once per attempt as a mechanical gate on Merriam that sends a failure back to Merriam to fix, Publish never verifies or pushes a dirty tree, no verify is skipped after a crash, and no agent prompt can hit the launch-line limit.

## Scope

In:

- **Per-card gate**: Go derives each card's gate command; it runs the whole untagged suite (repo scans included) and the integration-tagged tests of the card's own packages.
- **Plan-Write stencil**: a card that rewords a message greps the repo for the old text and takes the stale assertions into its own scope.
- **Webster verify gate**: a shuttle gate on Merriam's strand replaces Merriam's integration fork, Go's post-exit integration stage, the in-process bisect, the triage baseline and the `webster-recovery` integration-fix strand.
  Shuttle's gated wait stops counting a turn end with background work outstanding as an arrival.
- **Merriam's stencil**: no integration fork, no polling loop (recover-batch included), no "started non-interactively" framing, and a gate-failure section that forks a fixer.
- **Strand role**: Merriam's strand is named `<shortname>:<slug>:webster`.
- **Verified-tree record**: one record per worktree of the tree that last passed verify, written by every verify site, replacing landing's pending-verify marker.
- **Webster-Burler `verify` gate**: a Burler round that changed code ends with a passing verify.
- **Publish and Finalize**: clean-tree checks around merge-in, verify and push; verify runs only when the tree differs from the verified tree.
- **mergeresolve**: the conflict session's tracked edits land in the merge commit, untracked files halt, and stale conflict reports are cleared per `Resolve`.
- **Prompt delivery**: claudeengine launches every fresh session with a short pointer to the `prompt.md` it already writes.
- **Gate visibility**: the gate reports a running verify through loom status's `Waiting` hook.
- **Self-check**: a `lyx webster verify` verb runs the same function as the gates.
- **Docs**: the webster and landing package docs, `contracts/specs/webster-spec.md`, `contracts/specs/loom-plan-spec.md`'s verify model, `contracts/specs/refusal-spec.md`, `docs/overview.md`, CLAUDE.md's Merriam entry and a new `CONSTRAINTS.md` invariant.

Out:

- `begin-batch`'s dormant model assertion stays.
- `webster.yaml`'s `master` model key keeps its name.
- PR-Gate, Describe, the Bouncer judge (owned by `review-finding-class`) and the PATTERN move (owned by `pattern-init`) are untouched.
- Pre-existing failures on the parent branch get no special classification.

## Decisions

### D1 — Per-card gate is derived by Go, not written by Plan-Write

Decision: `begin-batch` renders each card's gate command into the fork prompt, replacing the stencil's "`go build ./...` and this card's package's unit tests".
The command is `go build ./...`, then `go test ./...` (untagged, whole repo), then `go test -tags integration <pkgs>` over the Go package directories holding the card's targets (path targets and the files holding symbol targets).
Directories that hold no Go package after the card (a Delete-only directory) are left out.
A card's own `**Verify:**` still runs after the gate when declared.
The integration-tagged run moves part of the spec's Tier 2 to per card, so `loom-plan-spec.md`'s Tier 2 text changes with Tier 1's; the plan-level `## verify:` stays the once-per-plan run.
Bound: the fork runs the gate and Go never does, as with today's stencil gate.
A fork that skips it is caught at the webster gate (D3), which localizes through D4's hint.
Rationale: the plan spec's Tier 1 already calls for a per-card run with no author enumeration.
The untagged whole-repo suite takes ~13 s cold and ~2 s cached here, so it covers callers and the `cmd/lyx` repo scans at near-zero cost, without a scan-package config key or a Plan-Write rule that can be forgotten.
The integration tests of the card's own packages are the localization refusal-audit lacked.

### D2 — Plan-Write greps for reworded messages

Decision: `loom-template-plan.md` gains a rule: a card that changes a user-visible message or error text greps the repo for the old text, and every test asserting it joins that card's targets.
Rationale: stale exact-string assertions then fail on the card that reworded them, not at the plan gate.

### D3 — The webster verify gate is a must-pass shuttle gate entry on Merriam's strand

Decision: `Run` passes `StartMaster` a one-entry `GateSpec` named `verify`, with `Attempts` from a new `webster.yaml` key `verify_gate_attempts` (template default 3).
On a gated run, shuttle treats a turn end with background work outstanding (`EventWaiting`) as no arrival, even when every output file exists.
Without that, the turn end that spawns D5's fixer re-arrives at the gate, because `outcome.yaml` and `summary.md` are still on disk from the first arrival, and the gate verifies the fork's half-edited tree.
Bound: the deferral holds only while the session is live; the deadline and liveness exits still classify Done from the output files, so `master_timeout_min` bounds a fork that never finishes.
The closure, evaluated at each gated Done arrival, does the following in order:

1. When `outcome.yaml` names an outcome other than `done`, it passes without verifying, so a stuck or paused outcome ends the run as today, without reaching Publish.
   An absent `outcome.yaml` cannot reach the gate, since a Done arrival requires both output files; if it did, the run errors at the post-gate outcome parse.
2. It runs the clean-tree check (D7); a dirty tree fails with the dirty paths as findings.
3. It checks the fixer commits (D5); a violation is a `Terminal` failure.
4. With no `## verify:` section it passes with a warning.
5. Otherwise it runs the shared verify function (D7).
   On a pass it records the verified tree.
   On a failure it reruns once to detect a flake; a rerun pass passes the gate with the existing flaky warning, summary section and friction note.
6. A failure that survives the rerun writes findings (D4).

An exhausted gate maps to outcome `stuck` with a reason naming the failing identities and the attempts spent.
Rationale: the shuttle gate re-prompts the live session, so a regression goes back to Merriam with its full plan context.
A must-pass entry cannot let a red tree through.

### D4 — Gate findings and the card hint replace bisect

Decision: the findings file names the attempt and cap, the failing test identities with their output tails (the existing failure parser), the full log path, and a hint.
The hint lists the cards whose recorded `CardSHAs` commits touched a file in a failing package's directory, by `NN-slug`.
Go writes the same content, plus the fix commits, to a per-run gate report `verify-gate.yaml` in the webster reports dir, replacing `integration.yaml`.
In-process bisect, the triage baseline run and every detached checkout of the live worktree go.
Rationale: bisect assumes intermediate card SHAs pass the integration suite and that the label list is in ancestry order.
Neither holds: a card's test can precede the card that makes it pass, and `begin-batch` does not enforce execution order, so the first failing SHA in list order is not the card that broke the test.
A touch-based hint never claims more than it knows, and Merriam judges it with the plan in hand.

### D5 — Merriam fixes through one fork per gate failure

Decision: on a gate re-prompt, Merriam reads the findings file and spawns one fixer fork in the background, then ends its turn.
The fork's prompt is a Go-rendered file from a new stencil `webster-body-verify-fix.md`, written at run entry in place of the integration prompt.
The fixer fixes the cause in source, never deletes, skips or weakens a test, never touches the plan directory or `_lyx`, and commits each fix as `fix: <summary>`.
It runs the failing packages' tests (with `-tags integration`) before reporting, and does not run the plan-level verify.
On the fork's notification Merriam rewrites `outcome.yaml` and `summary.md`, the summary gaining a "Verify gate fixes" section, and ends its turn, which re-arrives at the gate.
Go records the HEAD at the gate's first failure as the pre-fix head.
At each later arrival every commit from it to HEAD must be a non-merge commit or a clean parent merge on the first-parent chain, as record-batch already accepts, else the gate fails `Terminal`.
The plan directory and `_lyx` sit outside every warp commit, so the run-exit fork audit (`CheckFork`) is what catches a fixer write there: it audits the fixer fork as it audits a batch fork.
Bound: fixer commits skip record-batch's done-checks, drift detection and glyph scope guard.
They are bounded by the commit check, the run-exit fork audit, the gate's own verify and Webster-Review, which reviews the committed range.
Rationale: the board's fix direction; a fork keeps Merriam's context clean and Merriam never edits files.

### D6 — Merriam's stencil waits only on notifications

Decision: the integration-fork section, the "started non-interactively" framing and the `recover-batch` re-call loop go.
Merriam runs `lyx webster recover-batch <NN>` as a backgrounded Bash command and acts on its completion notification.
`recover-batch` blocks until the recovery strand reaches a terminal digest, and `poll_wait_s` retires.
The framing becomes: no operator answers, and ending a turn with no running background work and no outcome file ends the run.
Bound: an unbounded `recover-batch` is held by `recovery_timeout_min`, after which the strand classifies dead and the call returns.
Rationale: shuttle reads a turn end with backgrounded Bash outstanding as `EventWaiting` (`claudeengine/background.go`), so a backgrounded call keeps the run alive with Merriam idle; the Done-when forbids any polling loop.

### D7 — One shared verify function and a verified-tree record

Decision: one package function serves every verify site: the webster gate, the Webster-Burler gate, Publish, Finalize and `lyx webster verify`.
It takes told paths (worktree, record, log, marker) and the command, and does the following:

1. Clean-tree check: `git status --porcelain` with ignored files excluded must be empty, else it returns the dirty paths without verifying.
2. When the record holds HEAD's tree SHA and the same command, it returns a pass without running (skipped).
3. Otherwise it writes the running marker, runs the command through `verifyrun`, removes the marker, and on exit 0 writes the record `{tree, command, verified_at}`.

The record lives under `.lyx` (one per worktree, the path told to every consumer), and is written only after a pass.
Landing's pending-verify marker and its `treeChanged` argument retire.
Bound: the skip lets through only a tree byte-identical to one that passed the same command.
That holds because only this function writes the record, so `lyx webster verify` records only after a pass of its own.
No agent prompt names the record path; an agent writing it is outside the bound, which is a steering guard like the other agent write bans, not a security boundary.
A crash anywhere before the record write leaves a mismatch, so verify runs again; a lost `.lyx` only costs a re-run.
Rationale: the board's robust fix for `verify-gate-crash-after-merge`; keying on the tree, not the commit, also makes Publish's no-op merge skip and makes Burler's "changed code" check mechanical.

### D8 — Webster-Burler carries a `verify` gate

Decision: `loom-recipe.yaml`'s Webster-Burler row gains `gates: [{name: verify, attempts: 3}]`, resolved in `shedrecipe/entries_gate.go` to a closure over D7.
A round that changed no code sees the verified tree and passes without running.
A failure re-prompts the Burler round session with the findings.
An exhausted gate is Stuck to Webster-Bouncer, as every Burler gate failure already is.
Bound: a failing verify writes no record, so a red tree Webster-Bouncer approves reaches Publish, whose D7 verify runs because the tree differs from the record, and is Stuck there before anything is pushed.
Rationale: the perch that last edits code gets a mechanical test gate (operator, 2026-10-03), and the record makes the no-change case free.

### D9 — Publish and Finalize never verify or push a dirty tree

Decision: each producer runs the clean-tree check before merge-in, after merge-in, and after verify.
A dirty tree is Stuck naming the paths, before anything is pushed or merged into the parent.
Verify runs through D7 after merge-in, so it runs exactly when the tree differs from the last verified tree.
That covers resolved conflicts, new parent commits and a crash after a merge, and skips a no-op merge onto a verified tree.
Rationale: refusal-audit's `historyfold_test.go` and publish-review-fixback's uncommitted resolution edits; a verify that dirties the tree is a non-hermetic test, which must halt rather than ship.

### D10 — mergeresolve commits the conflict session's tracked edits

Decision: after a session whose markers are gone, `Resolve` stages every tracked modification and deletion (`git add -u`), not only the conflicted paths, and concludes the merge.
An untracked, non-ignored file the session created makes it abort the merge and report stuck naming the file.
The conflict stencil allows edits beyond the listed paths only to keep the merged tree building and passing, and its report names every such file.
Bound: the session can now change any tracked file inside the merge commit.
Verify (D9) and PR-Gate review the merge result, and new files stay refused.
Rationale: a semantic-conflict fix belongs in the merge commit; leaving it uncommitted is what put the dirty tree under verify.

### D11 — Conflict reports are per `Resolve` call

Decision: `Resolve` removes every `conflict-resolution-r*.md` in its scratch dir at entry, after its in-progress-merge abort and before attempt 1.
Rationale: the spec validator refuses an existing output file, so a report left by an earlier Publish refused every later conflicting merge-in.
The report is ephemeral diagnostics of one call.

### D12 — Every fresh launch carries a pointer, not the prompt

Decision: `claudeengine.Prepare` keeps writing `prompt.md` and launches every fresh session with one fixed pointer line: read `<abs prompt.md>` in full first, it is your complete, authoritative instructions.
The 30000-byte check applies to the launch argument, never to `spec.Prompt`.
Webster's `MasterPromptPointer` and its `master.md` copy retire, and Merriam's rendered prompt goes in `spec.Prompt` like every role's.
Rationale: one provider-level mechanism covers every role, including ones added later, and stays inside the Shuttle Provider-Seam Invariant.

### D13 — Merriam's strand role is `webster`

Decision: `Run` spawns Merriam with a strand-role constant `webster` declared in `websterengine`, separate from `RoleMaster`, which stays the `webster.yaml` model key `master`.
Rationale: the strand is named for what it is, and existing `webster.yaml` files load unchanged.

### D14 — Gate visibility through the `Waiting` hook

Decision: D7's running marker (`.lyx`, beside the record) holds the site label (`webster gate`, `Webster-Burler gate`, `Publish`, `Finalize`), the attempt where one applies, the start time and the pid.
Loom's `Hooks.Waiting` reports it as `verify <site> (attempt N, since HH:MM)`, ahead of the parent-review note, in both the one-shot status and `--watch`.
A marker whose pid is dead is ignored.
Rationale: `activity.wait` is persisted only for blocked, failed or awaiting states; the `Waiting` hook is the existing live channel for a running producer.

### D15 — `lyx webster verify` is the gates' self-check verb

Decision: a new verb runs D7 against the worktree's plan `## verify:` (hub or standalone geometry), records on a pass, and exits 0 on a pass or skip and 1 otherwise, with findings under the failure envelope's `findings` key.
The parity test pins that the webster gate, the Webster-Burler `verify` gate and the verb call the same function.
Rationale: Gate Self-Check Parity Invariant.

### D16 — New invariant: Verified-Tree Invariant

Decision: `CONSTRAINTS.md` gains: every plan-verify site goes through the one shared function; none verifies a dirty tree; a skip requires the record to name HEAD's tree and the same command; the record is written only after a pass.
The parity enforcement test doubles as its tripwire.
Rationale: four sites share the rule, and the next site must not reintroduce a marker or a working-tree verify.

## Constraints

- Gate Self-Check Parity, Completion Signal, Shuttle Provider-Seam, Agent Name, Durable-vs-Ephemeral State, Told-Geometry, Stencil Ownership, Test Tier Purity and Fabric Vocabulary invariants apply; landingshed and mergeresolve must not name either fabric side.
- Webster's degrading config: a hub `webster.yaml` still carrying `poll_wait_s` must keep loading, and a missing `verify_gate_attempts` takes its default.
- No migration: a webster run in flight with an `IntegrationFix` state record or an `integration.yaml` report restarts with `--fresh`; `state.json` decoding must not fail on the retired field.
- Gate attempt counts are in memory per shuttle run; after an attach they restart from zero, bounded by `master_timeout_min`.
- Card gates assume a Go repository, as the implementer stencil already does.

## Auto-mode assumptions

- The refusal-audit bisect misattribution is explained by the structural causes in the support log, without the incident's own logs.
- 3 gate attempts for both the webster and the Webster-Burler `verify` gate, matching every other recipe gate.
- Keep `master` as the model key; keep `begin-batch`'s model assertion.
- Drop the pre-existing classification; a parent-side failure halts at the gate as it would at Publish.
- Stage tracked edits from the conflict session rather than refusing them.
- Whole-repo untagged `go test ./...` per card instead of a configured scan-package list.

## Open risks

- The per-card integration run over a heavily tested package (`internal/fabricengine`) costs minutes per card that touches it.
- Webster-Bouncer's judge, being reworked in `review-finding-class`, could approve a red tree past an exhausted Webster-Burler gate, which costs a Describe and a Publish run before Publish halts it (D8).
- A pre-existing parent failure now consumes Merriam's gate attempts before halting.
- The pointer launch changes every role's first user message; a test or tool reading the first message for content would break.

## Acceptance criteria

- `go build ./...` and `go test ./...` pass, and the tagged suites of every touched package pass.
- A webster run's gate runs the plan's `## verify:` once per gate evaluation, and loom status shows `verify webster gate (attempt N, …)` while it runs.
- Test: a fake Merriam whose first done arrival fails verify and whose second passes ends `done` with one re-prompt, and the findings name the failing identity and the card whose commit touched its package.
- Test: an exhausted gate ends `stuck` with a reason naming the failing identities.
- Test: a dirty tree at a done arrival fails the gate with the dirty paths and does not run verify.
- Test: a turn end that spawns the fixer fork in the background does not evaluate the gate; the turn end after its notification does.
- Test: a merge commit above the pre-fix head, other than a clean parent merge, fails the gate `Terminal`, and a clean parent merge there does not.
- Test: a fixer fork that writes the plan directory is flagged by the run-exit fork audit.
- Test: a flaky failure that passes on rerun passes the gate with the flaky warning.
- Test: a non-`done` outcome passes the gate without verifying.
- Merriam's stencil contains no integration fork, no `await-batch` or `recover-batch` re-call loop, no `sleep`, and no "non-interactively".
  A template test pins the absence and the gate-failure fixer section.
- A template test pins D2's grep rule in `loom-template-plan.md`.
- Test: `recover-batch` returns only at a terminal digest or once `recovery_timeout_min` classifies the strand dead, and a `webster.yaml` still carrying `poll_wait_s` loads.
- Test: a failed gate writes `verify-gate.yaml` with the failing identities, the card hint and the fix commits.
- Merriam's strand is named `<shortname>:<slug>:webster`, pinned by a test, while `webster.yaml`'s `master` key still resolves its model.
- Test: `begin-batch` renders a card's gate command with `go test ./...` and `-tags integration` over the card's package directories, leaving out a Delete-only directory.
- Test: the shared function skips when the record names HEAD's tree and the command, runs when either differs, refuses a dirty tree, and writes the record only on a pass.
- Test: after a merge whose verify was interrupted before the record write, the next Publish runs verify.
- Test: a no-op merge onto a verified tree skips verify.
- Test: Publish with a dirty tree before merge-in, after merge-in or after verify is Stuck naming the paths and never pushes.
- Test: Finalize is Stuck on the same three dirty points and never merges into the parent.
- Test: a conflict session that edits a tracked non-conflicted file lands it in the merge commit, and one that creates an untracked file aborts stuck.
- Test: a second `Resolve` in the same scratch dir after an earlier conflicting one succeeds.
- Test: the Webster-Burler `verify` gate passes without running on an unchanged tree and re-prompts on a failing one.
- Test: a prompt over 30000 bytes launches, and the launch line carries the pointer to `prompt.md`, not the prompt.
- `lyx webster verify` exits 0 on a pass and 1 with findings on a failure; the parity test covers both gates and the verb.
- The package docs, `webster-spec.md`, `loom-plan-spec.md`'s verify model, `refusal-spec.md`, `docs/overview.md`, CLAUDE.md's Merriam entry and the new `CONSTRAINTS.md` invariant describe the result.

## Notes for the plan writer

- Retiring code: `runIntegrationStage`, `AwaitIntegration`, `bisect`/`LocalizeIntegrationFailure`/`BisectAndEscalate`, the triage baseline, `integrationfix.go`, `settleInterruptedFix`, the `IntegrationFix` state field, the `-1` integration record, and the stencils `webster-template-integration.md` and `webster-template-integration-fix.md`.
- `forkOwnReport` maps an out-of-batch fork transcript to the integration report; it must map the fixer fork to the gate report instead.
- `MasterStarter.StartMaster` already takes a `GateSpec` (empty today); check that the attach path for a resumed Merriam carries the same spec.
- `LoomVerifyPendingPath` and `verifyGate.check`'s `treeChanged` argument go; `LoomVerifyOutputPath` can become D7's log path.
- `internal/shedverbs/status.go`'s `waitingNote` serves `--watch`; confirm the one-shot status envelope surfaces the same note.
- `contracts/specs/loom-plan-spec.md` says the verify model is "designed, not implemented"; Tier 1 becomes implemented as D1, and Tier 2 gains D1's per-card integration-tagged run.
