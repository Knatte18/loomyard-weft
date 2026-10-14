# Support log — intake-bugs-1012

## Interview

Autonomous mode: no operator answers; every pick is the writer's and is in the question ledger.

Turn 1 — board read.
The entry bundles four bugs, not the three its brief names: external-test gate paths (#571, #577, #580, #587), the lyxtemp sweep flake (#584), the Merge-In `status-missing` halt (#581) and the plan-time CLI-tree conflict (#588).
`twill-plan-followups` mentions a wrong-parent-directory miss in plan checks; it is a different miss (a check against the wrong parent directory) and stays its own note.

Turn 2 — gate paths.
Card and batch gates are Go-derived in `websterengine/cardgate.go` from `planparser.CardTargetDirs`, not written by the plan author.
Quarry's `docs/glyph.md` names an external test package's unit `<dir>_test`, and `UnitPath` returns it unchanged, so a member glyph on that unit becomes the gate argument `./<dir>_test`.
The plan-gate-fixes commit `7b17ddfac` itself reports two such steps.
`planparser/vocabname.go` already trims `_test` from a unit for its owner check, which is precedent.
The glyph chokepoint forbids loomyard's own glyph-to-path conversion, so the mapping is recorded as a named change to that entry (D2).
Author-written Verify lines remain a second route, covered by a new plan check (D3), and `lyx gate test` names the bad argument (D4).

Turn 3 — lyxtemp flake.
Issue #584 was created at 11:33Z; `78e31aa06` with the bounded retry landed at 11:27Z.
`lyx gate test ./internal/lyxtemp -- -run TestSweep -count=300` and `-- -race -run TestSweep -count=1000` both pass on current code.
Decision D6: no code change.

Turn 4 — Merge-In halt.
`status-missing` is set by `shedverbs`' loop when the status read fails before any child runs; `readStatusAt` folds a decode error into "missing".
`routed_to` landed in `314816cde`, which a reroute edge writes into history; `e5b56fe91` later removed the strict decode that an older-binary loop applied.
The halt's empty trace and zero history follow from no child having run.
Decision D5: split the cause, log it, point the trace at the loop log.

Turn 5 — CLI-tree conflict.
`PATTERN-cli-cobra.md` already says a non-invocable group carries no audience, but not what makes a runnable group non-invocable (`clihelp.Invocable`).
Decision D7: write that rule into the background and add a Plan-Review rubric check.

Turn 6 — lens review; all six forks returned.

- premises: `internal/batcher` also reads `CardTargetDirs` — fixed (D1 names it, acceptance row).
- premises: `vocabname.go` trims a handle's package half, not a `UnitPath` result — fixed (helper input is any Go unit string).
- premises: "no repository in the hub" checked only this one — fixed (reworded).
- premises: refusal-spec table exists — no finding.
- goal-scope: D6's "ran without the fix" premise is inferred — fixed (premise cut from the record, timing kept here, risk kept).
- goal-scope: `docs/overview.md` audience paragraph not covered — fixed (D7 points it at the background).
- goal-scope: D3 checks `go` commands beyond the ask — fixed (rationale added).
- goal-scope: D4 makes an optional ask mandatory as a refusal — rejected: kept by Q1; the name-and-run alternative is now in Rejected alternatives.
- goal-scope: Goal does not say where the fourth bug comes from — fixed.
- board-fit: `convergence-plan` classes halt reasons by leading clause — fixed (constraint).
- board-fit: `depends_on` empty though D5 leans on `e5b56fe91` — fixed (D5 says it is in the branch base).
- pattern-fit: PATTERN.md one-liner left unchanged — fixed (D2 amends it).
- pattern-fit: D4 base directory must come from the seam cwd — fixed.
- pattern-fit: D7 rubric line restates the rule — fixed (pointer only).
- pattern-fit: refusal row class unstated — fixed.
- pattern-fit and scenarios (merged): `WriteFailedStop` cannot write over an unreadable status — fixed (D5 bound, named departure from `PATTERN-failure-stop-visible`, specific way forward, acceptance row).
- scenarios: Move destinations give false findings — fixed (bound and acceptance row).
- scenarios: dispatch re-validation and earlier deletes — fixed (dispatch drops done-card findings already; deletes stated as a bound).
- scenarios: `.go` file arguments — fixed (exempt in D3 and D4).
- scenarios: batcher estimate — merged with the premises finding.
- scenarios: halt-note history stays empty — fixed (stated in D5's bound); using the loop's last good status was rejected, see Rejected alternatives.
- scenarios: atomic-write premise scope — fixed (names `internal/state` as the only writer).
- scenarios: D4 on inherited and outside-hub paths — fixed (check precedes every slot path).
- leanness: D5 and D6 premises carry deliberation — fixed (trimmed; the narrative stays in turns 3 and 4).
- leanness: rejected alternative in Auto-mode assumptions — fixed.
- leanness: no-obligation Constraints bullets — fixed (removed).
- leanness: duplicates between Open risks and D4's bound, and between Scope Out and D6 — fixed.
- leanness: D1 bound rationale repeated in assumptions — fixed.

## Rejected alternatives

- Fixing the external-test mapping in quarry (a `UnitDir` or a changed `UnitPath`): another repository and release, outside this worktree; quarry's documented behaviour is correct for its own purposes.
- A disk-aware D1 that strips `_test` only when `<dir>_test` does not exist: changes `CardTargetDirs`' disk-free contract and every caller's signature for a case no repository in the hub has.
- Dropping a non-existent glyph-derived directory from the gate scope instead of mapping it: the external test package's tests would then never run in the card gate.
- Refusing external-test glyphs as plan targets: quarry resolves them, and authors legitimately target external test members.
- `lyx gate test` mapping `<dir>_test` to `<dir>` silently: hides the plan defect; naming it lets the author fix the line.
- A sweep code change or a test rewrite for #584: no failing case to verify against.
- `lyx gate test` naming the missing directory but still running the other packages: needs output post-processing of go test's stream, and a run that cannot pass is better stopped before the slot wait and prebuild.
- Feeding the halt note's history from the loop's last good status: widens the `AfterInterrupt` hook for a field the cause and the log path already make unnecessary.
- Writing a failed stop over an unreadable status file: destroys the evidence the halt points at.
- A re-step on `status-unreadable`: state writes are atomic, so the error is not a transient half-write.
- A mechanical CLI-shape plan check: card prose carries no `Use`, flags or annotations to check.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick, resolved): fix the derived gate's mapping, the plan check, the gate refusal, or all three? All three: the mapping fixes every reported case, the check covers author-written Verify lines, and the refusal is what the issues ask for at run time.
- Q2 (auto-pick, resolved): where does the external-test mapping live? One helper in `internal/planparser`, named in `PATTERN-glyph-conversion-chokepoint`'s background (D1, D2).
- Q3 (auto-pick, resolved): unconditional or disk-aware mapping? Unconditional, bounded as D1 states.
- Q4 (auto-pick, resolved): does the lyxtemp flake still reproduce? No; no code change (D6).
- Q5 (auto-pick, resolved): what caused the Merge-In halt? Most likely the strict status decode an older-binary loop applied after `routed_to` landed, since fixed by `e5b56fe91`; the task makes the stop diagnosable (D5).
- Q6 (auto-pick, resolved): which trace file does a no-child stop record? The loop's own log, as the `loop-exited` stop does.
- Q7 (auto-pick, resolved): plan-time CLI check or docs? Docs in `PATTERN-cli-cobra.md` plus a Plan-Review rubric check (D7).
- Q8 (auto-pick, resolved): the brief says three bugs and the body has four; all four are in scope, since the body is the entry's content.
