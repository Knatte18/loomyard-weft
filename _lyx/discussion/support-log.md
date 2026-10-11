# Support log — twb-seat-blind-1

## Interview

Autonomous run: the chair explored base `800a0e892`, then put two bare question batches to `ly:twb-seat-blind-1:discussion-advisor-1`.

### Turn 1 — chair exploration

- Shuttle's outstanding-shell rule (`internal/shuttleengine/doc.go`), Master's spec and done mapping (`internal/websterengine/runlevel.go`, `triage.go`, `summary.go`).
- The verify gate's re-prompt, `gateRepromptText` in `internal/shuttleengine/gate.go`: "read it, fix every finding, and end your turn".
- Recovery write evidence (`runwrites.go`, `state.go`) and `recover-batch`'s exit paths (`internal/webstercli/recoverbatch.go`).
- `internal/planglyph` DoneChecks and Plan-time rules; live `lyx quarry resolve` on unexported, test-file and per-GOOS members.

### Turn 2 — batch 1 (§1–§3) to the advisor

Advisor answers relied on (`ly:twb-seat-blind-1:discussion-advisor-1`):

- §1: at base `background_shell_wait_min` expires nothing; the cut-off is `countsAtOnce` (fresh run, first gated arrival, only unawaited shells, outputs present). Base already records ended shells as warnings, a friction note and a done-only summary section, but never says the result was discarded. Master's stencil sanctions no `lyx gate` run; `awaitedLabel` is a bare `HasPrefix` on the command.
- §2: the generic re-prompt tells Master to fix; the findings file names no next step and its dirty-tree arm says "Commit or remove these paths"; the rule sits only in the stencil.
- §3: `recordedSessions` covers Master sessions and fork batches only; a recovery strand's session is in neither, so its writes read foreign. `PlanReset`'s own-path exemption and `contractWritesFor` read the same evidence. `recover-batch` exits 0 on every terminal digest; only Master's completion notice reads the exit code.

### Turn 3 — batch 2 (§4–§6) to the advisor

Advisor answers relied on (`ly:twb-seat-blind-1:discussion-advisor-1`):

- §4: no deny blocks the gate commands; the cause is prompt wording; tmux runs at Publish per `PATTERN-test-speed`.
- §5: quarry resolves unexported and `_test.go` members; #511 was #500's ambiguity under the wrong words.
- §6: recognize build-disjoint ambiguity in `planglyph`'s resolve site as multipart; disjointness by enumeration over the two files' constraints with GOOS/GOARCH as exclusive groups, failing closed; Rename stays refused with a truthful message; Plan time can flag only existing members.

## Rejected alternatives

