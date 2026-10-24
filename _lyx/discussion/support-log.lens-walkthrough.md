```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism on one concrete case and checked the end state against the decision's stated purpose.

**Findings**

1. **BLOCKING** — D9 (`lyx beam create`), with D4 and D5: when run from a beam, a top-level beam is forked from the acting beam, not from the prime.
   - Case: from beam B's worktree, `lyx beam create C`, where C has no parent project.
   - D4 gives `ParentBeam` = "" for the prime. D9 step 1 calls `Add` with `Parent` = "".
   - D5 says an empty `Parent` "keeps today's fork from the acting worktree's HEAD", so C forks from B's branch and records B as its parent.
   - D7 avoids this by passing the prime's location as the acting location. D9 says no such thing, yet it states the verb "runs from any drivable worktree".
   - The prime cannot be named as `Parent` either, since D5 takes pair slugs only.
   - Purpose (parent from the project chain) is missed. The same gap hits any future caller of `AddOptions.Parent` with an empty `ParentBeam` result.

2. **MEDIUM** — D10 (`lyx beam promote`): a promoted beam's weft keeps the stopped run's `_lyx/discussion`, `_lyx/plan` and other files, and its members inherit them.
   - Case: promote task T, which ran Discussion and Plan, then start member M under it.
   - D10 removes only the pair's inner run directory.
   - The prime weft holds only `_lyx/config`, `fabric`, `shed` and `webster`, so a task forked from the prime starts with no discussion files. That is because weft content is "never a merge participant" (`fabricengine/doc.go:97`).
   - A fork skips only the shed run-records root (`Add`'s doc). So M's weft, forked from T's weft, carries T's decision record and plan into M's own loom run.
   - Purpose (the beam is a clean starting point for its members) is missed.

3. **MEDIUM** — D10 and D2: promoting a task that other tasks depend on is not settled.
   - Case: task U has `depends_on: [T]`; promote T.
   - D1 gives projects no `depends_on`, and the store already refuses to make an entry a note while tasks depend on it (`store.go:367`).
   - D10's single Bolt write would either be refused by the same rule extended to projects, or leave U depending on a project. The record says neither, and D10 lists no refusal or cleanup for dependents.

4. **MEDIUM** — D13: the ready notice's mechanism takes the wrong input.
   - Case: T1 lands in beam B and makes T2, a member of B that has no pair yet, ready.
   - D13 names each newly ready task "to its own parent's orch" through `ResolveParent`. `ResolveParent` takes a pair's Location and reads its origin record, which an unspawned task does not have.
   - The step needs `ParentBeam` (D4), then the walk from that beam's worktree. As written, the input does not exist.

5. **MEDIUM** — D11: the `beam` recipe's Finalize has no reroute target, and the record leaves what happens then as an unchecked claim.
   - Darn's Finalize reroutes a conflict or a large merge to `on_reroute: Code-Bouncer`.
   - The `beam` recipe has no such row. Open risks says "Finalize then halts for the orch instead", but nothing records what Finalize or shed does with an empty `on_reroute` (halt, error, or a recipe-load refusal).
   - This is a fact the design rests on, left for Plan to find out.

6. **LOW** — D10 and D12: a promoted beam gets no workspace file until some later fabric verb runs.
   - Case: promote T. D10 writes the beam mark on an existing pair.
   - D12 regenerates after fabric verbs that change the hub's worktrees, and D10 lists no regeneration step (D9 does, as its step 4).
   - Until then, T is listed in the hub file as a plain task and has no `<repo>-T.code-workspace`.

7. **LOW** — D8: "a beam landing leaves the parent's file as it was" is stated as a requirement, but it already holds.
   - Weft content never takes part in a landing merge (`fabricengine/doc.go:97`).
   - The Plan writer may build a mechanism that is not needed. It could be stated as a fact with that evidence.

8. **LOW** — D7 and D5: every member start is refused while the beam's worktree has tracked uncommitted edits.
   - Case: the operator edits files in beam B (their long-lived working branch) and the orch starts B's task.
   - The beam-side dirty check refuses the start.
   - Prime starts behave the same today, but the Goal is that the operator works in the beam, so this refusal will be routine. Neither the record nor its Open risks mention it.

**Per decision: case used, purpose met or not**
- D1: an upsert of a project carrying `recipe` → refused. Met.
- D2: rename of a member slug → `project` and `order` rewritten. Met.
- D3: an upsert setting `beam: true` → refused. Met.
- D4: a member of a grouping project inside beam B → B. Met for pairs and for owners.
- D5: `Add` with `Parent` = B → origin names B. Met. An empty `Parent` from a non-prime caller is not met (finding 1).
- D6: `lyx fabric add main` with the prime on a beam branch → adopted and records no parent, and the Land row stops on that (`land.go:190`). Met.
- D7: a task of B started from the prime → `Add` gets the prime location with `Parent` = B. Met. Dirty-beam friction is finding 8.
- D8: batten status read from a beam → paths under the prime. Met. The rules-file claim is finding 7.
- D9: create from a beam → not met (finding 1).
- D10: promote of T → not met (findings 2, 3 and 6).
- D11: land of B with all members done → Finalize, Land, teardown, project done. Met, except the reroute question (finding 5).
- D12: create and land of B → its file is written and then removed. Met for create; promote is finding 6.
- D13: a run in an orch-less B → the prime's orch. Met for pairs; the ready notice is finding 4.
- D14: an upsert of `foo-weft` as a task → refused. Met.
- D15: the Projects table renders. Met.

**Tool calls**
- Listed `loomyard-weft/` and its `_lyx`: to learn what a prime weft holds, which my context lacked.
- Grepped for raddle users and for discussion stripping in landing: to find how task records are kept out of the parent, not in my context.
- Grepped `fabricengine/doc.go` for weft-at-landing wording: to confirm weft content never takes part in a merge, not in my context.
