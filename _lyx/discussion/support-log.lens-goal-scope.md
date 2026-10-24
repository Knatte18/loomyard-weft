```yaml
lens: goal-scope
outcome: reported
```

Goal-scope lens: checking the decision record against the `umbrella-groundwork` board entry for missing scope, scope that nobody approved, hidden deferrals, and whether this is the right thing to build.

**MEDIUM — D8 (orch) narrows the entry's Want without operator licence.**
- The entry's Want says "`lyx orch` and `lyx batten` run from any drivable worktree in the hub".
- D8 limits orch to the prime and beams, and refuses it in every other pair.
- The reason sits only in the support log ("a task pair has its driver"). The operator views in the entry ("a thread they can open in any worktree", "the orch role should not be unique") lean the other way.
- Fix: either admit any drivable worktree, or state the narrowing in Scope "Out" with its reason in the Decision, so the operator sees it was a choice.

**MEDIUM — Scope / D11 / D7: the cross-machine landing into a beam is neither built nor scoped out.**
- The entry's 2026-10-19 beam-owner decision says: "normally the parent's owner lands a finished run of another machine from its branch on origin, after a ready-to-land handover through Bolt (creel reaches only one machine), with no takeover"; failing that, "the landing machine adopts the parent first".
- The record covers owners only through Constraints (PATTERN-owner-machine) and D7's precondition.
- A grep finds no ready-to-land handover in the code today, so a task on machine B whose beam is owned by machine A has no landing path.
- Fix: decide it in this task, or list it explicitly in Scope "Out" with the board entry that owns it (possibly `umbrella-daemon`).

**MEDIUM — Open risks / D11: a fact the design rests on is left unchecked.**
- The record says a large merge in the beam landing makes "Finalize then halt for the orch instead".
- But darn's Finalize reroutes along its `on_reroute` edge (`landingshed/finalize.go` `rerouteReason`). The `beam` recipe would have no such edge.
- What Finalize does with a reroute and no edge was not checked. It could be a halt, an error, or a recipe-validation refusal.
- Fix: research it now and state the behaviour in D11, or give the recipe an explicit edge or halt.

**LOW — D10: promote skips the workspace regeneration.**
- D10 writes the beam mark after the pair exists but never regenerates the workspace files. D9 does (step 4).
- D12's per-beam file is supposed to be "written when the beam is created". A promoted beam gets its file only at the next fabric verb.
- Fix: add the regeneration step to D10.

**LOW — Scope "Out": an Open point is answered without a reason.**
- The entry's Open point asks "what prime only protects today and whether any of it must stay prime-bound (board writes, deploy, `lyx board prune`, the hub workspace)".
- The record keeps `lyx adopt`, `lyx cleanup` and `lyx shed seed` prime-only with no stated reason (only Q14, an auto-pick).
- It never answers what the prime-only guard protects. A beam's orch can then not run `lyx cleanup`, although the orch rules make cleanup part of its job.
- Fix: give the reason, or decide differently.

**LOW — D9 versus the entry's "Create: its pair, thread and workspace, in one short command".**
- D9 needs a project entry upserted beforehand and starts no orch, so creating a beam takes two or three commands.
- The orch half is justified by the later decision that a beam's orch is optional. The need for a prior board upsert is not justified anywhere.
- Fix: consider letting `lyx beam create` make the project entry when it is absent, or say why it doesn't.

**LOW — D13 replaces the entry's orch sketch without recording it as rejected.**
- The entry's sketch: "the run records the thread's strand as its owner, and notices go to that owner through creel rather than to a fixed `ly:orch`".
- The record routes notices by the beam parent chain instead, and the support log's Rejected alternatives doesn't mention the sketch.
- The sketch was not a decision, so this is only a traceability gap.

**LOW — D8 / Open risks: "sees only its own tasks" is met by stencil text alone.**
- The operator decision says an umbrella's thread "sees and starts only its own tasks".
- The record meets it with a stencil rule plus the `parent_beam` field; nothing filters the views.
- This is acceptable if intended. State that "sees" is met by the `parent_beam` filter in the orch's own reading.

**LOW — Right thing to build: size.**
- The bundle is one loom over about 8 modules: a board kind, fabric, batten, orch, hubgeom, a new module with a recipe, workspace files, and notices.
- The operator asked for the bundle, and the record holds no MVP phase or hidden deferral, so this is not a scope fault. It is a run-size risk, and Open risks doesn't mention it.

Otherwise the entry's asks are covered:
- board projects, membership and README grouping (D1, D2, D15)
- pairs created from a parent pair (D5)
- Worktree-Create from the task's project (D7)
- batten from any worktree (D8)
- the slug check, with the blocked-run cleanup correctly found already met by `lyx batten cancel` (D14)
- promote and land (D10, D11)
- local-branch adoption (D6)
- workspace files per beam (D12)
- the "beam" rename

Calls made:
- grep for a ready-to-land handover and for Finalize's reroute handling: my context had no evidence whether the handover exists or how Finalize behaves with no reroute edge.
