# Support log — machine-switch

## Interview

Autonomous run: no operator answered in the pane; picks are auto-picks in the question ledger, bar the two parent decisions that arrived by creel.

### Turn 1 — what the code does today

Read the board entry, its done dependency `dobby-orch-pair-start`, notes `reed-status-buttons` and `run-pins-its-version`, `lym loom peers` (no peer touches adopt, beam or the pause record), the help of `lym adopt`, `lym fabric pull`, `lym beam start`, `lym tender pause` and `lym tender resume`, `internal/adoptcli/doc.go`, the head of `internal/adoptcli/adopt.go`, `fabricengine.ReconcileOrigin`'s targets, `fabricengine.PullHub`, dobby's pass planning and owner-pause lift, and the pause record's doc and location.
Findings: adopt already reconciles Bolt and every local pair against origin, settles local commits, adds pairs, writes owners and runs the beam steps; the gaps were that it resumes every moved run, that dobby, once up, passes any running run no pause covers, and that nothing reports the runs' states.
The operator's hand sequence on 2026-10-31 (`lym fabric pull`, `lym s`, `lym adopt`) needs neither of the first two, since adopt reconciles origin and runs the beam steps.

### Turn 2 — first design

Picked `lym adopt` as the verb, its reconcile as the sync, a pause entry with a new caller as the hold, a release verb `lym beam go`, a `runs` list, and no hold for released tasks with no run.
Wrote the three files; the gate passed; spawned the six lens forks.

### Turn 3 — parent decision, creel #211 from `ly:orch`

The runs wait only for the adopt check and start by themselves when it is green; no hand release, no release verb, no reed button; the status lines stay with the states landed, running here, waiting for the adopt check, held by a failed check and owned elsewhere.
Dropped `lym beam go`; the check's green end lifts the holds and starts the runs (D3, D5).

### Turn 4 — parent decision, creel #212 from `ly:orch`

A failed check files a darn fix task in the beam, released at once and the only run that starts; the darn diagnoses, fixes a code failure with its perch, or stops on a machine failure telling the operator what to do; a landed fix reruns the check, up to a configured number of tries; the mechanism is one general piece note `parent-merge-task` reuses.
Read note `parent-merge-task`, `lym board upsert`'s fields, `lym board profiles`, dobby's `notify_on_land` memory, the darn writer stencil's stop marker and the `beam` config module.
Since the fix lands in the beam's own branch and today's check tests only the prime, the check became one per adopted subject (D5); the fix loop is D6, the general piece is front tasks (D8), and the machine-failure stop is the darn writer's stop on a condition its task names (D9).

### Turn 5 — lens review

The six forks reviewed the Turn 2 record; their findings were judged against the record as rewritten after Turns 3 and 4.
- board-scope, D6 released tasks start without a go-ahead (decision class): fixed, released tasks under an adopted subject are held too (D4); no parent question needed, since the parent's rule answers it.
- leanness, Open risks entry is a settled fact: fixed, moved into D4 as the exit-78 backstop; Open risks reads none.
- leanness, D4 shared-function sentence repeats the Notes: fixed, `lym beam go` is gone.
- leanness, D3 restates D4's resumes: fixed, one line pointing at D5.
- leanness, D1 rationale carries incident narrative: fixed, moved to Turn 1.
- leanness, Scope Out repeats D6: fixed, the bullet is gone.
- leanness, D2's last sentence adds nothing: fixed, cut.
- leanness, `PATTERN-cli-cobra` constraint is a restatement: fixed, cut; `PATTERN-spawn-observability` kept, since it now covers the new `on_land` spawn.
- leanness, `lym beam go` with its dobby down: fixed, the green end brings the subject's pair up through `orchcli.BeamUp` (D5).
- existing-paths, the lift and `StartFirstPass` race: fixed, a start refused for a held driver lock is no error (D5).
- existing-paths, adopt's standing-pause text names `lym tender resume`: fixed, it names the check and the re-run of `lym adopt` (D7).
- existing-paths, `lym tender resume` skips the Finalize-child resume: fixed, stated as a bound in D4.
- existing-paths, `OwnerEntryExempt` push child loses its exemption: fixed, stated as a bound in D4.
- existing-paths, the network gate holds a held pair's pushes: fixed, stated in D4.
- bound-breaker, dobby's owner-pause recheck owes a lift pass under the hold: fixed, dobby owes none for a `HoldCaller`-covered run (D4) and an acceptance criterion pins it.
- bound-breaker, a covered run still gets continue passes: fixed, dobby gives a held run no pass of any kind (D4).
- bound-breaker, `landed` read from a run record the reconcile tore down: fixed, `landed` reads the board entry done after the reconcile (D7).
- bound-breaker, `--keep-local` over a live pair runs without a go-ahead: fixed, stated as a bound in D4.
- bound-breaker, the double start after a release: fixed with the race finding above.
- interactions, the re-run criterion cannot report `landed` again: fixed, the criterion covers the runs not done.
- interactions, a nested beam's pawl run released by two verbs: superseded, no release verb remains; the pawl run is held only when it reads owned here after the owner write, and its subject's check lifts it.
- interactions, held and owned elsewhere at once: fixed, such a run is not held, and D7 states the precedence.
- interactions, owner-entry conversion and dobby's recheck: fixed with the bound-breaker finding above.
- interactions, the double start after a release: fixed with the race finding above.
- walkthrough, the owner a hold is decided by read from a stale local Bolt (BLOCKING): fixed, it is read from the Bolt the reconcile left (D4) and an acceptance criterion pins the stale case.
- walkthrough, continue passes under the hold: fixed with the bound-breaker finding above.
- walkthrough, `landed` detection: fixed with the bound-breaker finding above.
- walkthrough, D2's claim that adopt answers a prime code-branch stop: fixed, such a stop is now a warning naming `lym fabric pull` (D2).
- walkthrough, the Finalize-halted child after `lym tender resume`: fixed, stated as a bound in D4.

