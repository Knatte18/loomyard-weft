```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I tried to break each bound, guarantee and claim about existing code in the decision record.

**Claims attacked**
- D8 "What it can skip or let through: nothing": broken (F1).
- D8 "bounds it: the class definition of D7": broken, it is a prompt rule only (F5).
- D8 arbiterstop "refuses a `decision` escalation … as it does a `budget` one": held. `Eligibility` admits only `hard-cap` and `circling` (`internal/arbiterstop/arbiterstop.go:77`), so any other cause is refused with `HandBackWayForward`.
- D8 re-call returns the same Awaiting "whatever `lyx loom decision add` committed meanwhile": held as specified. The record names the `settledGeneration` bypass, which the generation and subject check at `bouncer.go:430` would otherwise take.
- D8 "one escalation per judged round" and accept-only at or past the hard cap: held. A continue at the hard cap is retired, and the next call re-escalates the round.
- D5 "Each fork has 6 tool calls …, which D6's reads fit": broken (F2).
- D2 in-flight claim "passes on its next gate evaluation: the row keeps its name and its gate list holds only `discussion`": held. The recipe is the embedded one that `loomrecipe` builds (`internal/loomrecipe/doc.go:1-20`).
- D10 "`lyx loom decision add` refuses a darn run until Darn-Write has validated and committed its record": held (`lyx loom decision add --help`).
- D10 bound "`writer_timeout_min` … a respawn after the timeout repeats the check": the bound holds, but the wake path breaks (F4).
- D11 "A status on `Merge-In` resumes on `Parent-Align`", with only the fixer report given a fallback: broken (F3).
- D11 `reportengine` keeps the archived `Merge-In` classification: held as a design. The current reads are `loomshed.NameMergeIn` in `timingspans.go:74,104` and `wakes.go:106`.
- Constraint "D2 and D11 change no row position": held.

**Findings**

F1
- Severity: MEDIUM. Class: design. Location: D8, first bullet and the "What it can skip or let through: nothing" line.
- Summary: the escalation runs before the judgment refusal, so a parent `accept` passes a judgment the refusal would have rejected, and the brief never shows the parent why.
- Counterexample: round n's review holds a BLOCKING `design` finding X, which the judge marks closed in the ledger and rules CONVERGED, plus a LOW `decision` finding Y.
  - Today `judgmentRefusal` refuses the round ("review finding %q is BLOCKING") and `rejudge` re-judges it (`internal/shedadapters/bouncer.go`, `judgmentRefusal` and `rejudge`).
  - Under D8 the round escalates first, and the brief lists only Y.
  - The parent records `accept`, and the round reaches `acceptRound` (`bouncer.go`, `acceptRound`), which settles, approves and commits.
  - X is never re-judged. D8's carry-over takes the `decision` findings and the ledger's open entries, so X, closed in the ledger, is not carried into `## Open risks`.
  - The dropped-key refusal (`judgmentRefusal`, round ≥ 2) is bypassed the same way.
- Evidence: `bouncer.go`, `judgmentRefusal` (the CONVERGED/BLOCKING and dropped-key checks), `settleUnlessRefused`, `acceptRound`.

F2
- Severity: MEDIUM. Class: design. Location: D5 ("which D6's reads fit") and D6 (`goal-scope` reads, and the "Both lenses" decision-note rule).
- Summary: the reads D6 prescribes do not fit the fork's 6-call cap.
- Counterexample for `board-fit`: it already spends all six calls (`lyx board list`, `lyx loom peers`, and up to four reads). The "Both lenses" rule makes a `Decision for <slug>:` note win over the entry, but `board-fit` has no call left to read a note's body.
- Counterexample for `goal-scope`: it reads "every `Decision for <slug>:` note" with no limit. Finding the notes takes `lyx board list`, which is not in its read list, and then one `lyx board get` per note. A task with six notes needs eight calls.
- Evidence: the cap is in `_board/_lyx/stencils/burler/burler-cluster-fan.md:17` ("at most 6 tool calls"). The reads are in `decision-record.md:74,79,85`.

F3
- Severity: MEDIUM. Class: design. Location: D11, the in-flight carry-across paragraph.
- Summary: Merge-In's fixer attempt record is stored under the row name, and `RenameProducers` rewrites only the status file. A carried-across run therefore loses its attempts, its over-cap halt and its `AlignFrom`.
- Counterexample:
  - A run halts over-cap at `Merge-In`, and the operator aligns it by hand.
  - After the upgrade, `Parent-Align` reads `FixerAttemptsPath(scratch, "Parent-Align")`. That path is absent, so `rec.Halt != fixerHaltOverCap` and `overCapResume` is false.
  - The row then spawns a fixer over the operator's work with a fresh `parent_align_fix_attempts` cap. It should have read that work as the alignment.
  - The stale `Merge-In` record is never removed.
- Evidence: `internal/landingshed/mergein.go:18`, `:208`, `:283`, `:306-316`; `internal/shedengine/doc.go:119-122` (`RenameProducers` rewrites only status fields).

F4
- Severity: LOW. Class: design. Location: D10, first bullet and the "What bounds it" line.
- Summary: the writer asks for the answer as a board note and ends its turn. A parent that follows the ask literally writes the note and never messages the writer.
- Counterexample: nothing wakes the held session, so the writer idles for the full `writer_timeout_min`, 180 minutes by default. The run then stops at Darn-Write, and only `lyx loom resume` brings the respawn that would read the note.
- D10 does not say the parent must also reply by message to wake the writer, or that the writer re-checks the board on any message.
- Evidence: `internal/darnengine/template.yaml:2`; `internal/loomshed/darnwrite.go:70-83` (the session outcome is returned as is, and the darn writer does not respawn itself); `internal/shuttleengine/doc.go:69-82` (a hold never extends the deadline).

F5
- Severity: LOW. Class: design. Location: D8 "What bounds it: the class definition of D7", and support-log "D7's tight class makes any-severity safe".
- Summary: the bound is reviewer-prompt text only, so a mis-tagged finding still stops the run.
- Counterexample: a reviewer files a writer's choice as a NIT `decision` finding. D8 fires "whatever the judge ruled", at any severity, so the judge cannot re-class or close it, and the run stops for the parent.
- Nothing mechanical enforces D7. The record should state that the bound is reviewer discipline, or let the judge's ledger narrow it.
- Evidence: `decision-record.md:97-98,107,117`.

**Tool calls**
1. A grep of `internal/` and `internal/loomrecipe/doc.go` for `loom-recipe.yaml`, to check whether the recipe is embedded or seeded for D2's in-flight claim; the manifest did not cover it.
2. `internal/loomshed/darnwrite.go:70-130`, to see how a writer timeout returns and whether the writer respawns itself, for D10's bound; the manifest held only the file's head.
3. A grep of `internal/landingshed` for on-disk paths keyed by `mergeInName`, for the D11 carry-across; the manifest covered only `haltSpent`.
4. `internal/landingshed/mergein.go:296-320` and `fixerattempts.go`, to read what the attempt record holds (halt, `AlignFrom`), which the counterexample in F3 depends on.
