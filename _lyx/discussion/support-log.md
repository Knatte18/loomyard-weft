## Interview

### Turn 1 — chair exploration (autonomous mode)

Read the board entry and the related entries `fork-wait-noise`, `proc-child-lifetime`, `verify-watchdog`, `weaver`, `kick-start-pack`, `glyph-practice-audit` and `discussion-premise-cites`, and the GitHub issues #505, #509, #517, #522, #511, #500 and #507.
Read the base `800a0e892` with `git show`: shuttle `doc.go`, `wait.go` (`expiredTurnEnd`, `ShellWaitBound`, `awaitedLabel`), `gate.go` (`gateRepromptText`, `GateEntry`), `claudeengine/background.go` (`launchKindAndLabel`), webster `doc.go`, `runlevel.go`, `recoverbatch.go`, `runwrites.go`, `state.go` (`BatchState`), `verifygatereport.go`, `cardgate.go`, the Master template, the recovery prefix and implementer body, `webstercli/recoverbatch.go`, `planglyph` `donecheck.go`, `resolve.go`, `create.go`, `handle.go`, `claudeengine/settings.go`.
Ran `lyx quarry resolve` on `internal/proc#StartTime` (ambiguous over `proc_linux.go`/`proc_windows.go`), `internal/websterengine#ownTrackedPaths` and `cmd/testtiming#rusageStats` (both found), and checked quarry v0.2.1's walker: no exported filter, same-package test files in the package's unit.

Findings: shell expiry is already gone at base, and the remaining cut-off is the count-at-once rule; the re-prompt text says "fix every finding"; recovery sessions are never write evidence; `recover-batch` exits 0 on stuck and dead; #511's unexported premise is false.

### Turn 2 — advisor notes, unprompted

`ly:twb-combo-1:discussion-advisor-1` sent its notes before any question: the same four findings (count-at-once, the "fix" wording in `gateRepromptText` and `renderVerifyGateFindings`, recovery sessions missing from `loadRunWrites`, quarry ignoring build constraints), plus the overlap with `fork-wait-noise` and the refusal-spec row requirement.

### Turn 3 — round 1, six questions to the advisor

Sent bare questions on: the gate shell at a gated turn end and how to recognise it (Q1), what makes Master take the fixer path (Q2), how a recovery treats an earlier recovery's work and what other `RunWrites` readers must see (Q3), `recover-batch` exit statuses (Q4), the recovery prompt's allowed commands and tagged steps (Q5), where disjoint-constraint ambiguity is accepted and what "disjoint" means (Q6).

Advisor `ly:twb-combo-1:discussion-advisor-1` answered:

- Q1: hold the turn end on a `lyx gate test` shell at every arrival; recognise it per simple command (split on separators, skip `cd … &&` and `VAR=x`), as a new caller-data field; a contains-match also hits `grep`/`echo`; stopping the shell is infeasible (no pid).
- Q2: put the instruction and the ready fork prompt line at the top of the findings file (pass `verifyFixPromptPath` into `NewVerifyGate`), add an optional one-line per-entry re-prompt on `GateEntry`; a spawning verb cannot work; a PreToolUse deny is a hard guard but touches `standingDenies` and fork mode.
- Q3: a recovery-session list on `BatchState` carried across spawns (amendment respawns exceed two); a new `RunWrites.Recoveries` field kept out of `contractevidence.go`; including it in `ownTrackedPaths` is a behaviour change for the chair to decide; tolerate a missing recovery transcript.
- Q4: today done/stuck/dead exit 0, failed and refusals exit 1; the ladder reads JSON fields, never the exit code; recommend stuck and dead exit 1 with the same fields, plus a refusal-spec row.
- Q5: the settings deny raw tagged `go test`, not `lyx gate test` nor `go vet -tags`; name the gates as allowed, compile-check tagged steps with `go vet -tags`, report them left to verify; whether verify covers those tiers is not shown by code.
- Q6: one predicate in `planglyph`, applied in `doneCheckVerdicts` only; `statusFindings` must not accept because expand requires `found`; Create targets cannot be flagged at Plan time; disjointness over filename suffix AND `//go:build`, enumerated over a closed GOOS/GOARCH table with a sentinel and implied tags, failing closed; name status and candidates in `create-not-done`.

### Turn 4 — chair settles the design

Settled Decisions 1 to 6 as the record states, weighing each answer against the private pick in the ledger.

### Turn 5 — lens review

Six forks returned reports; none was missing.
Duplicates across lenses merged.

