```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens, covering what the diff's changes force outside the lines they touch.

**Per change, the set found and whether it is covered**

1. **New `Deps.IsAncestor` seam** (`internal/landingshed/deps.go`). The diff covers every place that builds or lists the seam. I searched for every `landingshed.Deps` literal and every seam list:
   - production wiring at `internal/loomcli/landingdeps.go:346`: covered.
   - `internal/loomcli/wiring_seams_test.go:36` and `internal/battencli/wire_seams_test.go:116`: covered.
   - `internal/testkit/envkit/envkit.go:239`: covered.
   - `internal/landingshed/fixerrun_test.go:170`: covered.
   - `internal/landingshed/parentalign_test.go:147` sets `TaskRemoteTip` to a fixed `remote-1` and leaves `IsAncestor` nil. A fixed tip never takes the moved-tip branch, so the nil seam is never called. Not stale.
   - The `IsAncestor:` hits in `internal/shedarbiter/call_test.go:162` and `internal/shedworktree/shedworktree.go:26` belong to other structs. Not affected.

2. **`TaskRemoteTip` doc text** (`deps.go`): the doc now says any writer may have moved the ref, which matches the new behaviour. The only other readers are `fixerrun.go:141` and `fixerrun.go:185`, both in the diff. Covered.

3. **`fixerOutcome`/`fixReviewOutcome`: `sessionLog` becomes `session`, `pushedRef` becomes `remoteRef`/`remoteTip`/`startHead`.**
   - Readers: `haltReason` (`fixerrun.go:408-420`), `verdictSummary` (`fixreview.go:274`) and the fix review "changed" halt (`finalize.go:1144`). All updated.
   - A search for `sessionLog` and `fakeFixerRunDir` in `internal` finds no stale use; `fakeFixerRunDir` is still used on purpose in `fixerrun_test.go`. Covered.

4. **Halt-reason wording** ("session log:" becomes "session:", and "pushed …"/"check the pushed commits" becomes "moved during"/"deleted during"):
   - Go assertions: `fixerrun_test.go` and `finalize_gate_test.go:851,864` are updated.
   - Nothing else in `internal` matches "session log" or "check the pushed commits". The one hit, `battenshed/teardown_test.go:2`, is about another subject.
   - Stencils, specs and non-research docs: no match. `docs/overview.md:1418-1441` describes Finalize halts without the pushed end or the session-log text, so it is not stale.
   - Halt consumers read only `LandingStopPrefix`, and the prefix is unchanged. Covered.

5. **The `pushed` end now has narrower meaning** (`fixerEndPushed` comment, `run` doc, `internal/landingshed/doc.go:52-56`): all updated. Covered.

6. **Trace line in `gitrepo.recordLandedPush`** (`push.go:183`):
   - It is reached from `pushPlainWithRetry` (used by `Push`, `PushBranch`, `PushCommit`, `PushRebaseFree` and `PushCoalesced`, retries included), and from `DeleteRemoteBranch`, `DeleteRemoteBranchLeased` and `UpdateRemoteBranchLeased`.
   - `PushWithRetry` has no other production caller (`pushretry.go:113`, `push.go:164`).
   - The only other raw `push` site is `fabricengine/archive.go:277`, a tag push, not a task code branch.
   - `internal/gitrepo/doc.go:98` is updated. `pattern/PATTERN-gitrepo-client-boundary.md` says nothing about push logging, so it needs no change. Covered.

7. **Repository rules** (`PATTERN.md`, `pattern/`):
   - No entry describes the fixer's pushed end or the halt text.
   - PATTERN-fabric-boundary: git runs through fabric (`fabricengine.Open(l).IsAncestor`) and gitrepo (`TaskRemoteTip`); the `TaskRemoteTip` read is pre-existing.
   - No declared PATTERN changes are listed for this round. Covered.

8. **Commit messages and departures:**
   - 8562edcea carries a `Root-Cause:` trailer naming the defect the diff removes: the moved remote tip was not tested against the start head, and `RunDir` was named although it is removed on a done outcome.
   - The range has no `Pattern-Departure:` or `Loosened-Check:` trailers, and none is needed. Covered.

**Findings**

None.

**Tool calls**

1. grep for `TaskRemoteTip:`/`IsAncestor:` Deps literals, old halt wording and stale test symbols across `internal`, stencils, specs and docs: the manifest did not list every Deps construction site or text consumer.
2. grep in stencils, specs and docs for pushed-end or run-dir wording: the manifest covered no stencil or doc text.
3. grep in `docs/overview.md` for those terms: to see whether the hits in call 2 were about this subject.
4. grep in `docs/overview.md` for fixer halt mentions, and for references to the seam lists: to check the reference doc and the seam lists.
5. Read `docs/overview.md:1418-1441`: to confirm its Finalize halt passage does not state the old behaviour.
