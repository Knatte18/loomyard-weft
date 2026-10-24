```yaml
lens: interactions
outcome: reported
```

Pairs checked:
- D1 × D10 (no task may depend on a project vs promote refusing only an *open* dependent): broken.
- D2 × D10 (membership/run lock through promote; claim cleared at step 1 before kind change at step 3): held.
- D2 rename × D4/D7/D12 (renaming a live beam project, whose pair branch is derived from its slug): broken.
- D3 × D9 (flag first; a stopped create only finishable by `create` re-run): held.
- D3 × D10 (owner setter "gains the beam flag" only, yet promote also changes kind and clears fields through it): broken (folded into D14×D10 finding).
- D4 `parent_beam` (board) × D8 orch stencil filter × D9 (members spawned under a grouping project before it became a beam): broken.
- D4 × D10 (ParentBeam vs origin parent of promoted beam): held.
- D4 × D11 land refusal (ParentBeam + origin records; conservative over pre-spawned members): held.
- D5 × D7 × D8 (Parent from ParentBeam with prime acting, run from a beam; prime dirt still checked): held.
- D6 × D4/D12/D13 (root pair with empty parent → prime, listed in hub file, walk terminates at prime): held.
- D7 × D9 (precondition wording during a stopped create): broken.
- D8 `--window` placement × D11 (land's window in parent's session): broken.
- D8 orch in beam reed session × D11 teardown of beam pair: broken.
- D8 orch name × D13 (ResolveParent changes to liveness walk): held.
- D9 × D11 (land of a beam whose create stopped before its pair): goes via D7; covered by D7×D9.
- D10 × D12, D9 × D12 (beam mark in origin commit, regeneration last step): held.
- D11 × D1/D2 (project done at settle; members keep `project` naming a done project): held (D2 exempts unchanged).
- D11 × D7 (landing's own Worktree-Create finds pair present; refusal re-applied post-claim): held.
- D12 × D6 (adopted root pair without beam mark): held.
- D13 × D8 (only prime/beams host an orch, so the walk passes over task pairs): held.
- D13 × Open risk 2 (driver parent fixed at launch): stated, held.
- D14 × D10 (kind change to project through owner setter, which D14 says checks nothing): broken.
- Declared PATTERN-pair-parent × D11/D13/D5: broken.
- Acceptance × D14, D10, D12: gaps.

Findings:

1. **BLOCKING · consistency · `## PATTERN changes` (PATTERN-pair-parent) vs D11, D13** — The declared entry says a board entry's parent is read "only" through `ParentBeam` and hubgeom's one `ParentOf`, but D11's land refusal reads "any local pair's origin record names the beam as its parent" directly, and D13's `ResolveParent` walks origin records upward directly; neither goes through `ParentOf`. Fix: narrow the entry to owner reads and new-pair parent picks, or route D11/D13 through `ParentOf`. Evidence: decision-record.md:263, :172, :194; `internal/hubgeom/parent.go:35-53` reads the origin record directly.

2. **MEDIUM · consistency · D4 `parent_beam` × D8 stencil filter × D9** — D9 keeps already-spawned members of a grouping project on the prime as parent and they land there, but `parent_beam` is `ParentBeam` over the board, so once the project becomes a beam those members read `parent_beam` = the new beam: the prime's orch, filtering by `parent_beam`, stops seeing runs that land into the prime, and the beam's orch sees runs it neither spawned nor receives. `parent_beam` for a spawned entry should come from the one `ParentOf` (origin first). Evidence: decision-record.md:92 (list/get carry `ParentBeam`), :137 (filter), :151 (members keep the prime).

3. **MEDIUM · design · D2 rename × D4/D7/D12** — D2 lets rename rewrite `project`/`order` and refuses nothing for a beam; a live beam project holds no run lock, so it can be renamed. Its pair's branch is derived from the slug (operator decision in the board entry), so after rename `ParentBeam` names a slug with no pair and every member stops at D7's precondition, and D12's mark-based cleanup sees a beam file whose beam pair is "gone". No Decision refuses or handles renaming a `beam: true` project (or a spawned task). Evidence: decision-record.md:71, :75, :189; board entry "The umbrella's branch is derived from the slug and fabric's `branch_prefix`".

4. **MEDIUM · design · D8 orch hosting × D11 teardown** — D8 runs a beam's orch in the beam pair's reed session with its watcher and files under the beam's `.lyx/orch/`; D11 then tears the beam pair down, and pair teardown ends the pair's reed session before removal. Nothing states the orch's end: the session is killed, the watcher's fate is unstated, and D8's tender/overview/cleanup readers then hold an orch whose worktree is gone. D11 neither refuses land while the beam's orch is live nor stops it as a step. Evidence: decision-record.md:133, :174; `pattern/PATTERN-pair-teardown.md` (session end before worktree removal).

5. **LOW · consistency · D7 × D9** — D9 step 1 records this machine as the project's owner before step 2 makes the pair; a member started between them (D9 says it "stops at Worktree-Create with D7's precondition") meets D7's rule "`lyx adopt <beam>`" since the board does name an owner — wrong on the owner machine, where the way forward is re-running `lyx beam create <beam>`. Evidence: decision-record.md:121, :145, :150.

6. **LOW · consistency · D14 × D10 (and D3)** — D14 applies the slug rule to "a kind change to project", yet says the owner setter "checks nothing; none of them makes a slug"; D10 step 3 makes exactly that kind change through the owner setter, so the stated write site is unguarded. D3 also scopes the owner setter's extension to the beam flag alone, while D10 needs it to change kind and clear four fields. State the promote kind change as exempt (slug already names a pair) or give the owner setter the seam. Evidence: decision-record.md:81, :163, :201, :204.

7. **LOW · consistency · D1 × D10** — D1: "No task may depend on a project"; D10 refuses promote only when an *open* task depends on it, leaving a done task's `depends_on` naming a project. The record doesn't say D1's refusal judges only the written entry; if not, D10 step 3 is refused by D1's check when a done dependent exists. Evidence: decision-record.md:54, :158.

8. **LOW · consistency · D8 `--window` × D11** — D8 places `lyx batten run --window`'s window in the acting worktree's reed session; D11's land opens it in the parent worktree's session with prime cwd. Which mechanism land uses to get the parent's session is unstated (neither the beam nor necessarily the prime is the parent), and land's batten process runs from the prime, so D8's self-pair refusal never fires for it. Evidence: decision-record.md:131, :172.

9. **LOW · consistency (method) · Acceptance criteria vs D10, D12, D14** — Acceptance lines cover a subset of what their Decisions decide: D10's refusals for no pair / owned elsewhere / past the parent-side merge are not reached (only the dependent one); D14's project-slug upsert and kind-change write sites are absent (only "a new task slug"); D12's gone-or-unrecorded-parent rule is absent. One method gap: acceptance lines were not derived from each Decision's refusals and rules. Evidence: decision-record.md:158, :188, :201, :249, :257, :258.

Tool calls: none.
