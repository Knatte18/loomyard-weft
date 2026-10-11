## Goal

Close the Webster faults that stop or mislead runs outside the audit (board entry `twb-combo-1`: #505, #509, #517, #522, #511, #500, and #507 in part), facing base `800a0e892`.

Base facts the design rests on:

- Shells no longer expire: `ShellWaitBound` only logs, and `Result.EndedShells`, the `webster-background-shell` friction note and a `RunResult.Warnings` entry record a shell outstanding at the run's end.
  What still cuts a gate shell off is `expiredTurnEnd`'s count-at-once rule (`countsAtOnce`): a fresh gated run's first arrival with every output file present counts at once when every outstanding task is an unawaited shell.
  Master's only awaited prefix is `masterAwaitedShellPrefix` (`lyx webster recover-batch`), and `awaitedLabel` matches a prefix at the start of the label, which is the shell's whole command line (`launchKindAndLabel`, `payloadTask`).
- The issues name the cut-off shells as `lyx gate test --tags integration` runs, some piped through `grep` and `tail`.
  The `master-fork-running` corpus case shows a shell a returned fork left in Master's turn end, but its command is a scratch script, not a gate run.
- After a verify-gate failure, Master reads shuttle's `gateRepromptText` ("read it, fix every finding, and end your turn") and the findings text `renderVerifyGateFindings` returns as `GateResult.Findings`, which shuttle writes to the run directory's `gate-findings.md`; neither names the fixer fork.
- `loadRunWrites` audits only `State.MasterSessionID` and each batch's `SessionID` (a Master session, empty for a recovery batch); no recovery session is stored, and each recovery spawn replaces the `BatchState`.
- `recover-batch` returns `output.Ok`, exit 0, for every terminal digest, `stuck` and `dead` included.
- Quarry v0.2.1 has no exported-name filter (`internal/websterengine#ownTrackedPaths` and `cmd/testtiming#rusageStats` answer `found`), and its walker puts same-package `_test.go` files in the package's unit, so #511's "unexported" premise is wrong.
  Quarry ignores build constraints: a member declared once per file-level constraint answers `ambiguous` (`internal/proc#StartTime` over `proc_linux.go` and `proc_windows.go`), which `doneCheckVerdicts` never counts as created.
- Nothing refuses `lyx gate test --tags tmux` from a strand: `rawGoCommandPattern` denies only raw `go test -tags tmux|llm`, and the implementer body the recovery shares says a `tmux`-tier run goes through `lyx gate test`.

## Scope

In:

1. Master's wait holds a gated turn end while a `lyx gate test` shell is outstanding, and Master's template says what Master does when such a shell completes.
2. The verify-gate failure texts Master reads name the fixer-fork path and forbid Master's own edits.
3. A recovery's prompt counts earlier recoveries of the same batch as the batch's own work.
4. `recover-batch` exits non-zero for a terminal `stuck` or `dead` digest, and Master's ladder rungs say so.
5. The recovery prompt names its allowed gate commands, `tmux`- and `llm`-tagged steps included.
6. Create and Rename-new done-checks accept an ambiguous answer whose candidate files have pairwise disjoint build constraints; the `create-not-done`, `rename-not-done` and `glyph-ambiguous` details say how the target was looked up and what came back.

Out:

- How forks and Master run gates (in-turn waits, foreground quick checks, no `cd`), the fork directive and the implementer body: board entry `fork-wait-noise`.
  This task owns the Master template's verify-gate failure section, its failure-ladder rungs for `recover-batch`, and a new rung for a completed gate shell.
- Stopping a shell from lyx.
- Quarry honouring build constraints.
- #507's Edit and Delete by glyph for an existing per-platform member, and its re-create remedy: the member keeps its Plan-time `glyph-ambiguous` finding, now naming the file-path spelling that the gate accepts.
- `lyx webster reset`'s own-path refusal: unchanged.
- `record-batch`'s exit status, and reviving a dead batch through `lyx loom goto`.

## Decisions

### 1. Master's wait holds on a running `lyx gate test` shell

- Decision: `awaitedLabel` matches each awaited prefix at the start of every simple command in the label.
  A simple command starts at the label's start or after `&&`, `||`, `;`, `|`, `&`, a newline or an opening `(`, after leading whitespace and `VAR=value` assignments; a leading `cd <dir> &&` therefore needs no special case.
  Webster's Master spawn adds `lyx gate test` to its `AwaitedShellPrefixes`.
  The splitter is read-only label parsing inside `internal/shuttleengine`, not command building, so `internal/shell` keeps its role.
- Decision: the Master template gains a rung: when a backgrounded `lyx gate test` command's completion notification starts a turn, Master edits nothing; a zero exit leaves its outcome and summary as they are and ends the turn, and a non-zero exit rewrites the outcome as `stuck` with a `stuck_reason` naming the command and quoting its last lines, then ends the turn.
- Decision: for a shell outstanding at the run's end whose label matches `lyx gate test`, the existing note and warning add that the gate's result was discarded and is not a pass.
- Rationale: the hold stops a turn end from counting while a gate runs, so strand removal no longer kills it unobserved, and the rung gives the result a reader, so a failing gate cannot end the run `done`.
  This is the wait-side backstop whatever forks do, so it stays useful after `fork-wait-noise` makes forks wait for their gates in-turn.
- Bound: an awaited gate shell holds Master's turn end until a later turn end no longer lists it, the run's `Timeout` (`MasterTimeoutMin`) or the liveness check; a run ended by its deadline with the shell outstanding is a timeout, never `done`.
  It removes only the count-at-once exit for that shell; an attached or resumed run, which never counts at once, is unchanged.
  A gate run wrapped in a script or `bash -c` is not recognised and still counts at once with the existing warning.

### 2. The verify-gate failure texts carry the fixer-fork rule

- Decision: `shuttleengine.GateEntry` gains an optional one-line re-prompt text; empty keeps `gateRepromptText` for every other gated run.
  Webster's verify entry sets a line naming the findings path, saying Master edits nothing, and telling it to spawn the one fixer fork with the prompt line at the top of that file, then rewrite the outcome and summary.
- Decision: every variant of the findings text (failing identities, timeout, dirty tree) opens with the same do-not-edit instruction and the ready-to-paste fixer fork prompt line naming the verify-fix prompt path; the dirty-tree "Commit or remove these paths" becomes the fixer's work.
- Decision: both texts live in a webster stencil read through `internal/stencilstore` at call time, with Go filling only the paths, so no agent instruction becomes a Go literal.
- Decision: the Master template's "A gate failure: spawn one fixer fork" section stays the long form and points at the prompt line in the findings file instead of having Master assemble it; webster `doc.go`'s line saying the message names `VerifyGateReportPath` is corrected to the shuttle findings file.
- Rationale: the texts Master reads at the moment of failure said "fix", which outweighed a rule read once at the start.
- Bound: the per-entry text replaces only the re-prompt wording of the entry that sets it, stays one line (`validateSendText`), and changes no verdict, attempt count or budget.

### 3. A recovery counts the batch's earlier recoveries as its own work

- Decision: `BatchState` gains a list of recovery session ids, one per recovery spawn of the batch, counted or not, carried onto the fresh record by every later recovery spawn and re-begin.
  The id is recorded in the state the spawn saves before the bounded wait begins.
- Decision: `RunWrites` gains a `Recoveries` field, filled by auditing the target batch's recorded recovery sessions; the recovery prompt's "Written by this run" group reads it beside the Master and fork writes.
  The contract-file evidence and the reset's own paths keep reading `Master` and `Forks` only.
- Decision: a recorded recovery session whose transcript is absent, truncated or unparseable counts as no evidence, never as an error, so its paths fall under "Not written by this run" and no spawn fails on it.
- Rationale: attribution by batch, as the board expects, keeps the first attempt's work, which the prompt already tells the strand to check against the card before building on it.
- Bound: only the target batch's own recovery sessions are adopted; a crash between the strand's start and the state save leaves that one recovery's writes unattributed, as today.

### 4. `recover-batch` exits non-zero on a terminal stuck or dead digest

- Decision: a terminal `stuck` or `dead` digest returns through the error form, exit non-zero, carrying today's fields (`batch`, `status`, `dead_reason`, `recovery_retry`, `recoveries`, `warnings` and the digest fields); `done` and the operator's `running` snapshot stay exit 0, and `failed` and the refusals are unchanged.
  The error text names the batch, the status and the way forward: run `recover-batch` once more when `recovery_retry` is true, else stop the run stuck.
- Decision: the Master template's stuck and dead rungs say the digest arrives with a non-zero exit and an error envelope that carries `status`, and that such an envelope is a terminal digest, not a refusal or a transient error to re-run.
- Rationale: a recovery that ends the batch without its work is a failed command, and an operator or script reading the exit must see it.
- Bound: changes the exit status and the `ok`/`error` keys of those two outcomes only; it is not a refusal, so it adds no `refusal-spec.md` row.

### 5. The recovery prompt names its allowed gates

- Decision: `webster-prefix-recovery.md` states that the card gate and batch gate steps in the prompt, every card `verify:` step, `lyx gate test` with or without `--tags` and `lyx loom lint-comments` are allowed and are not `lyx fabric` commands, so the sandbox rule never covers them.
  A `tmux`- or `llm`-tagged step runs through `lyx gate test --tags <tag>` like any other step, as the shared implementer body already says.
- Rationale: the board expected tagged steps to be left to verify, but nothing at base refuses them, and a skipped step would go unrecorded, since the report has no field for it; running them through the allowed command keeps one rule in both prompt parts.
- Bound: grants nothing new: every command named is already allowed by the settings, and the prompt names no raw `go` command.

### 6. A member declared once per build-constraint set passes its Create check

- Decision: one predicate in `internal/planglyph` reports whether an ambiguous answer's candidates sit in files whose build constraints are pairwise disjoint.
  A file's constraint is its filename's `_GOOS`, `_GOARCH` or `_GOOS_GOARCH` suffix (after stripping `_test`) ANDed with its `//go:build` line, parsed once per file with `go/build/constraint` and evaluated in memory.
  Two files are disjoint when no configuration satisfies both: one GOOS and one GOARCH from a closed table plus a sentinel matching no known name, the implied tags (`unix`, `android`→`linux`, `ios`→`darwin`, `illumos`→`solaris`), and every other tag either file mentions as a free boolean.
- Decision: `doneCheckVerdicts` counts such an answer as resolved for `create-not-done` and `rename-not-done-new`; the Delete and Rename-old checks keep counting any non-`not_found` answer as still present.
- Decision: the `create-not-done` and `rename-not-done` details name the key resolved (a `plan:` handle's glyph body), that the worktree tree with untracked files was searched, the answer's status, the unit's answer for a `not_found`, and for an ambiguous answer its candidate files and whether they are disjoint.
  The Plan-time `glyph-ambiguous` detail says when the candidates are disjoint and names the file-path spelling as the way to target the member.
- Decision: no other consumer of a resolve answer changes.
- Rationale: the done-check is where #500 and #511 failed.
  A Create target answers `not_found` at Plan time, so no validator can see beforehand that it will be split per platform.
  Edit and Delete targets keep `glyph-ambiguous` because the downstream glyph consumers (scope guard, delete order, drift, rename) read ambiguous as unresolved, while the file-path spelling already passes them.
- Bound: the predicate fails closed: same-file candidates, an unreadable file, an unparseable `//go:build` line, an answer with no candidate files and more than eight free tags across a pair all count as not disjoint.

## Constraints

- `PATTERN-shuttle-provider-seam`: the command-start matching is provider-neutral string logic in `internal/shuttleengine`.
- `PATTERN-completion-signal`: Decision 1 changes when a waiting turn end counts, never the output-file check a negative finish answer consults first.
- `PATTERN-stencil-ownership`: every new or moved agent instruction (Decisions 1, 2, 4, 5) lives in a stencil read at call time.
- `PATTERN-no-denied-recovery`: no prompt names a command the agents' settings deny.
- Docs in the same commit: shuttle `doc.go` (the waiting turn end paragraph, `AwaitedShellPrefixes`, the per-entry re-prompt), webster `doc.go` ("Background shells in Master's wait", the "Gate findings recorded at …" line, the recovery uncommitted-paths paragraph, `recover-batch`'s exit), `planglyph/doc.go` (the done-check list) and `contracts/specs/loom-plan-spec.md` where it describes the done-checks.
- `contracts/stencils/claims_test.go` and `internal/websterengine/template_test.go` pin Master and recovery prompt text; a stencil edit updates its pins in the same card.

## Auto-mode assumptions

- #522's "exited 0 while the batch ended failed" is read as a terminal `stuck` or `dead` digest, since no `failed` path exits 0 at base.
- `lyx gate test` is the only shell prefix Webster awaits beyond the recovery verb.
- A completed `lyx gate test` shell's exit code and output tail reach Master in its completion notification, as a backgrounded `recover-batch`'s output does.

## Open risks

- The hold releases only when a later Master turn end stops listing the shell; if a returned fork's shell ending never starts a Master turn, every held gate shell runs Master to `MasterTimeoutMin`.
  The plan must confirm the notification from a real transcript or the claudeengine corpus before relying on it.
- If #522's real envelope came from another path, its exit-0 symptom stays.
- `BindHandles` was not exercised against a per-GOOS delta: two `Created` symbols sharing one ID must bind one handle, not trip `bind-count-mismatch`.
- `fork-wait-noise` edits the same shuttle doc text and other Master template sections; whichever lands second rebases onto the other.
- The GOOS/GOARCH table is another copy beside `cmd/lyx/tmuxisolation_test.go`'s; a port missing from it can only make a pair read not disjoint.
  A later quarry that honours build constraints (`enclose-kickstart` plans a quarry bump) would make the predicate redundant.
- `weaver` plans to reuse the recovery prompt and `recover-batch`; Decisions 3 and 4 change that surface.

## Acceptance criteria

- A fresh gated Master run whose turn end has every output file present and a `lyx gate test` shell outstanding keeps waiting, for a plain, a `cd <dir> && lyx gate test …` and a piped `lyx gate test … | tail` label; a later turn end without the shell releases the hold; an unrelated shell still counts at once; an attached run behaves as before.
- A run ended by deadline with a `lyx gate test` shell outstanding ends as a timeout, records the shell on `EndedShells`, and its note and warning say the gate result was discarded.
- The rendered Master template carries the completed-gate-shell rung, and its stuck and dead `recover-batch` rungs describe the non-zero error envelope.
- A verify-gate failure sends Master the webster entry's one-line re-prompt, and each findings variant opens with the do-not-edit instruction and the fixer fork prompt line naming the verify-fix prompt path; another gated run still receives `gateRepromptText`.
- A second recovery of a batch whose first recovery left uncommitted files lists them under "Written by this run"; a path no recorded session wrote, or another batch's recovery wrote, stays under "Not written by this run"; a recorded recovery session whose transcript is missing or unparseable lists its paths as not written and fails nothing.
- The recovery session list survives a re-begin, a counted respawn and an amendment respawn.
- `recover-batch` exits non-zero with the digest fields for a terminal `stuck` and a terminal `dead` digest, and 0 for `done`.
- The rendered recovery prompt names the card and batch gates, `verify:` steps, `lyx gate test` with tags and `lyx loom lint-comments` as allowed and not fabric commands, and does not contradict the implementer body.
- A Create whose member is declared in `x_linux.go` and `x_windows.go`, or in a `//go:build tmux || llm` file and a `//go:build !tmux && !llm` file, passes `create-not-done`; one declared in two files with overlapping or no constraints still fails, and so does a same-file duplicate.
- A `create-not-done` detail names the resolved key, the status, the unit answer for `not_found`, and for an ambiguous answer its candidate files and whether they are disjoint.
- Delete and Rename-old checks still fail while any per-platform declaration remains.

## Notes for the plan writer

- `countsAtOnce` and `expiredTurnEnd` are in `internal/shuttleengine/wait.go`; `masterAwaitedShellPrefix`, the Master spawn's `AwaitedShellPrefixes` and `verifyFixPromptPath` are in `internal/websterengine/runlevel.go`.
- `recordedSessions` and `loadRunWrites` are in `internal/websterengine/runwrites.go`; the recovery strand's session id has to be read from its shuttle run record at spawn.
- `candidateList` in `internal/planglyph/resolve.go` already renders candidates for findings.