- **§1 — lyx stops the shell and records it discarded.** The provider reports no pid, so lyx cannot stop a shell.
- **§1 — demote a done run to stuck when any shell is outstanding.** It also demotes the harmless leftover-server case `countsAtOnce` exists for; once gate shells are held, a done run cannot end over one, so a gate-only demotion guards nothing.
- **§1 — record-only (no hold).** The record would be truthful but the gate result would still be lost.
- **§2 — a `lyx webster` verb that spawns the fixer.** A verb cannot spawn an in-session fork; a cold strand is a new mechanism beside the fork-based fixer.
- **§2 — a PreToolUse deny on Master's writes.** Provider-level and new; the run-exit audit already catches the write, and the fix is to stop Master reaching for it.
- **§2 — stencil wording only.** The stencil already states the rule; the message Master reads last contradicts it.
- **§3 — second recovery starts from a clean tree with the first diff as reference.** Destructive to possibly good work, and it routes through fabric destruction.
- **§3 — every uncommitted path counts as the batch's own on a respawn.** It also claims files the operator edited.
- **§4 — the statement in the recovery prefix only.** Forks run the same gates under the same sandbox rule; the body is the one place both read.
- **§4 — tmux-tagged steps run like any other.** Contradicts `PATTERN-test-speed`: the tmux tier runs at Publish and by hand.
- **§5 — change quarry or DoneChecks resolution for unexported or test-file members.** Both already resolve.
- **§6 — quarry honours build constraints.** Another repo, a release and a `go.mod` bump, and quarry would still need a caller-stated target platform; the candidates' files are all lyx needs.
- **§6 — DoneChecks only.** Leaves Edit, Uses and Delete of existing per-GOOS members blocked at Plan time (#507).
- **§6 — a Plan-time flag on a new Create.** The glyph resolves not_found before the card runs; nothing in it says the member will be split.
- **§6 — #507's Create reusing a name the same card deletes.** Independent of ambiguity, and it needs `createFindings`, `doneCheckVerdicts` and `deleteorder` to agree on one key; its own design.

## Review rounds

_No rounds yet._

## Question ledger

Autonomous mode: no operator answers; each pick below is the chair's, weighed against advisor `ly:twb-seat-blind-1:discussion-advisor-1`.
The code faced is base `800a0e892`, per the board entry's bench note.

### Batch 1 — private picks, before advisors

- **Q1 — background gate shell at Master's turn end (§1).**
  Private pick, before advisors: Master's spec awaits `lyx gate test` shells like `recover-batch` (the awaited-prefix match also holds at the start of any `&&`, `;` or `|` segment of the label), bounded by `master_timeout_min` and liveness; as a guard, `finishMasterDone` demotes a done outcome to stuck when `EndedShells` holds a gate shell.
  Alternatives seen: (b) keep finishing done but have lyx stop the shell and record the result as discarded (lyx has no pid, so it cannot stop the shell); (c) demote only, without holding; (d) forbid Master from running gate shells at all in its stencil.
- **Q2 — Master fixing a verify failure itself (§2).**
  Private pick, before advisors: `shuttleengine.GateEntry` gains an optional per-entry re-prompt text; webster's verify gate sets it to a line that says not to edit and to spawn one fixer fork from the verify-fix prompt file; the verify-gate report opens with the same rule.
  Alternatives seen: (b) a `lyx webster` verb that spawns the fixer (an in-session fork cannot be spawned from Go, so it would be a separate strand); (c) stencil wording only; (d) the verify gate itself detects a Master write at the next arrival.
- **Q3 — second recovery reads the first recovery's files as foreign; exit code (§3).**
  Private pick, before advisors: each recovery spawn records its provider session id on the batch record, carried forward across recovery records; `loadRunWrites` audits those sessions too, so their writes count as the run's own; `recover-batch` exits non-zero on a terminal stuck or dead digest, with the digest fields still in the error envelope.
  Alternatives seen: (b) second recovery starts from a clean tree with the first attempt's diff as reference (a destructive reset of uncommitted work); (c) attribute by path set of the batch's declared targets.

### Batch 1 — advisor answers and settlements

- **Q1 — differed in part, added an option.**
  The advisor corrected the premise: at base `background_shell_wait_min` expires nothing; the cut-off is `countsAtOnce` at the first gated arrival, for unawaited shells only.
  It agreed on holding `lyx gate` shells through Master's awaited prefixes, flagged the bare `HasPrefix` miss for `cd … && lyx gate …`, rejected a demotion to stuck as also catching the harmless leftover-server case, and added option (b): every shell warning says its result was discarded, and the summary section appears on every outcome.
  It noted Master's stencil sanctions no gate run, so a Master gate shell is off-script.
  Settled (advisor-informed, `ly:twb-seat-blind-1:discussion-advisor-1`): hold `lyx gate` shells, with the awaited-prefix match applied at the start of every command segment; the record says the result was discarded on every outcome; no demotion, since once gate shells are held a done run cannot end over one.
- **Q2 — agreed, added detail.**
  The advisor found the findings file's dirty-tree arm also instructs the reader ("Commit or remove these paths") and that the card hint invites localizing.
  Settled (advisor-informed): per-entry re-prompt text on `GateEntry`, the verify gate's line points at the fixer-fork section; the findings file opens with the same next step and its dirty-tree arm addresses the fixer fork.
- **Q3 — agreed, widened the reader list.**
  The advisor confirmed attribution is by Claude session, named `PlanReset`'s own-path exemption (and fabric destroy's dirtiness check fed from it) and `contractWritesFor` as readers of the same evidence, and asked that `AuditForks` be checked on a fork-less recovery transcript at Plan time.
  On the exit code it found no Go consumer, only Master's completion notice, and noted `record-batch` exits 0 on a stuck fork digest.
  Settled (advisor-informed): pick (A) as privately chosen; `recover-batch` exits non-zero on stuck and dead, 0 on done and on a running snapshot; `record-batch` stays as it is.

### Batch 2 — private picks, before advisors

