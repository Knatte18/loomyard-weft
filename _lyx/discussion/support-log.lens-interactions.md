```yaml
lens: interactions
outcome: reported
```

Task: check the decision record's decisions against each other, pair by pair, for any that undo or contradict one another.

**Pairs checked**

- D1×D11 (only a task is claimable, but a beam's landing claims its project): held.
- D1×D10 (promote's kind change made after cancel clears the claim): held.
- D2×D11 (a run-held project refuses new members; land refuses open members): held, apart from the narrow window noted under finding 7.
- D2 prune × D11/run lock (prune rewriting `order` on a landing project): **broken**, finding 1.
- D2×D10 (promote turns a task into a project while others still depend on it): **broken**, finding 2.
- D3×D9/D10 (`beam` written only by the beam verbs, folded into the owner setter's commit): held only if a dedicated writer exists, finding 6.
- D4×D7 (ParentBeam chooses Worktree-Create's parent): held.
- D4×D11 (ParentBeam after the beam is done): **broken**, finding 3.
- D4 owner fallback × D9/D10: held.
- D5×D7 (Add with Parent from the prime location; dirt checked in the beam worktree in both places): held.
- D5×D6 (`--parent` combined with adopting a local branch): held.
- D6×D12/D13 (a root pair with no parent): held only if `ParentWorktree` is empty too, finding 8.
- D7×D8 (batten run from any worktree; create always uses the prime location): held.
- D8 window placement × D11 (land's window in the parent's session): held. D11 decides placement itself, and a resume from inside the beam is refused by D8.
- D8 orch rules × D11 landing (parent's `rules.md` unchanged): **broken**, finding 4.
- D8 orch gate × D12 mark (D9 writes the mark in Add, D10 writes it itself): held.
- D9×D12 (mark written in the origin commit before Add regenerates the files): held.
- D9 ordering × D7 (board flag written last): held. A member started between steps 1 and 3 forks from the prime, finding 7.
- D9 restart (each step skipped when already done): held.
- D10×D11 (inner run directory and batten records removed, so the landing seeds fresh): held.
- D10×D12 (promote never regenerates the workspace files): **broken**, finding 5.
- D10×D14 (kind change slug check vs "an unchanged slug is never refused"): wording contradiction, finding 9.
- D11×D12 (teardown removes the marked beam file): held.
- D11×D13 (landing notices go to the nearest orch from the beam's parent): held.
- D11 × pairteardown settle (project marked done): held as long as the settle accepts a project entry, which Notes already flags.
- D13×D8 (`ly:<slug>:orch` naming survives the walk-up): held.
- D13 × batten watched-marker rule: weak, finding 10.
- D14×D2 rename (slug check plus rewriting `project`/`order`): held.
- D15×D2 (unordered members after by handle; done project in Done while still named): held.
- Acceptance criteria: the land and promote criteria rest on findings 1, 2 and 5. The notice and workspace criteria hold.
- Constraint PATTERN-pair-teardown: D8, D11 and D10 (promote without `--remove`) hold.

**Findings**

1. **BLOCKING** — D2 × D11 and the run lock: the beam's Land row prune is refused.
   - D2 makes `order` scope, guarded by the run lock, and says prune drops pruned slugs from every `order`.
   - The run lock's only prune exemption is for `depends_on`. boardengine doc: "Prune passes, since a held entry's dependency on a done entry … is not a change; every other depends_on change is refused", and any non-status field change on a held entry is refused.
   - During `lyx beam land` the project is run-held. Its Land row prunes the board, and members are marked done only at their own teardown, so they are typically still unpruned at that point.
   - The prune's `order` rewrite on the held project is therefore refused, and the landing stops at Land. D2 needs the same exemption prune has for `depends_on`.

2. **MEDIUM** — D10 × D2/D1: promote can leave tasks depending on a project.
   - D1 says a project carries no `depends_on`, but nothing stops other tasks from depending on one.
   - Promote changes kind task→project while other tasks may still list the promoted slug in `depends_on`.
   - Today's store refuses the analogous case for notes ("note %q cannot be a note: %s depend on it"). D10 and D2 say nothing, so either promote is refused unexpectedly or dependents are left naming a project that can never be done as a task.

3. **MEDIUM** — D4 × D11: ParentBeam still returns a landed beam.
   - D4 returns the nearest project whose `beam` is true, and D3 says nothing sets it back, so a done beam still qualifies.
   - D11's land check covers only the project's own members and live child pairs. A nested grouping project can be set done by hand while its tasks stay open, then the beam lands.
   - Those tasks later resolve to the done beam, whose pair is gone. D7 then stops with "pair absent", whose way forward (`lyx adopt <beam>`) is wrong.
   - Fix options: ParentBeam skips done projects, or land checks transitive membership.

4. **MEDIUM** — D8 × D11: nothing in the landing keeps the parent's rules file.
   - D8 states "a beam landing leaves the parent's file as it was" for `_lyx/orch/rules.md`.
   - D11's landing is darn's Finalize: it merges the parent in, then fast-forwards the parent. No row or decision excludes `_lyx/orch/rules.md`, so a beam whose orch edited its own rules overwrites the parent's on landing.
   - D8's claim has no mechanism behind it, and no acceptance criterion covers it.

5. **MEDIUM** — D10 × D12: promote writes no workspace file.
   - D12 regenerates after fabric verbs. D10 writes the beam mark through a commit of its own, not through Add, and lists no regeneration step.
   - A promoted beam therefore has no `<repo>-<beam>.code-workspace` until some unrelated fabric verb runs. D9 has an explicit step 4; D10 needs the same.

6. **LOW** — D3 × D9/D10: the owner setter and the beam flag.
   - D3 refuses any payload that changes `beam`, but D9 step 3 and D10 write it in the same Bolt write as the owner setter.
   - The owner setter (`SetOwners`) is the only owner writer and commits "board: owner …" alone. The record should say the setter is extended, or that a dedicated beam writer exists, so the D3 refusal does not catch the verbs themselves.

7. **LOW** — D9 ordering × D7/D4: members started mid-create fork from the prime.
   - Between step 1 (pair created) and step 3 (`beam: true`), a member started by any orch resolves ParentBeam to the prime and gets a prime-parented pair, permanently.
   - The bound is unstated. Separately, D11 checks members at `beam land` and again at batten's start guard, but the guard runs before the claim at Worktree-Create, so a member added in that gap is admitted.

8. **LOW** — D6 × D12/D13: what "records none" covers.
   - D6 says an adopted default branch "records none" for `ParentBranch` but is silent on `ParentWorktree`.
   - If `ParentWorktree` is still recorded from the acting worktree, D12 lists the root pair under that worktree's file and D13 routes its notices to that worktree's orch. The record should say both fields are empty.

9. **LOW** — D14 internal contradiction.
   - It lists "a kind change to project" as a checked write, and also says "a write that leaves an existing slug unchanged is never refused by it".
   - Promote keeps the slug, so the two sentences disagree on whether promote's kind change is checked.

10. **LOW** — D13 × the batten watched-marker rule (battenshed doc).
    - Notices resolve per delivery, but a driver's watched/silent rule is fixed at its launch from `noticesReachDriversParent`.
    - If a beam orch starts after the launch, batten's destination moves to the beam orch while the driver's rendered parent stays the prime orch. The prime orch then gets neither the driver's "run stopped" message nor batten's notice.
    - Open risks covers only the agents' own escalation, not this split.

Further tool calls: none.
