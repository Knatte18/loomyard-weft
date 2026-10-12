---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:134
    summary: D10's time join maps landings to runs with no archive tag, but the hand test, the label lookup's run identity and the land-incomplete flag all read sources D4 and Scope define only for tagged runs, so every time-joined run is undefined or misread as hand
    fix: In D10, state that a run mapped by the time join is never `hand`, because the checkpoint commit it joined through is written by Finalize itself, and that `hand` is tested from the tag tree's loom history only for runs mapped by the exact join; state that a time-joined untagged run is identified by its slug and contributes only to the landing table, never to per-run rows, and that its `land-incomplete` flag is absent since D4 defines no batten span without a tag. In Scope Out, amend the in-flight bullet to say a landed pair not yet torn down still maps its landing under D10 but enters no per-run figure. Extend the acceptance criterion at line 198 so the not-yet-torn-down landing is asserted to keep its label kind and never be `hand`.
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:134
    summary: The time join names the slug's checkpoint commit and board done commit as if each were unique, but a reused slug leaves one pair per run and Finalize's retry path commits a second checkpoint, so which pair bounds a run's window is left open
    fix: In D10's time-join sentence, state that a tagged run's window opens at the newest checkpoint commit for the slug dated after the slug's previous archive tag's commit and at or before this tag's commit, mirroring D4's own-seed rule, and that an untagged run's window opens at the newest checkpoint for the slug after the slug's newest archive tag; state that the window closes at the first board commit marking the entry done after that checkpoint; name which commit date (committer date) every window comparison uses. Add an acceptance criterion that a slug with two runs, each with its own checkpoint and done commit, maps each landing to its own run, and a run with two checkpoints opens its window at the newer one.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:129
    summary: With landing.yaml squash false, Finalize's parent-side merge after its catch-up merge-in is a fast-forward, so every task commit becomes a first-parent commit; D10 counts each as a landing and all but the tip become unmapped and are blamed into the unmapped share
    fix: In D10, after the exact-join sentence, state that when a run's landing is a fast-forward, the first-parent commits between the previous landing and the run's task commit belong to that landing and are neither counted as landings nor classed `unmapped`; add the matching clause to D11 so those commits are not blamed. Add an acceptance criterion that a non-squash fast-forward landing of a task branch with several commits yields one landing and no `unmapped` commits.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:201
    summary: The criterion says a commit landed by another child inside a run's window is not claimed by that run, but D10 takes the first first-parent commit in the window and only excludes commits already claimed, so a hand landing or direct push in the window that no exact join claimed is taken
    fix: Either narrow the criterion at line 201 to a commit already claimed by the exact join (another tagged child's landing in the window is not claimed by the time-joined run), or add to D10 that a time-joined landing must also differ from every commit dated in the window that is a direct push, and state how that is told; the narrower criterion is the recommended fix, with the residual direct-push case named in Open risks.
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:78
    summary: Several D6 classification rules that carry a requirement have no criterion a test can fail on, and failure episode is undefined, so a Land or Worktree-Teardown failure count cannot be pinned
    fix: Walk every classification rule in D4 to D11 and add an acceptance criterion wherever one has none of its own, not only the instances named here. In D6's fourth bullet, define a failure episode as the batten history entries from the row's first failed state through its next done or the end of the history, counted once. Add acceptance criteria for the rules that have none, among them that a blocked change at a producer other than a Bouncer or PR-Gate is one wake of kind state plus producer; keep-alive pings are counted beside parent-review wakes and add none; a Worktree-Teardown failure episode with two failed entries is one wake; alignment outcomes are reported per point and verdict.
---

### [BLOCKING] Time-joined runs have no defined loom history, run identity or batten span

**Location:** `_lyx/discussion/decision-record.md:134`

**Issue:** D10 applies the time join to "a run ... whose pair is not yet torn down", so it maps landings to runs that have no archive tag.
D4 defines a run's loom records as "the tree at its tag" and its batten span relative to the slug's archive tags, and Scope Out (line 22) says a run counts only once its tag exists.
D10's hand rule then reads "a mapped run whose loom history holds no Finalize `done` entry", which has no source for an untagged run: read literally, the history is empty and every time-joined run is `hand`, though the checkpoint that joined it is written by Finalize inside its own sequence (`internal/landingshed/finalize.go` Step 2b, `internal/loomcli/landingdeps.go` `parentRecordsCommitter`), so Finalize performed that merge.
The focus asks that `hand` follow who performed the merge; for this branch of the join the decision record gives no rule that does.
The same gap leaves `land-incomplete` undefined for these runs and contradicts the Scope Out bullet.

### [MEDIUM] Which checkpoint and done commit bound a reused slug's window

**Location:** `_lyx/discussion/decision-record.md:134`

**Issue:** The window opens at "the slug's `loom: run records checkpoint for landing <slug>` commit" and closes at "the board commit marking the slug's entry done".
D4 establishes that a slug can be reused, one run per tag, and the commit message carries only the slug, so the records branch holds a checkpoint per run.
Finalize calls `commitParentRecords` twice when its first parent-side merge needs a merge-in (`finalize.go` Step 2b and again on the Step 5 retry), so one run can leave two.
D4 settles the analogous question for fresh seeds with an explicit interval rule; D10 does not, and neither says which date a window comparison reads.

### [MEDIUM] Non-squash fast-forward landings flood `unmapped`

**Location:** `_lyx/discussion/decision-record.md:129`

**Issue:** D10 takes "the first-parent commits on the prime's parent branch in range" as landings, and names a fast-forward as one of the exact-join shapes.
`landing.yaml`'s `squash` key is hub-wide and configurable (`internal/landingshed/config.go`), and the verb is meant for any hub (D1).
With squash off, Finalize merges in the parent under the parent lock first, so the parent-side merge is a fast-forward (`fabricengine` reports `MergeFastForwarded`), and the parent's first-parent chain then runs through every task commit.
Only the tip maps to the run; the rest match no source and become `unmapped`, which D11 blames into the `unmapped` share, so the share measures commit granularity, not direct pushes.
This hub sets `squash: true`, which keeps the defect latent here, hence MEDIUM.

### [LOW] Window-exclusion criterion asserts more than D10 delivers

**Location:** `_lyx/discussion/decision-record.md:201`

**Issue:** D10 claims "the first first-parent commit dated in that window" and excludes only commits already claimed.
The parent lock keeps another Finalize landing out of the window, but an orch hand landing (`contracts/stencils/orch/orch-template-role.md` squashes onto main by hand) or a direct push takes no such lock; if no exact join claimed it, the time join takes it and the run's real landing becomes `unmapped`.
The criterion's first clause states the opposite outcome, so a test written from it cannot pass against the rule as written.

### [MEDIUM] D6 rules without a failable criterion, and an undefined episode

**Location:** `_lyx/discussion/decision-record.md:78`

**Issue:** The generic wake kind ("else the state and producer"), keep-alive pings counted beside parent-review wakes, the Worktree-Teardown failure wake and the per-point alignment outcomes each carry a requirement, and no acceptance criterion exercises them; line 183 says it checks counts only and defers classification to the criteria below.
"A failure episode of Land or Worktree-Teardown" is never defined, so the Land criterion at line 204 ("one wake") has no rule telling one episode from two.
This is the third round with the same method gap: round 1's F8 named the Worktree-Teardown failure wake and round 2's F7 asked for one criterion per classification rule, and fixes added criteria for the named examples only.
The fix therefore walks every classification rule in D4 to D11 and adds a criterion wherever none exists, not only the instances named here.

## Focus verdicts

- D10: the decision now names Finalize as the merge's performer and no join uses batten's Land `done` commit, which matches `landingdeps.go` and `internal/battenshed/doc.go`.
  The checkpoint-before-merge join is bounded by Finalize's own done commit, and the claim-once rule is stated; the remaining defects are F1 to F4.
- D4, D5, D9: one tolerance rule holds across D5, D9, the Notes and Constraints: a separate tolerant parse-from-bytes function per owning package, with `shedrun.ParseSeed` (strict, `DisallowUnknownFields`, called by `ReadSeed`) unchanged.
  D4's own-seed interval yields an absent batten source when no fresh seed falls between the previous tag and this one, with criterion 187 pinning it.
  Not a defect.
- Notes, D7, criteria: the verdict pointer names the tolerant function D5 adds in `alignment`, so it agrees with the rule and avoids the path-based `ReadVerdict` (`KnownFields(true)`).
  D7 counts a clean Tier 2 run as absent, not zero stops, and states it, with the consequence in Open risks; the friction dir is tracked (`loomengine.LoomFrictionDir`), so the premise holds.
  Criterion coverage gaps are F5.