- **Q4 — the recovery prompt's allowed gates (§4).**
  Private pick, before advisors: the recovery prefix states that the card and batch gate steps it renders, `lyx gate test` in any tier and `lyx loom lint-comments`, are always allowed and are not fabric verbs; tmux-tagged steps run like any other, since nothing at base refuses them in a strand.
  Alternatives seen: (b) as the board says, tmux-tagged steps are left to the plan-level verify; (c) put the statement in the shared implementer body so forks read it too.
- **Q5 — `create-not-done` on unexported targets (§5).**
  Private pick, before advisors: quarry already resolves unexported and `_test.go` members (checked live: `cmd/testtiming#rusageStats`, `cmd/testtiming#runStandInGo` resolve found), so the #511 failures were per-file ambiguity; the fix is the refusal text: the looked-up glyph, quarry's status, and for ambiguous each candidate's file and build constraint, plus a test pinning that an unexported and a test-file Create pass.
  Alternatives seen: (b) change resolution, since it is not broken; (c) leave the text.
- **Q6 — build-tagged and per-GOOS members resolving ambiguous (§6).**
  Private pick, before advisors: one `planglyph` helper reads an ambiguous answer whose candidates sit in files with pairwise disjoint build constraints (filename GOOS/GOARCH suffixes included) as a build-variant member; every planglyph rule reads it as it reads multipart (DoneChecks Create and Rename-new pass, Plan-time status policy passes Edit and Uses, Create of it is `create-already-exists`, Delete still requires not_found); quarry is unchanged; the Plan-time flag is the status policy passing variants and still blocking non-disjoint ambiguity; #507's delete-then-recreate-in-one-card rule is out.
  Alternatives seen: (b) quarry honours build constraints (a cross-repo change); (c) DoneChecks only, leaving Plan time to block Edit of per-GOOS members; (d) a Plan-time check on a new Create that cannot know its future files.

### Batch 2 — advisor answers and settlements

- **Q4 — differed.**
  The advisor found no deny on `lyx gate test` or `lyx loom lint-comments`; the confusion is the body's "never run a mutating `lyx fabric` verb" and the prefix's "never run `lyx webster`" beside Go-rendered gate steps, plus body line "a `tmux`-tier run goes through `lyx gate test`", which reads as permission while `PATTERN-test-speed` runs the tmux tier at Publish only.
  It recommended the statement in the shared implementer body, read by fork and recovery alike, and tmux- and llm-tagged steps compiled with `go vet -tags <tag>` and left to verify and Publish.
  Settled (advisor-informed): the advisor's version; my private pick (run tmux steps like any other) contradicted `PATTERN-test-speed`, which I checked.
- **Q5 — agreed.**
  The advisor confirmed unexported and `_test.go` members resolve, traced #511 to the same run as #500 (per-file ambiguity reported as "still does not resolve"), and asked that the refusal follow `PATTERN-refusal-way-forward`.
  Settled (advisor-informed): text fix and a resolution test, no resolver change; #511's `goto`-revives-a-dead-batch ask is out of scope.
- **Q6 — agreed on the outcome, added the site and corrections.**
  The advisor placed the recognition in `planglyph`'s single resolve site, rewriting a build-disjoint ambiguous answer to multipart so every rule follows; found `answerSymbols` returns nothing for ambiguous today, so delete-before-reference skips such a member; kept Rename refused (`renameDeclSource` requires found) with a truthful message; added an informational Plan-time finding naming the build-split candidates; specified disjointness by enumeration with fail-closed limits.
  It corrected two facts: Edit fails at Plan time (`glyph-ambiguous`), not at record-batch, and `internal/gatecli#runChild` does not exist at base.
  It recommended leaving #507's delete-and-recreate-in-one-card rule out, as an independent design touching three rules.
  Settled (advisor-informed): the advisor's version; #507's other ask is out and goes to the board as a follow-up for the orch to file.

### Other picks

- Auto-pick: Master's stencil gains one never-line against running `lyx gate` or any test itself, beside the re-prompt fix, since the advisor found no sanctioned Master gate run.
- Auto-pick: `record-batch`'s exit code on a stuck fork digest is unchanged; a stuck fork is the normal entry to the failure ladder, while a terminal stuck or dead recovery is the ladder's own failure.
- Environment: nothing outside the two output files looked broken.
