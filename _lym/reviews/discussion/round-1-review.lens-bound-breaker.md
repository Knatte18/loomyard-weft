```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked every bound, guard, ordering promise and claim about existing code in the decision record.

**Claims attacked**

- D1, records side before code side, `PushPairAnchored` for every caller: held as written.
- D1, the admission means a later pause never holds the push's first attempt: held. Side effect: `Admit` lets every network call in the process through during the lock wait (hold.go:455-458), up to `StatusPushLockWait`. Not filed.
- D1, a push owed because of a held teardown lock reads into `PushOwedOffline`: **broken** (wording, F6).
- D1, retiring the respawn, with the next synchronous push carrying the branch: held as written.
- D2, the takeover entry acts as a compare-and-swap on a shared records branch, with replay only when no entry is found: held. Each rejection reads origin first, and the adopter's retry covers a race.
- D2, "a given-up tip never lands": **broken** (F2).
- D2, today Finalize marks the board `done` before the parent push: held (internal/landingshed/finalize.go:708-711).
- D2, a Finalize cut between the local merge and the parent push pushes at its next pass: **broken under a hold** (F7).
- D3, "one identity", and a registration clash never overwrites another machine's entry: **broken** (F3).
- D4, a catch-up runs before any act rule answer after an absence, and an offline catch-up acts on nothing: held.
- D5, the act rule already lets a landing past its parent-side merge act under a hold: held (internal/runfsm/act.go:91-93), but only when origin holds the merge (F7).
- D5, a `hold` entry is lifted only by the listed verbs: held. `lym batten cancel` passes every pause (cancelcmd.go:202 `SetGateExemptAll`), which D5 permits. Hub-wide `Resume` picks every live entry (resume.go:140), which D5 changes explicitly.
- D5, a run holding its parent lock is exempt from slug entries: held, and stated (hold.go:305).
- D5, another machine goes on for at most the dobby's pull interval: held.
- D6, "only the operator's own shell runs" `--take-hold`: **broken** (F1).
- D6, a re-run goes on from tracked state, and the fix-task tries stay within `adopt_fix_tries`: **broken** for a plain re-run over this machine's own `adopt-fix` hold (F4).
- D6, two machines adopting at once end with one owner: held. The rejected push reads Bolt again.
- D7, "A pair kept by a cancel before the upgrade has no batten record": **broken** (F5).
- D7, a kept pair's batten record keeps it out of the orphans: held for cleanup, since cleanup keeps a cancelled record while its pair stands (cleanup.go:74).
- D9, a hub entry already standing covers the stop and stays, so the stop lifts none: held.
- D9, "as after a crash" for a run cut mid-step: held as a claim of equivalence.
- D10, a prime with no recorded owner is skipped and only owned beams start: held. `BeamUp` calls `EnsureLive` only for its own worktree (orchcli/start.go:388).

**Findings**

F1 — MEDIUM, design. D6, "only the operator's own shell runs it", and the `PATTERN-shell-guard-table` constraint.
The bound rests on a shell-guard row, but the table states every row is "a guardrail over static text and not a barrier" (internal/shellguard/doc.go:23).
A heredoc that writes a script and then runs it passes the row (doc.go:19-20), and so does a command behind a command substitution inside double quotes (doc.go:21).
Counterexample: an orch agent writes `lym adopt --take-hold &lt;subjects&gt;` into a script and runs it. That moves another machine's live hold and its adopt to this machine.
The decision has to state the real bound: the old adopter stops at its next re-read, and no run goes on before a green check. It must not claim the operator alone can run the flag.

F2 — MEDIUM, design. D2, "A machine owing a tip pushes it only while the run's last transition on origin names it, so a given-up tip never lands."
The check and the code push are separate refs with no lease between them.
Counterexample: the owing machine reads origin, and the last transition names tip T. The owner then gives T up and pushes a records commit that returns the run to its previous transition. The owing machine then pushes T to the code branch, which no takeover entry or records rejection guards.
T lands on origin's code branch while no transition names it. The owner's next code push is then non-fast-forward.
Evidence: D2's bullets in decision-record.md:88-92, and `PushPairAnchored` pushing each side on its own ref (decision-record.md:34).

F3 — MEDIUM, design. D3, "gives each machine one identity", and the D3 bound.
The only refusal is for a former name equal to another machine's current name.
Counterexample A: machine A renames itself from `osl46` to `osl-1046`, and a new machine C is then named `osl46`. C's current name equals A's former name. An owner recorded as `osl46` then reads as owned here on both A (through the registry) and C (by direct match), so two machines each act as owner.
Counterexample B: two machines with the same current name each register into one entry, and one adds its former names to the other's entry.
Today `boardengine.OwnedElsewhere` is a plain string comparison (internal/boardengine/owners.go:175-177), so the registry is the only guard, and D3 states no refusal for either case.

F4 — MEDIUM, design. D6 step 1 against step 8 and the try limit.
A plain `lym adopt &lt;subjects&gt;` re-run on the machine whose own `adopt-fix` hold stands passes step 1, because the hold is not another machine's. Step 1 then writes an `adopt` hold.
Only `--take-hold` is said to keep the kind, the fix task and the try counts (decision-record.md:208). A plain re-run that rewrites the hold to `adopt` drops the counts.
The next failing check then files tries from zero, which breaks "failing files the next task within `adopt_fix_tries`".
D5's upgrade notice and D10 both point the operator at exactly this re-run (decision-record.md:172 and :310).

F5 — BLOCKING, design (false premise about existing state). D7, "A pair kept by a cancel before the upgrade has no batten record."
Today a cancelled run keeps its batten record while its pair stands. See internal/battencli/cleanup.go:74 ("A cancelled run keeps its record while its pair stands on either"), and cleanup.go:558-573, which refuses to remove such a record and names `lym fabric remove`.
So a pre-upgrade kept cancel has a record in state cancelled with no tracked cancel intent saying keep. The upgrade rule skips it because a record exists. D7's teardown then reads FSM `cancelled`, finds no keep, and tears down a pair the operator kept.
The upgrade rule must also cover a legacy cancelled record that carries no cancel intent.

F6 — LOW, consistency. D1, "A wait past `fabricengine.StatusPushLockWait` turns the push owed", read into `PushOwedOffline`.
A push owed because a teardown lock is held is not offline. Yet the act rule's refusal for that input says "a push is owed to origin, which is unreachable" (internal/runfsm/act.go:115), and the field doc says "because origin is unreachable" (act.go:25).
D1 does not say whether a push owed on the lock sets the same input and wording.

F7 — LOW, design. D2's same-machine resume of a Finalize cut after the local merge, under a D5 hold.
The act-rule exemption needs `PastParentMerge`, which reads origin's parent branch (internal/runfsm/act.go:29, 91). Under D2's new order, a Finalize cut between the local merge and the parent push has the merge only locally. Under a hold, the act rule therefore answers `wait` (act.go:93-94).
The local parent branch stays ahead of origin for the whole hold, which can span an `adopt-fix` cycle. D5's line "the act rule already lets a landing past its parent-side merge act under a hold" does not reach this state.

**Tool calls**

1. A grep of battencli cleanup, cancelcmd, landingshed finalize, the beam-start `EnsureLive` and `OwnedElsewhere`, to test the claims in D7, D2, D10 and D3 against code the manifest did not hold.
2. A read of cleanup.go:60-90 and cancelcmd.go:230-260, plus a grep for keep, to confirm what a kept cancel leaves today. This is needed for D7's upgrade premise.
