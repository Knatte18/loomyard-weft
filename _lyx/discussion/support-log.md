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
- walkthrough, MEDIUM; goal-scope, LOW; existing-paths, LOW; interactions, LOW: `lyx loom resume` on a blocked Finalize goes through a tackler, and a refused goto from another row then resumes the wrong row. Fixed: the way forward is `lyx loom goto --to Finalize` then `lyx loom resume`, as the settled-segment refusal already does; the assumption is restated.
- leanness, MEDIUM: the three-site requirement sat in the optional notes. Fixed: moved into D2.
- goal-scope, MEDIUM; interactions, LOW; bound-breaker, LOW: D3 drops the paths for task-side `MergeIn` dirty halts too. Fixed: Scope's Out says so and why nothing correct is lost.
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
- A D4 arm keyed on a Finalize target alone: the tackler's routed-from resume re-enters Finalize through the row that routed into it.
- `lyx loom resume` alone as the way forward: on a blocked Finalize it spawns a tackler to get back in.
- A new driver rule handing a dirty-parent blocked stop straight back: the rule table reads no reason text except the budget prefix, and a rule there would leave orch gotos unguarded.
- A tackler prompt rule against re-stepping Finalize: an agent rule, where a Go guard decides exactly.
- Refusing the goto when the parent cannot be read: wedges a run whose parent is missing for the moment, while Finalize itself reports that case.
- No cap on the path list: an unbounded reason lands in the status file and the orch's notice.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Does `_lyx/shed/**` need excluding from the dirty guard? Resolved by reading: no, the guard reads only the code checkout (D1). Auto-pick: no code change, an acceptance criterion pins it.
- Q2 Where do the paths travel? Auto-pick: a typed field on `MergeGuardError`, rendered by Finalize (D2), because `Error()` must stay path-free.
- Q3 Keep the halt note's parent section? Auto-pick: remove the writer, keep `Parse` tolerant (D3), because the reason now holds the list.
- Q4 How is the re-step stopped? Auto-pick: an arm in the loom and darn goto guard (D4), because every goto route passes it.
- Q5 Guard behaviour on an unreadable parent? Auto-pick: admit and warn (D4 bound).
- Q6 Cap on listed paths? Auto-pick: 20, then `and N more` (D2).
- Q7 Which targets does D4 refuse? Auto-pick: Finalize and every row whose `OnDone` routes to it, after the lens review found the routed-from resume.
- Q8 Guard behaviour when a fabric merge record exists on the parent? Auto-pick: admit, leaving it to Finalize's own guard (D4 bound).
- Q9 Way forward? Auto-pick: clean the parent, then `lyx loom goto --to Finalize` and `lyx loom resume`, so no tackler is spawned to get back in.