Every fork returned a report.
The record changed after the lens pass by the parent's two decisions, which no fork reviewed.

## Rejected alternatives

- A new verb wrapping `lym fabric pull --hub`, `lym adopt` and `lym beam start`: adopt already runs the reconcile and the beam steps, so the wrapper would only add a second pull and duplicate adopt's refusals.
- Running `lym fabric pull --hub` inside adopt: the reconcile already fast-forwards every local pair and Bolt; the cases it leaves are reported as warnings naming `lym fabric pull`.
- Keeping the owner pause as the hold: dobby lifts its own owner pause once the owner reads this machine, and a moved run often has no owner entry here, since the pause record is machine-local.
- A batten pause (`lym batten pause`) as the hold: it acts at a producer boundary of a running pass, not before a pass starts.
- A beam-scoped pause entry as the hold: a slug entry covers that slug only, so it would not hold the beam's member runs.
- A release verb `lym beam go`: ruled out by the parent (creel #211).
- One adopt check in the prime for every subject: a fix lands in the beam's own branch, which a check of the prime's tree never tests.
- Storing the land action as a command on the board entry: any board edit could then make dobby run an arbitrary command; a closed action table keeps it bounded.
- Dobby's `notify_on_land` memory as the land-action record: it never remembers a run first seen past its Land row (`internal/dobbyengine/landnotice.go`).
- A front task that also pauses the worktree's runs already going: adopt holds its runs itself, and stopping running members is not asked by either user of the piece.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): a new verb or reshape `lym adopt`? Picked reshaping `lym adopt`; it holds every step bar the hold, the fix loop and the report.
- Q2 (auto-pick): does the verb add a `lym fabric pull`? Picked no; the origin reconcile covers Bolt and every local pair (fact from `internal/fabricengine/originreconcile.go` at `2de97ce7`), and an unmoved code branch is a warning.
- Q3 (auto-pick): what is the hold? Picked a slug-scoped pause entry with a new fixed caller, written before the owner write.
- Q4 (auto-pick): which runs are held? Picked moved runs owned elsewhere per the reconciled Bolt that will be owned here after the write, bar a run live here; plus released tasks not started under the subject, bar front tasks.
- Q5 (resolved by parent, creel #211): how is the hold released? By the subject's green adopt check only; no release verb or button.
- Q6 (resolved by parent, creel #212): what happens on a failed check? A darn fix task in the subject, retried up to a configured count, through a general piece.
- Q7 (auto-pick): one check or one per subject? Picked one per subject, in its worktree.
- Q8 (auto-pick): where does the try limit live? Picked `adopt_fix_tries` in `beam.yaml`, default 3.
- Q9 (auto-pick): what is the general piece? Picked front tasks: filing, a start hold in the worktree and a land action from a closed table.
- Q10 (auto-pick): how does the darn stop on a machine failure? Picked a stop on a condition the task body names, through the existing stop marker.
- Q11 (auto-pick): which runs count as landed? Picked runs whose board entry is absent or reads done after the reconcile among those this machine held not done before it, since Land prunes every done entry.
- Q12 (auto-pick): a run under an operator's own pause? Picked state `paused`, naming the caller.
