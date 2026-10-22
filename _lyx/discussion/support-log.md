# Support log — finalize-dirty-names-paths

## Interview

### Turn 1 — exploration (autonomous)

Read the board entry (issue #776) and the board list; no other entry touches Finalize's dirty guard.
Traced the refusal: `fabricengine: merge preconditions failed: worktree dirty` comes only from `MergeGuardError`, and in Finalize the bare reason comes from steps 6-7 after the parent-side `Merge`.
The task-side merge-in runs behind `gate.clean`, which has its own reason, so the bare text is always the parent side.
`pairDirtyReason` reads only the code checkout; the code checkout's `_lyx` is a link into the records checkout, so run records cannot trip it, and step 2b commits the parent's run records anyway.
The halt note's `parent fabric status:` section is filled by `parent.DirtyPaths()`, which appends both checkouts' paths: that is where the run record showed up.
A blocked stop goes to the tackler; a resume verdict passes the recipe's `GotoGuard`, and a refused goto hands back.
So in the incident the refusal was the code checkout's `docs/overview.md` both times; the `_lyx/shed/<slug>/status.json` came from the halt note's own both-checkouts read.

### Turn 2 — decisions (autonomous)

Settled D1-D4 as auto-picks; see the question ledger.

### Turn 3 — lens review

- bound-breaker, BLOCKING: the tackler may resume at the routed-from row (PR-Gate), which routes back into Finalize. Fixed: D4 also refuses a row whose `OnDone` routes to Finalize; acceptance criteria widened.
- existing-paths, MEDIUM: same routed-from route. Merged with the above.
- bound-breaker, MEDIUM: "never blocks a goto that could land" is false, since dirt can be cleaned during Finalize's earlier steps. Fixed: D4's bound now says advisory, decided at goto time.
- bound-breaker, MEDIUM: a sibling run's in-flight parent merge shows as dirt, and the way forward would say discard it. Fixed: D4 admits when a fabric merge record exists on the parent.
- interactions, MEDIUM; bound-breaker, LOW: the guard's decision and the path list could be two `git status` reads. Fixed: D2 decides on the list itself; acceptance criterion added.
- interactions, MEDIUM: reusing `halt.go`'s route would bring back `DirtyPaths`' records-checkout read. Fixed: D4 and the note name the code-checkout-only read.
- walkthrough, MEDIUM: Finalize's parent handle carries no checkout path. Fixed: the error carries the checkout path too.
- walkthrough, MEDIUM; goal-scope, LOW; existing-paths, LOW; interactions, LOW: `lyx loom resume` on a blocked Finalize goes through a tackler, and a refused goto from another row then resumes the wrong row. Fixed: the way forward is `lyx loom goto --to Finalize` then `lyx loom resume`, as the settled-segment refusal already does; the assumption is restated (round 1 moved it into D2 as a settled fact).
- leanness, MEDIUM: the three-site requirement sat in the optional notes. Fixed: moved into D2.
- goal-scope, MEDIUM; interactions, LOW; bound-breaker, LOW: D3 drops the paths for task-side `MergeIn` dirty halts too. Fixed: Scope's Out says so.
  The old halt-note section listed the parent's paths for those halts, the wrong tree for a task-side refusal, so D3 loses nothing correct there.
- goal-scope, LOW: other surfaces of the bare error unstated. Fixed in Scope's Out.
- goal-scope, LOW: "halts once" against the tackler spawn. Fixed: Goal reworded.
- leanness, LOW: D1's incident forensics. Fixed: moved to Turn 1.
- leanness, LOW: Auto-mode assumption repeating D1. Fixed: cut.
- leanness, LOW: Constraint restating test PATTERN entries every task inherits. Fixed: cut.
- leanness, LOW: D4's routes paragraph mixes existing behaviour in. Fixed: shortened.
- existing-paths, LOW: no rendering for an empty list. Rejected: D2 now decides on the list, so a dirty error always has paths.
- existing-paths, LOW: per-resume tackler cost. Rejected: the new way forward goes through a goto, so a resume spawns no tackler.
- bound-breaker, LOW: removing `IsWorktreeDirtyReason` touches tests. Rejected: call-site detail for the plan.
- board-fit, LOW: D3 shares files with `loop-exit-halt-names-its-cause`. Rejected: compatible edits, nothing to decide.
- board-fit, LOW: the way forward names `lyx loom` verbs that `run-verbs-shared` plans to move. Rejected: it is the current convention, and that entry already covers way-forward texts.
- board-fit, LOW: `batten-cancel-verb`'s dirty-source guard. Rejected: a different guard, no overlap.
- walkthrough, LOW: the operator sees the path list twice. Rejected: not a defect.

## Rejected alternatives

- Put the paths into `MergeGuardError.Error()`: breaks the package's closed, side-free reason rule that its vocabulary tests pin.
- Add a `_lyx/shed/**` exclusion to the merge dirty guard: the guard never sees that path, so the exclusion would be dead code.
- Keep the halt note's parent section, limited to the code checkout: it would repeat the stuck reason's list from a later read.
- Read the dirty paths again in Finalize after the refusal: a second read can differ from the one the guard decided on.
- A D4 arm that also refuses every row whose `OnDone` routes to Finalize: a Finalize Stuck appends Finalize's own history entry, so the tackler's routed-from row at a Finalize halt is Finalize itself, and the arm would only refuse orch gotos onto PR-Gate for dirt that does not concern it.
- `lyx loom resume` alone as the way forward: after the tackler's refused resume the hand-back closes the attempt (`attemptClosed` in `internal/sheddriver/drive.go`), so a bare resume spawns a Go driver that steps the blocked Finalize directly (`resumeOpenAttempt` there, `Step` in `internal/shedengine/run.go`), with no tackler.
  D5 now checks that driver too, so the goto is kept because it fails while the parent is still dirty before any driver spawns, and it matches the settled-segment refusal's way forward.
- A new driver rule handing a dirty-parent blocked stop straight back: the rule table reads no reason text except the budget prefix, and a rule there would leave orch gotos unguarded.
- A tackler prompt rule against re-stepping Finalize: an agent rule, where a Go guard decides exactly.
- Refusing the goto when the parent cannot be read: wedges a run whose parent is missing for the moment, while Finalize itself reports that case.
- No cap on the path list: an unbounded reason lands in the status file and the orch's notice.

## Review rounds

### Round 1

- F1, BLOCKING: the way forward named `lyx fabric commit`, which commits only the records pathspec and is refused from task sessions. Fixed: D2 names plain git in the parent's code checkout, run by the operator alone (the self-check narrowed it from the operator or the orch).
- F2, MEDIUM: a hand-over successor re-steps a blocked Finalize with no goto. Fixed: new D5, a told check at a Go driver's start; acceptance criterion added.
- F3, MEDIUM: the routed-from row at a Finalize halt is Finalize itself. Fixed: D4 narrowed to Finalize, reverting Turn 3's routed-from widening; criteria narrowed.
- F4, MEDIUM: a merge record and dirt together still rendered the discard text. Fixed: D2 surfaces the error's own text when `merge already in progress` is present; criterion added.
- F5, MEDIUM: D4's refusal had no refusal-spec class. Fixed: D4 names the transient class.
- F6, LOW: false reason for the goto. Fixed in D2 and the rejected alternatives.
- F7, LOW: the path list collapses to one line. Fixed: Go-quoted paths joined by a comma and a space.
- F8, LOW: the first criterion contradicted the cap. Fixed.
- F9, LOW: D1's premise lacked its evidence. Fixed.
- F10, LOW: a settled fact filed as an assumption. Fixed: moved into D2.
- F11, NIT; F12, NIT: a note pointed at deleted code, and deliberation sat in Scope. Fixed.

### Round 2

- F1, BLOCKING: plain git in the parent's code checkout is no way forward the refusal spec admits. Fixed: D2 decides to widen the spec's item 3 in the same commit; Scope, Constraints and a criterion follow.
- F2, MEDIUM: an operator commit reroutes Finalize to Merge-In. Fixed: D2 states the route, including Merge-In's ahead-of-upstream stop until the operator pushes; criterion added.
- F3, MEDIUM: the merge-in-progress way forward named no verb. Fixed: D2 names the parked merge and `lyx fabric merge --continue` / `--abort` with the role-guard clause, and warns that `--abort` hard-resets the parent's code checkout (self-check); criterion updated.
- F4, LOW: other aggregated reasons beside the dirt. Fixed: D2 keeps the full text and the list, names the dirt step first and then Merge-In's step for each other parent-side reason (self-check); criterion added.

## Question ledger

- Q1 Does `_lyx/shed/**` need excluding from the dirty guard? Resolved by reading: no, the guard reads only the code checkout (D1). Auto-pick: no code change, an acceptance criterion pins it.
- Q2 Where do the paths travel? Auto-pick: a typed field on `MergeGuardError`, rendered by Finalize (D2), because `Error()` must stay path-free.
- Q3 Keep the halt note's parent section? Auto-pick: remove the writer, keep `Parse` tolerant (D3), because the reason now holds the list.
- Q4 How is the re-step stopped? Auto-pick: an arm in the loom and darn goto guard (D4), because every goto route passes it.
- Q5 Guard behaviour on an unreadable parent? Auto-pick: admit and warn (D4 bound).
- Q6 Cap on listed paths? Auto-pick: 20, then `and N more` (D2).
- Q7 Which targets does D4 refuse? Resolved by reading: Finalize alone, since at a Finalize halt the tackler's routed-from row is Finalize itself (round 1, F3).
- Q8 Guard behaviour when a fabric merge record exists on the parent? Auto-pick: admit, leaving it to Finalize's own guard (D4 bound).
- Q9 Way forward? Auto-pick: the operator commits or discards the dirt in the parent's code checkout with plain git, then `lyx loom goto --to Finalize` and `lyx loom resume`, so the guarded goto fails fast while the parent is still dirty.
- Q10 How is the hand-over route into a blocked Finalize stopped? Auto-pick: a told check at a Go driver's start, wired to D4's Finalize arm alone (D5), because the hand-over successor's first step passes no goto.
