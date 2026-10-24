```yaml
lens: bound-breaker
outcome: reported
```

Task: the bound-breaker lens over `decision-record.md` and `support-log.md`. I attacked every bound, guard, ordering promise and code claim in the record and report each one held or broken.

**Claims attacked**

- D11 "Worktree-Create finds the pair present and skips": **broken.** The start guard refuses before that row is reached.
- D8 "the only run whose agents work in a beam is the beam's own landing": **broken as a bound.** It names the exception but limits nothing.
- D5 "does not refuse tracked dirt in a named parent's worktree" and D7 "checks none for one forked from a beam": **broken.** Add's dirt check reads the acting worktree, which is the prime.
- D4 "only when no record is found, `ParentBeam`", read against the D4 and PATTERN-owner-machine owner promise: **broken.** A found record whose parent branch has no local worktree reads as the prime.
- D11 "a large merge runs the gate like any other": **broken as a bound.** An unreviewed resolution by the conflict session lands with nothing stated to limit it.
- D3 hand-set `beam: true` → D7 precondition → D9 "admits that create": **held, with a gap.** No owner recorded reads as the prime's owner. On a hub whose prime owner is another machine, create is refused and points at adopt.
- D12 "`beam` written only by `lyx beam create` and `lyx beam promote`": **held under the deploy-first Open risk.** An older binary keeps an inherited `beam` key (state.go:28; add.go:439). Only the "every machine deploys first" rule stops that.
- D5 "Landing a member does wait on them: Finalize keeps its Stuck": **held.** finalize.go:338, :371.
- D6 "the Land row already stops on an origin record without a parent branch": **held.** land.go:190. childSeedParams also leaves the seed with no parent (wire.go:526).
- D10 "an entry left an open unclaimed task by a promote stopped after step 3 is never started": **held**, through the new marked-pair refusal.
- D10 "a promote stopped before step 2 leaves the run claimed": **held**, as stated.
- D11 "no orch starts between the claim and the teardown": **held.** The orch refusal checks the run-held entry, and Worktree-Create re-checks after the claim.
- D11 the Land row "prunes the board and deploys when the parent is a deploy branch": **held.** land.go:199, :289.
- D11 "a spent or over-cap fixer halts for the orch": **held.** With `RerouteReviewed` nil, Finalize halts (deps.go:305).
- D13 "a stopped orch keeps its recorded strand", so liveness decides: **held.** orchcli/start.go:450 reads strand liveness.
- D9 "a member started between steps 1 and 2 stops at Worktree-Create": **held.** The beam branch is absent from origin, so the precondition names `lyx beam create`.
- D14 "a board built without the seam … makes no slug": **held.**
- D2 project-change refusal "on an entry whose pair exists on this machine": **held as stated.** D11's Bound admits the other-machine hole.
- D8 "`hubgeom.ResolveParent` already gives `ly:<slug>:orch`": **held, with a caveat.** parent.go:83-92 names the parent worktree, which is the slug for a beam. The `ly:` prefix is this hub's shortname, not a constant.
- D1 "a done task whose unchanged `depends_on` names a promoted project refuses nothing": **held**, by the changes-only rule.

**Findings**

1. **BLOCKING · design · D11 / D10.** `lyx beam land` starts `lyx batten run <slug>` on the beam project. That run has no batten status file, because a promote clears batten's records (D10 step 3) and a create writes none. The beam pair is present. Batten's start guard refuses exactly that case: "left a pair from an earlier run … run `lyx batten cancel <slug> --remove`". So the landing can never reach Worktree-Create's skip, and the guard's way forward would remove the beam itself. The record never exempts a beam project from that refusal.
   - Evidence: internal/battencli/startguard.go:77-85, startguard.go:89-101.

2. **MEDIUM · design · D5 / D7.** A member of a beam B is created with the prime as acting location and Parent=B. `add` runs `SourceDirtyPaths(l)` on that acting location, the prime, before it reads any parent. Worktree-Create maps `ErrSourceDirty` to a PreconditionError. So tracked dirt in the prime stops every beam member's create. That contradicts D7's "checks none for one forked from a beam" and D5's rule that the dirt that matters is the named parent's.
   - Evidence: internal/fabricengine/add.go:188-193, internal/battencli/wire.go:321-327.

3. **MEDIUM · design · D4 / Constraint PATTERN-owner-machine.** The counterexample: a member pair whose origin record names beam B's branch, on a machine where B has no live worktree. B could be pruned, removed, or never made here because the member was adopted from origin.
   - `ParentSlug` finds the record but returns `""`, because no live pair holds the parent branch.
   - D4 falls back to `ParentBeam` only when no record is found, so the member reads as prime-owned.
   - `Owners.Effective` then answers the prime's owner, not B's. That breaks the rule that an unrecorded owner reads as its parent's owner.
   - Evidence: internal/fabricengine/origin.go:139-157, internal/boardengine/owners.go:118-125.

4. **MEDIUM · design · D8 (attack surface).** The wider guard admits any session in a beam's worktree. The beam's own landing runs darn's Finalize there, with its merge fixer and conflict session. Nothing denies `lyx batten`, `lyx adopt`, `lyx cleanup` or `lyx beam` to agents, so during a landing those agents may run any of them against any run of the hub. D8 names this run as the exception but states no bound on it.
   - Evidence: internal/shellguard/rows.go has no batten row (grep of rows.go and the agent settings found none); landingshed/doc.go:33 (darn's resolver runs the conflict session).

5. **LOW · design · D11 (attack surface).** For the beam's Finalize, D11 drops the large-merge reroute, the guard that sends a conflicted or overlapping merge to review. With it gone, a conflict-session resolution of the beam against its parent's newer work lands behind only the `finalize` gate. The rationale "each member was reviewed" does not cover that resolution. Nothing states that what lands unreviewed is the resolution, or what limits it.
   - Evidence: internal/landingshed/doc.go:33-36, finalize.go:871-883.

6. **LOW · design · D3 / D9.** A hand-set `beam: true` on a project with no recorded owner reads as owned by the prime owner. When the prime owner is another machine, D9 refuses create here, naming adopt. So D3's bound "D9 admits that create and makes the pair" holds only on the prime owner's machine.
   - Evidence: internal/boardengine/owners.go:104-125.

**Tool calls**

- grep `SourceDirty` / `ParentBranch` in battencli, adoptcli and landingshed: to see how Worktree-Create surfaces Add's dirt refusal, which the manifest did not cover.
- sed of wire.go:312-335 and :515-530: `createRefusal` and `childSeedParams`, to confirm the mapping of `ErrSourceDirty` and how an empty parent branch is handled; these bodies were not in context.
