```yaml
lens: code-scope
outcome: reported
```

**Decisions (goal points of the board entry under `## Goal`; the record has no `Added after Discussion` entries and an empty `## Decisions`)**
- G1 (#232): a beam member's run records the beam's orch as its parent, so (G1a) that orch's answer is accepted and (G1b) parent-review asks, notices (dobby's included) and escalations go to the beam's orch.
- G2 (#231): the tackler and driver prompts of a beam member name the beam's orch as parent.
- G3 (#236): the beam's orch can add to or replace its beam's decision record while the beam lives; the caller check counts it as an orch, not a task session.
- G4 (#252, #251): the orch that parents a task, beam orch included, may resume that task's pair (or the notice names a way forward open to it).

**Per change → decision**
- C1 `hubgeom.OrchChain` + `chainWorktrees` + `walkStart` (`nearestlive.go`, `doc.go`, `nearestlive_test.go`) → G1a, G3, G4 (the chain the caller checks read).
- C2 decision `add`/`replace`/`store` admit the chain (`decision.go` `decisionCallerRefusal`, `decisionStoreVerb`, `decisionOrchChain`, `onChain`; `doc.go`; `decision_test.go`) → G3 (`store` forced: `add`/`replace` refresh the board copy through it).
- C3 `parent-review answer` admits the chain (`parentreview.go`, `parentreview_test.go`, loomcli `doc.go`) → G1a.
- C4 hand-back directive: new stencil `parent-directive-handback.md`, `stencils.go` registry, `parentdirective.HandBack`, tackler and arbiter prompts rendering it, stencil header comments, loom `ParentName` wiring (`arbiter.go`, `drivewiring.go`), batten `ParentOrchName` wiring (`drive.go`), `hubgeom.ParentOrchName` + test → G2 (tackler). The arbiter is not named by the entry, but it carries the same no-parent defect in a sibling verdict-only prompt — forced, not flagged.
- C5 tender caller rules (`tendercli/cli.go`, `doc.go`, `cli_test.go`): `callerOf` beam-orch kind, `refuseCaller`, `pairUnderBeam`, `refuseCallerIn`, reap message → G4 (resume); pause/revive admission beyond the entry's ask (see F2); reap wording forced by the new caller kind.
- C6 two comment-rewrap commits (`e571bcbcc`, `d90c67bcc`) → forced by C1–C5.

**Per decision → changes**
- G1a → C1, C3. Built.
- G1b → **none**. No change touches `NotifyParent`/`orchroute.FromPair`/`Asker`, the PR-Review ask destination (`ParentStrand`), or the dobby's `NearestLiveOrch` call (`internal/beamcli/dobbydeps.go:454`). Not built (F1).
- G1 "records the beam's orch" → no change; the recorded parent (`ResolveParent`) already reads the origin record's parent worktree, so what the fault report shows as "recorded" is the nearest-live destination of the ask, covered under G1b.
- G2 tackler → C4. Built. G2 driver → no change; holds: `RenderDriverPrompt` already renders `Directive(stencilsDir, c.parentName, false)` with `c.parentName` from `ReedGeometry` → `ResolveParent` (`internal/loomcli/driverprompt.go:59`, `internal/loomcli/wiring.go:685`); the quoted no-parent wording "so the question is written for whoever owns it" no longer appears in `contracts` or `internal` (my grep).
- G3 → C1, C2. Built; past the caller check the pawl recipe has no approval or Code-Review row, so nothing later refuses the beam's orch (`contracts/recipes/pawl-recipe.yaml`).
- G4 → C1, C5. Built.

**Findings**

- **F1 — BLOCKING, class `scope` — `internal/orchcli/notify.go` / `internal/orchroute/walk.go` (`FromPair`, `Asker`) and `internal/beamcli/dobbydeps.go:454`** — G1b is not built: the entry expects a member's parent-review asks, notices and escalations to go to the beam's orch, but they still go to the nearest orch on the chain that the liveness probe reports live, which in the reported failure passed over the beam's orch; the `[pr-review request]`, LOOM asks and dobby notices will still go to `dop:orch` there. The change description defers this on its own say ("The fault report gives no reproduction… this change leaves the routing alone", `_lym/landing/summary.md` `## Design decisions`). It is the writer's own call, not an `Added after Discussion` entry, and not named as an unrecorded parent correction. Evidence: `NotifyParent` → `parentOrchStrand` → `orchroute.FromPair` with `OrchLive` (`internal/orchcli/notify.go:50`), `NotifyRunFrom` (`internal/orchcli/notify.go:73`), dobby `hubgeom.NearestLiveOrch(worktree, orchcli.OrchLive)` (`internal/beamcli/dobbydeps.go:454`); the diff touches none. Fix: either route a member's asks/notices/escalations to its recorded parent orch, or find and fix why `OrchLive` reads a live beam's orch as down; the deferral is the parent's to rule, not the writer's.

- **F2 — NIT, class `scope` — `internal/tendercli/cli.go` `refuseCaller`** — the beam's orch is also admitted to `pause` and `revive` of its beam's pairs, while #251/#252 ask only for `resume`. Evidence: `refuseCaller`'s `c.beam != ""` branch serves all three verbs (diff piece 011 lines 1198-1208). Mitigation: the orch role text already tells an orch to "hold the run with `lym tender pause <slug>`" (`contracts/orchrole/role-beams.md:105`), so a beam orch that could resume but not pause would be stuck by its own role text; this reads as forced by the traced change and the handler may reject it.

Tool calls:
1. grep of `contracts` and `internal` for the driver no-parent wording "written for whoever owns it" — context lacked whether any stencil still carries it, which settles whether G2's driver half needs a change.