- scenarios BLOCKING, D1 hold releases only on a new turn end: fixed; bound restated (later turn end, Timeout or liveness; deadline end is a timeout), confirmation of the notification moved to Open risks, release scenario added.
- scenarios + goal-scope MEDIUM, D1 nobody reads a held gate shell's result: fixed; new Master rung (zero exit unchanged, non-zero writes stuck), owned by this task.
- premises MEDIUM, D1 corpus shell was a script, not a gate run: fixed; Goal bullet corrected, wrapped runs named in D1's bound.
- scenarios LOW, resume path unpinned: fixed; attached-run scenario added.
- board-fit MEDIUM + LOW, `fork-wait-noise` section ownership and backstop relation: fixed; Scope Out names the Master sections each task owns, D1 rationale names it the backstop.
- pattern-fit MEDIUM, D2 Go literal of agent instructions: fixed; texts move to a webster stencil read through stencilstore.
- premises LOW, D2 which findings file: fixed; named shuttle's `gate-findings.md`, doc.go line correction added.
- pattern-fit + scenarios MEDIUM, D4 is not a refusal and the ladder rungs read refusals: fixed; no refusal-spec row, Master stuck/dead rungs updated.
- goal-scope MEDIUM + pattern-fit LOW, D3 reset widening unlicensed: fixed; reset dropped from the decision and put in Scope Out.
- goal-scope LOW, D3 attributes by run: fixed; evidence scoped to the target batch's own recovery sessions.
- scenarios MEDIUM, D3 unparseable transcript blocks spawns: fixed; absent, truncated or unparseable all count as no evidence.
- scenarios LOW, D3 crash window: fixed; stated in the bound.
- premises LOW, Notes claim that `recoverSpawn` has the session id: fixed; note says it must be read from the shuttle run record.
- premises + scenarios MEDIUM, D5 nothing refuses tagged `lyx gate test` and the body says to use it; skip would be unrecorded: fixed; D5 now runs tagged steps through `lyx gate test --tags`, departing from the board's "left to verify" with the reason stated.
- pattern-fit LOW, D5 `go vet` module-wide risk: moot after the D5 change.
- goal-scope LOW, D5 additions beyond the board: moot after the D5 change.
- goal-scope MEDIUM, D6 #507 claimed closed and the expand premise is weak: fixed; Goal says #507 in part, Scope Out names what stays, the rationale now rests on the downstream glyph consumers, not on `quarry expand`.
- goal-scope MEDIUM, D6 "how the target was looked up": fixed; detail names the resolved key, the searched tree, the status and the unit answer.
- premises LOW, test-file members not evidenced: fixed; the claim now cites quarry's walker, not the two lookups.
- scenarios LOW, D6 per-configuration file reopening: fixed; constraints parsed once per file, the `MatchFile` note removed.
- pattern-fit LOW, D1 splitter vs `internal/shell`: fixed; D1 says why it stays in shuttleengine.
- board-fit LOW, quarry bump and `weaver`: fixed; both in Open risks.
- goal-scope LOW, #522 reinterpretation: fixed; added to Open risks.
- leanness MEDIUM, rejected alternatives in Decisions: fixed; comparisons cut, needed requirements kept.
- leanness MEDIUM, Goal facts duplicated in rationales and Notes: fixed; rationales shortened, Notes trimmed.
- leanness MEDIUM, D6 consumer list: fixed; replaced by "no other consumer changes".
- leanness LOW, Goal sentence, Scope Out reasons, D1 existing-behaviour bullet, compliance-only Constraints, process bullet in assumptions: fixed.
- leanness LOW, D4 refusal classification: fixed with the D4 change above.

## Rejected alternatives

