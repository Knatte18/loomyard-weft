# Support log — Webster stability: gate shells, Master self-fix, recovery, Create checks

## Interview

The session ran autonomously with no advisors, so every turn below is the chair's own batch of questions answered by exploration and self-pick.

### Turn 1 — scope and the board

Read the board entry and the seven linked issues.
The sibling `dbench-*` entries carry the same body and were not read as design context.
Nearby planned work: `proc-child-lifetime` (child-process lifetime in `internal/proc`) and `lyx-halt` touch process stopping; D2 deliberately stops no process itself, so it neither duplicates nor contradicts them.
Issue #511's third ask (`goto` reviving a dead batch) and #507's first option (same-card delete-and-recreate) were set out of scope.

### Turn 2 — gate shells (#505, #509, #517)

Question: hold the turn end on a running gate shell, or stop it and record the discard?
Found: shuttle already awaits shells by caller-supplied label prefix, bounded by the run `Timeout`; webster awaits only `lyx webster recover-batch`; every shell outstanding at run end is already recorded and written to a friction note.
Pick: both, split as D1 (hold, shuttle-wide, on a `lyx gate` label) and D2 (discard mark on the record).
A shuttle-wide constant was preferred over per-caller lists because every lyx-spawned role runs the same gate verb.

### Turn 3 — Master's self-fix

Question: restate the rule where Master reads the failure, or give Master a verb?
Found: the Go-rendered gate message says "read it, fix every finding, and end your turn", contradicting the stencil section it precedes.
A verb cannot spawn an in-session fork.
Pick: D3, a per-entry re-prompt text that points at the stencil section.

### Turn 4 — recovery attribution and exit code (#522)

Question: attribute by batch, or restart from a clean tree with the diff as reference?
Found: the write history audits Master's session and each batch's begin-time session; a respawned recovery replaces the batch record, so the first recovery's session is lost.
The success path of `recover-batch` returns an ok envelope for every terminal digest.
Pick: D4 (carry recovery sessions on the batch record) and D5 (non-zero exit on a non-done terminal digest).

### Turn 5 — allowed gates in the recovery prompt

Found: the fabric ban and the gate-verb rule both live in the shared implementer body, which the recovery prefix composes.
Pick: D6, one statement in the body; the board's expectation that `tmux`-tagged steps are left to verify is followed under assumption A3.

### Turn 6 — Create checks and ambiguity (#511, #500, #507)

Question: does quarry resolve unexported or test-file members?
Verified by running `lyx quarry resolve` on an unexported function, a same-package `_test.go` helper and an external-test-package helper: all `found`.
A per-GOOS function resolved `ambiguous` with both files as candidates, each carrying its path.
Pick: D7 (no resolver change; refusal names the lookup) and D8 (constraint-disjoint ambiguity reads as `multipart`, in `planglyph`).
The Plan-time flagging the issues ask for is already present for a truly ambiguous Edit or Delete, and D8 keeps it.

## Rejected alternatives

- Per-caller awaited prefix for gate shells (webster adds `lyx gate` to its own list): every lyx role runs the same verb, so the rule would be copied per spawner; the shuttle-wide constant holds it once.
- Stop the shell by PID at run end: shuttle has no PID for a provider-run shell, and the strand's removal already ends it; the record is what the issues need.
- Expire a gate shell after `background_shell_wait_min`: that is the behaviour the issues report as the fault.
- A `lyx webster` verb that spawns the verify fixer: an in-session fork needs the session's own Agent tool; Go cannot spawn one.
- Restart the second recovery from a clean tree with the first attempt's diff as reference: discards finished card work and makes the recovery redo it from a diff.
- Patch quarry to honour build constraints: a separate module with a syntactic contract; lyx's policy layer is where the interpretation belongs, and the change would not land in this commit.
- Let a Create handle reuse a name the same card deletes (#507): unnecessary once per-GOOS symbols are editable by glyph.
- Accept any `ambiguous` answer in DoneChecks: fails open on a real duplicate declaration.
- Resolve ambiguity by picking the candidate for the host GOOS: a Delete would then pass with the other GOOS's body still present.

## Review rounds

_No rounds yet._

## Question ledger

| # | Question | State | Resolution |
|---|---|---|---|
| Q1 | Hold the turn end on a gate shell, or stop and record? | resolved, auto-pick | Both: D1 and D2. |
| Q2 | Where does the gate-shell prefix live? | resolved, auto-pick | A shuttle-wide constant, unioned with the spec's list (D1). |
| Q3 | How does Master learn not to fix a verify failure itself? | resolved, auto-pick | A per-entry re-prompt text naming the stencil section (D3). |
| Q4 | Attribution by batch or clean restart? | resolved, auto-pick | By batch: recovery sessions carried on the batch record (D4). |
| Q5 | Exit code on a terminal non-done recovery? | resolved, auto-pick | Non-zero with the digest fields and a spec row (D5). |
| Q6 | Where is the allowed-gates statement? | resolved, auto-pick | Once, in the shared implementer body (D6). |
| Q7 | Are `tmux`-tagged steps run by the strand? | resolved, auto-pick | No, left to verify, per the board; reason confirmed at Plan time (A3). |
| Q8 | Does quarry resolve unexported or test-file members? | resolved by exploration | Yes, verified; no resolver change (D7). |
| Q9 | Where does the build-constraint reading live? | resolved, auto-pick | `internal/planglyph`, applied by every policy there (D8). |
| Q10 | Is `goto` reviving a dead batch in scope? | resolved, auto-pick | No; not in the board entry. |
| Q11 | New PATTERN entry? | resolved, auto-pick | None; module rules go in each `doc.go` (D9). |
| Q12 | Which signal's label does D1 match? | open | Assumption A1; Plan confirms what each signal puts in `Label`. |
| Q13 | Is a terminal `failed` digest reachable from the success path? | open | Assumption A5; D5 covers it either way. |
