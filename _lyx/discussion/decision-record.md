# Decision record — twb-twc-1: Webster stability

## Goal

Close the six Webster faults the board entry lists outside the audit, so no run reports `done` over an unfinished gate, Master leaves verify fixes to its fixer fork, a second recovery keeps the first one's work, and Create checks pass for build-constraint variants and say why when they fail.

The record faces the code at base `800a0e892`.

## Scope

In:

1. Master's turn-end wait holds on a running `lyx gate test` shell, whoever launched it, and on a `lyx webster recover-batch` shell, wherever in the command line the invocation sits (#505, #509, #517).
2. The verify gate's re-prompt to Master says Master spawns the fixer fork and never edits or commits.
3. A recovery prompt counts every recovery session of the run as the run's own write evidence (#522).
4. `lyx webster recover-batch` exits non-zero whenever its terminal digest is not `done` (#522).
5. The recovery prompt names the commands it may always run and leaves tmux- and llm-tagged steps to the verify gate.
6. The `create-not-done` and `rename-not-done` details say how the target was looked up and what quarry answered (#511).
7. An `ambiguous` answer whose candidates are build-constraint variants of one member counts as resolving, at Plan time and at record-batch, through one planglyph rule (#500, #507).

Out:

- quarry (`github.com/Knatte18/quarry`) is unchanged.
- Fork-side gate waiting, interim "fork finished" notices and `cd`/absolute-path noise in fork commands belong to `fork-wait-noise`.
- `#507`'s second remedy, a `Create` handle reusing a name its own card deletes.
- `#511`'s third point, reviving a dead batch through `lyx loom goto`.
- `lyx quarry resolve`'s own exit code on `ambiguous`.

## Decisions

### D1 — An awaited shell prefix matches at the start of any command segment

Decision: shuttle's awaited-shell match (`internal/shuttleengine#awaitedLabel`) tests each prefix in `Spec.AwaitedShellPrefixes` against the start of every segment the label splits into at the whole operators `&&`, `||`, `|`, `;` and a newline, each segment trimmed of whitespace; `2>&1` and other redirections are not operators.
`Spec.validate` refuses an empty or all-whitespace prefix.
Rationale: forks and Master run gates as `cd <pair> && lyx gate test … 2>&1 | tail -5`, and the provider labels a shell by its command (`claudeengine#payloadTask`, `claudeengine#launchKindAndLabel`).
The rule is provider-neutral string handling in `shuttleengine` (PATTERN-shuttle-provider-seam).
It does not reuse webster's audit classifier, which shuttle cannot import and which tokenizes for a different question; the segment rule is a few operators, not shell parsing.
Bound: the match widens only which shells hold a turn end.
A held turn end still ends on a later turn end, the run's `Timeout` or the liveness check, as a fork's does.
A quoted operator (`echo "x && lyx gate test"`) may match and hold; that costs a wait, never a false `done`.

### D2 — Master awaits `lyx gate test` shells, and an unfinished gate never reads as passed

Decision: Master's spawn (the `shuttleengine.Spec` that `websterengine` builds for `StartMaster`) declares `lyx gate test` beside `masterAwaitedShellPrefix`.
A waiting turn end with a gate shell outstanding therefore never counts at once (`shuttleengine#Run.countsAtOnce` admits only unawaited shells).
`webster-template-master.md` gains a rung: a turn that starts on a notification that is neither your fork's completion nor your `recover-batch` command's (a background command finishing, a fork's interim stop) changes nothing; end the turn without rewriting the outcome or summary.
When the run ends on its deadline or the liveness check with a gate shell still outstanding, shuttle classifies it `Done` because both output files exist (`classifyDeadlineExpiry`, PATTERN-completion-signal, unchanged), the verify entry reports waiting rather than passed, and webster's existing post-wait demotion ends the run `stuck`.
That stuck reason names each outstanding `lyx gate test` shell from `Result.EndedShells` and says its result was discarded.
Rationale: at base a batch fork's backgrounded gate shell left in Master's Stop payload (corpus case `master-fork-running`) lets the first gated arrival count at once and finish `done` while strand removal kills the shell.
Holding keeps the result observable; the demotion covers the timeout path, so a run never reads `done` over an unfinished gate.
Bound: as D1; a hung gate shell costs at most `master_timeout_min`.

### D3 — The verify re-prompt tells Master to fork, not to fix

Decision: `shuttleengine.GateEntry` gains an optional caller-supplied re-prompt text that replaces `gateRepromptText` for that entry; empty keeps today's text, and a text with a newline is refused at spec validation.
Webster's `verify` entry (`NewVerifyGate`) supplies one short line: `Gate findings recorded at <path> — do not edit, commit or fix anything yourself: follow "A gate failure: spawn one fixer fork" in your prompt, with the fixer prompt <verify-fix prompt path>.`
The line points at the template section rather than restating the fork prompt (PATTERN-producer-pointer-rule); it is a one-line runtime notice of the kind `gateRepromptText` already is, not a prompt body, so PATTERN-stencil-ownership is unaffected.
Rationale: at base the one line Master reads at a verify failure is shuttle's generic "read it, fix every finding, and end your turn", which instructs the self-fix the stencil forbids.
Bound: the short line stays well under tmux's message size (`paste-size-limit`).

### D4 — Every recovery session of the run is write evidence for uncommitted paths

Decision: a recovery's session id is recorded on its batch record when `recover-batch` resolves the spawned run, and read from the record's `ShuttleRunDir` run record when the respawn would replace a record that lacks it.
The batch record carries a list of earlier recovery session ids across respawns, the way it carries `Recoveries`.
`loadRunWrites` audits those sessions into a group of their own, beside its Master and fork groups.
Only the uncommitted-path classing (`uncommittedPathsBlock`) and the reset's own-path set (`ownTrackedPaths`) read that group; the contract-file evidence never does, so a recovery write to `outcome.yaml` or `summary.md` is never Master's.
A path any recorded recovery wrote renders under "Written by this run" in the next recovery prompt.
A recovery session whose transcript cannot be audited contributes no evidence and is logged at Warn; the spawn goes on, since attribution is a hint and its absence is today's behaviour.
Rationale: at base `loadRunWrites` reads only Master sessions and their forks, and a respawn replaces the record naming the earlier recovery, so #522's second recovery saw 19 of its batch's files as foreign.
The run-wide set is simpler than a per-batch one and no less true: every recovery is part of the run.
Bound: `lyx webster reset` now treats recovery-written tracked paths as the run's own.

### D5 — `recover-batch` exits non-zero unless the batch ends done

Decision: every emission of a terminal digest whose `status` is not `done` — a fresh terminal classification and a re-entrant call over an already-terminal record alike — exits non-zero, with the same digest fields on the failure envelope and a message naming the status and its way forward: `lyx webster recover-batch NN` once more when `recovery_retry` is true, else end the run stuck naming the batch.
A `done` digest and a `running` snapshot keep exit 0.
The new exit lands a row in `contracts/specs/refusal-spec.md`'s webster section with a test reaching it (PATTERN-refusal-way-forward), and leaves its friction note through `noteRefusals` like every webster refusal (PATTERN-friction-capture).
`webster-template-master.md`'s recover-batch rungs say the call exits non-zero for those statuses and still read the `status` field.
Rationale: the base `failed` path already exits non-zero; a terminal `stuck` or `dead` digest goes out through `output.Ok`, which is the exit 0 #522 reports for a batch that ended terminal.

### D6 — The recovery prompt names its allowed gates

Decision: `webster-prefix-recovery.md` gains a section: the recovery may always run its card gates, its cards' `verify:` lines and the batch gate as rendered below, `lyx gate test` over any package list, `lyx loom lint-comments`, `lyx quarry resolve`, and normal git to commit its cards; none is a fabric verb, and the sandbox rule bans only mutating `lyx fabric` verbs.
A step whose `--tags` value includes `tmux` or `llm`, in any spelling (`--tags tmux`, `--tags=tmux`, `--tags integration,tmux`), is not run by the recovery, is left to the plan-level verify gate and never makes the report `FAILED`.
Rationale: the board asks for exactly this; `llm` joins `tmux` because both tags switch on tmux servers in test binaries (`internal/testkit/tmuxkit/allow_tmux.go`).
Bound: the skip lives in the recovery prefix only; the shared implementer body is unchanged, so batch forks keep running every step.

### D7 — Build-constraint variants of one member resolve

Decision: planglyph gains one exported rule: an `ambiguous` answer is a variant set when every candidate sits in the same unit, in distinct files, and every pair of those files has disjoint build constraints.
A pair is disjoint when no single build context — one GOOS, one GOARCH, any set of other tags — satisfies both files under `go/build`'s matching: the `//go:build` line (else the legacy `// +build` lines), the filename's `_GOOS`, `_GOARCH` and `_GOOS_GOARCH` suffixes after a trailing `_test` is stripped, and the tags `go/build` implies (`unix` for the Unix GOOS values, `android` matching `linux`, `illumos` matching `solaris`, `ios` matching `darwin`).
The check enumerates assignments of the tags the pair names, at most 12 non-GOOS, non-GOARCH tags; past that bound, an unparsable constraint and a file with no constraint all count as overlapping, so the rule fails closed.
planglyph's status readers that treat `found` or `multipart` as resolving — the plan-time status findings, `DoneChecks`' Create and Rename-new bars, delete ordering — treat a variant set the same way.
A Delete or Rename-old done-check still needs `not_found`, so one surviving variant keeps `delete-not-done`; a Rename old side still needs `found` (`renameDeclSource`); a Create target that already resolves as a variant set still gets `create-already-exists`.
Rationale: quarry reports every declaration and declines to choose a build context (`statusForMatches`); loomyard can prove exclusivity without choosing one.
At Plan time the rule lets a variant set be an `Edit`, `Delete` or `Uses` target by glyph (#507), and any other `ambiguous` target keeps `glyph-ambiguous` there, as at base.
A Create whose implementation turns out per-tag is not knowable at Plan time; it now passes at record-batch.
Bound: only answers whose every candidate pair is provably exclusive pass, so a real duplicate declaration still blocks.

### D8 — Done-check details say how the target was looked up

Decision: the `create-not-done` and `rename-not-done` details name quarry's answer.
For `not_found` they say whether the unit exists and that the lookup covered every file of the unit's package, its same-package `_test.go` files and unexported names included, while an external test package (`package x_test`) is the separate unit `<dir>_test`.
For a non-variant `ambiguous` they name each candidate file and say their build constraints overlap.
The details ride the existing `batch_failed` refusal; no new refusal-spec row is needed.
Rationale: base `git grep` finds a single `rusageStats` (`cmd/testtiming/resources.go`) and a single `var tmuxAllowed` (`internal/testkit/tmuxkit/allow_none.go`), and `lyx quarry resolve` answers `found` for both, so unexported and test-file members already resolve.
#511's targets most likely failed as per-file variants, the shape D7 admits; this is inferred from #500 and the landed restructure, not observed. The old detail hid the difference.

## Constraints

- PATTERN-shuttle-provider-seam: D1's segment match and D3's per-entry re-prompt live in `shuttleengine` and stay provider-neutral.
- PATTERN-completion-signal is kept: D2 adds no negative finalize; the demotion to `stuck` happens in webster after shuttle's `Done`.
- PATTERN-refusal-way-forward and PATTERN-friction-capture: D5's exit carries its way forward, its row and a test, and leaves a note.
- PATTERN-stencil-ownership and PATTERN-producer-pointer-rule: stencil changes go into `contracts/stencils/webster/`; D3's line points at the template section.
- PATTERN-sole-parsers: D7 reads build constraints with `go/build/constraint`, which is no plan, discussion, summary or recipe parse.
- PATTERN-no-denied-recovery: D6 names only commands the agents' settings allow.
- PATTERN-test-economy and PATTERN-test-speed: scenarios extend existing tables where they exist; new top-level tests that push a package past its row list `cmd/lyx/testdata/test-budget.yaml` under `Edit:`.
- Docs in the same commit: `internal/shuttleengine/doc.go` (D1, D3), `internal/websterengine/doc.go` (D2–D5), `internal/planglyph/doc.go` (D7, D8), and `contracts/specs/webster-spec.md` where it covers recover-batch's exit or the recovery prompt.

## Auto-mode assumptions

- A fork's backgrounded shell reaches Master's Stop payload or transcript under its command label, as corpus case `master-fork-running` records (D2).
- A held turn end usually ends because the fork resumes on its shell's exit and completes again, which starts a Master turn; when no turn follows, D2's timeout path applies.
- A recovery strand's session can be audited for writes the way a Master session is, through `Engine.AuditForks` (D4).
- The board's "exited 0 while the batch ended failed" means a terminal non-`done` digest, since the base `failed` path already exits non-zero (D5).

## Open risks

- A tmux- or llm-tagged card `verify:` line the recovery skips (D6) is checked only when the plan's `## verify:` runs that tier; `weaver`, which plans to reuse the recovery prompt as its normal path, inherits the skip and D4's attribution.
- `fork-wait-noise` edits `webster-template-master.md` and the fork directives too; the tasks are independent, the second to land rebases its stencil text, and D2's rung must agree with its gate-waiting wording.
- `lyx quarry resolve` still exits non-zero on a variant set; a fork resolving its card glyphs still gets every candidate's file and span.

## Acceptance criteria

- A gated Master run whose turn end has both output files present and an outstanding shell labelled `cd <dir> && lyx gate test --tags integration ./x 2>&1 | tail -5` does not finish; the next turn end with no shell outstanding arrives at the verify gate.
- The same holds for a label starting `lyx gate test` and for `cd <dir> && lyx webster recover-batch 3`; a shell with no awaited prefix in any segment still lets the first gated arrival count at once.
- An empty awaited prefix is refused at spec validation.
- A held gate shell that outlives the run's `Timeout`: shuttle ends `Done`, webster ends the run `stuck` with a reason naming the gate shell and its discarded result; the same on the liveness path.
- A gate entry with a supplied re-prompt sends that text on failure; one without sends today's text; a supplied text with a newline is refused.
- Webster's verify re-prompt starts `Gate findings recorded at`, says not to fix it yourself and names the template section and the verify-fix prompt path.
- A second recovery whose first recovery left uncommitted writes renders those paths under "Written by this run", also after a Webster restart between the two; a path no session of the run wrote stays under "Not written by this run".
- A recovery session that wrote `outcome.yaml` is not counted as Master's contract-file evidence.
- A recovery session whose transcript is missing contributes nothing, and the spawn still succeeds.
- `lyx webster reset` treats a recovery-written tracked path as the run's own.
- `recover-batch` exits non-zero with the digest fields, the way forward and a friction note on a terminal `stuck` and a terminal `dead` (with `recovery_retry` true and false), also when re-called over an already-terminal record; it exits 0 on `done` and on a `running` snapshot.
- The rendered recovery prompt names the allowed commands and the tmux/llm skip.
- Variant sets: a member in a `//go:build tmux || llm` file and a `//go:build !tmux && !llm` file; in `_linux.go` and `_windows.go`; in a `//go:build unix` file and a `_windows.go` file.
  Each passes a Create done-check, still binds its `plan:` handle when the card creates both variants, and validates as an `Edit` and a `Delete` target at Plan time.
- Not variant sets, each keeping `glyph-ambiguous` at Plan time and `create-not-done` at record-batch: `_linux.go` beside `//go:build unix`; an unconstrained file beside a tagged one; an unparsable `//go:build` line; a pair naming more tags than the bound; candidates in two units.
- `foo_linux_test.go` and `foo_windows_test.go` form a variant set.
- A Delete of a variant set with one variant left stays `delete-not-done`; a Create target already resolving as a variant set gets `create-already-exists`.
- A `create-not-done` detail for `not_found` names the lookup, the unit's existence and the external-test-unit rule; for a non-variant `ambiguous` it names each candidate file.

## Notes for the plan writer

- At base, `internal/proc#StartTime` resolves `ambiguous` (`proc_linux.go`, `proc_windows.go`), a ready fixture shape for D7.
- `BindHandles` matches handles against the post-batch delta by symbol id, not by resolve status, so D7 may need no change there; the acceptance line pins the outcome.
- D4's render and evidence points are `uncommittedPathsBlock` in `recoverbatch.go` and `loadRunWrites`/`recordedSessions` in `runwrites.go`.
- Webster's post-wait demotion of a done run whose gate did not pass is described in `internal/websterengine/doc.go` under the verify gate.