- Item 1, stop the gate shell from lyx and record its result discarded: the provider reports no pid and lyx kills no shell, so lyx cannot stop it.
- Item 1, a contains-match on `lyx gate test` anywhere in the label: also matches `grep` and `echo` of the text.
- Item 1, a second field `AwaitedShellCommands` beside `AwaitedShellPrefixes`: matching the existing prefixes at each command start serves both callers with one field, and the recovery prefix only gains from it.
- Item 1, holding every outstanding shell on Webster's gated turn ends: a stray long-running shell would hold Master to its deadline, which the count-at-once rule exists to avoid.
- Item 2, a verb that spawns the verify fixer: forks are the Agent tool's inside Master's session, and Go has no handle on them.
- Item 2, a PreToolUse deny on Master's Edit and Write outside its two contract files: forks inherit the session's settings, so the deny reaches the fixer fork that must edit; separating them reworks the fork-mode settings beyond this task.
- Item 3, a second recovery from a clean tree with the first attempt's diff as reference: lyx would discard uncommitted work, which goes through the fabric destruction chokepoint, and it throws away work the recovery can check and keep.
- Item 3, reading the prior record's `ShuttleRunDir` instead of a session list: each spawn replaces the record, and amendment respawns can exceed two, so only one prior would be seen.
- Item 3, adding recovery writes to `ownTrackedPaths` so `reset` adopts them: widens what reset may discard, which the board does not ask for (lens review).
- Item 3, adopting every batch's recovery sessions: one batch's recovery could take another batch's leftovers as its own.
- Item 5, skipping `tmux`/`llm` steps and leaving them to verify, with a `go vet -tags` compile check: nothing refuses the tagged gate, the shared body says to run it, and the skip would go unrecorded.
- Item 6, driving `go/build.Context.MatchFile` per configuration: reopens each file for every configuration.
- Item 1, leaving a completed gate shell's result unread: a failing gate could still end the run `done`.
- Item 2, keeping the instruction text as Go literals: `PATTERN-stencil-ownership`.
- Item 4, a `refusal-spec.md` row for the stuck/dead exit: a terminal digest is not a refusal.
- Item 4, leaving the exit 0 and only documenting it: the board asks for non-zero, and no reader depends on exit 0 for stuck or dead.
- Item 6, quarry honouring build constraints: external module, not editable here.
- Item 6, accepting disjoint ambiguity at Plan time for Edit, Delete and Uses: the downstream glyph consumers (scope guard, delete order, drift, rename) read ambiguous as unresolved, and the file-path spelling already passes them; the earlier reason, that `quarry expand` requires `found`, was weak, since only the operator verb does.
- Item 6, #507's first remedy (a Create handle reusing a name its own card deletes): the board's item 6 expectation is met through the done-check; the re-create rule is a separate plan-validator change the entry does not ask for.
- Item 6, changing `lyx quarry resolve`'s exit for disjoint ambiguity: it reports quarry's raw answer, which operators need.

## Review rounds

_No rounds yet._

## Question ledger

The prompt asks for private picks written before advisors are asked, and also says to write no output file before the lens review; the picks below were fixed before round 1 was sent and are recorded here at writing time.

- Q1 gate shell at a gated turn end. Private pick, before advisors: Webster awaits `lyx gate test`, and shuttle matches awaited prefixes at any command start. Alternatives seen: hold every shell for Webster; drop count-at-once for gated runs; stop the shell via gate-slot holder pids. Advisor `ly:twb-combo-1:discussion-advisor-1`: agreed on holding and on per-command matching, added the option of a separate command-words field. Settled: private pick, one field (Decision 1). Advisor-informed.
- Q2 Master self-fix. Private pick, before advisors: per-entry re-prompt text on `GateEntry` plus a findings-file header naming the fixer prompt. Alternatives seen: a spawning verb; a settings deny. Advisor: agreed, and added putting the ready-to-paste prompt line in the findings file. Settled: pick plus the prompt line (Decision 2). Advisor-informed.
- Q3 recovery attribution. Private pick, before advisors: record recovery session ids per batch and add their writes to the evidence. Alternative seen: clean tree with diff reference. Advisor: agreed, added the separate `RunWrites` field, the `contractevidence` exclusion, the missing-transcript tolerance and the `ownTrackedPaths` question. Settled: pick with those additions; first settled with `ownTrackedPaths` including recovery writes, then narrowed after the lens review to the target batch's own recovery sessions and the prompt grouping only (Decision 3). Advisor-informed.
- Q4 `recover-batch` exit. Private pick, before advisors: non-zero for stuck and dead, 0 for done and running. Alternative seen: non-zero for dead only. Advisor: agreed, confirmed the ladder reads fields only. Settled: pick (Decision 4). Advisor-informed.
- Q5 recovery gates. Private pick, before advisors: name the allowed gates in the recovery prefix; skip tagged steps and leave them to verify. Advisor: agreed, added the `go vet -tags` compile check and that the settings do not deny it. First settled: pick plus the compile check. Revised after the lens review: tagged steps run through `lyx gate test --tags`, since nothing refuses them and a skip would go unrecorded; this departs from the board's "left to verify" wording, stated in Decision 5 (Decision 5). Advisor-informed for the settings facts.
- Q6 build-constraint ambiguity. Private pick, before advisors: accept disjoint ambiguity at the `planglyph` resolve layer for every consumer. Alternatives seen: quarry upstream; done-check only. Advisor: differed, done-check only, because expand requires `found`. Settled: done-check only, plus clearer `create-not-done` and `glyph-ambiguous` details (Decision 6). Advisor-informed.
- #522 exit premise. Auto-pick: read as a terminal stuck or dead digest, since no failed path exits 0 at base.
- Q7 (lens-raised) what Master does when a held gate shell completes. Auto-pick, no advisor round: edit nothing; zero exit leaves the outcome, non-zero writes `stuck` quoting the command (Decision 1).
- Q8 (lens-raised) does a returned fork's shell ending start a Master turn. Open: unverified, left to the plan to confirm, recorded under Open risks.
- Free-tag cap of eight in the disjointness predicate. Auto-pick: bounds the enumeration and fails closed beyond it.
