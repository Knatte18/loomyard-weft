---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/pairbranch.go:20
    summary: A pair made on one machine cannot be found or re-adopted on another machine whose branch_prefix differs, contradicting the board entry's premise that such a pair keeps its name when adopted elsewhere.
    fix: >-
      In internal/fabricengine/pairbranch.go, make the origin lookup independent of the current machine's prefix.
      In originTaskBranch and originPairBranch, keep trying pairBranchNames(t.cfg, slug) first.
      When both names miss, list origin's warp branches and take the one whose name is the slug or ends in "/" plus the slug, and whose records branch (RecordsBranchName of that name) exists on the weft origin.
      agentname.ValidateSlug admits no "/", so the suffix match is unambiguous for one slug.
      Return an error naming every candidate when more than one matches.
      In internal/fabricengine/add.go around line 229, under AdoptOnly and AdoptLocal, take the warp branch name from that lookup when it finds one, and use t.cfg.BranchPrefix+slug only for a new pair.
      That way the re-adopt seam in internal/battencli/readopt.go adopts the pair's existing branch.
      PairTaskBranchOrOrigin's callers in ownerarchive.go and pairteardown/pullrequest.go then read the right branch with no change.
      Add one row to the existing pair-branch integration test: a pair pushed under prefix a/ and re-adopted with prefix b/ resolves to a/<slug>.
      Update the "Open risks" bullet of _lym/landing/summary.md to match what the code now does.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-affected-set
    location: internal/fabricengine/pairbranch.go:29
    summary: slugOfBranch strips only the current machine's prefix, so a branch another machine made (a/foo read under prefix b/) gets the slug a/foo in TaskBranchSlugs, the orphan sweep and the remote-warp sweep, including a teardown lock path that differs from the pair's own.
    fix: >-
      In internal/fabricengine/pairbranch.go, change slugOfBranch to trim the current prefix when the branch carries it.
      Otherwise trim everything through the branch's last "/", since agentname.ValidateSlug admits no "/" in a slug.
      Update its doc comment to say so.
      The callers in pairparts.go (TaskBranchSlugs), cleanup.go (cleanupOrphan) and cleanupremotewarp.go (deleteLeftoverUnderLock, keptReason) then derive foo from a/foo and take foo's teardown lock.
      Add a row to the test that covers TaskBranchSlugs: branch a/foo, read under prefix b/, lists slug foo.
  - id: F3
    severity: MEDIUM
    class: decision
    origin: lens:code-scope
    location: _lym/landing/summary.md
    summary: The board entry's second goal point, to review the other personal keys and give them the same per-machine layer, is declined by the writer's own call with no Added after Discussion entry recording the narrowing.
    fix: >-
      Leave the code in place and defer this finding to the run's parent.
      The parent either records the narrowing with lym loom decision add, citing the reasons in the summary's "The other personal keys stay where they are", or refuses it, in which case the per-machine layer for those keys becomes further work.
      Either sound change is the parent's and lies outside the fixer's write surface.
---

### [BLOCKING] A pair made under another machine's prefix is not found or re-adopted

**Location:** `internal/fabricengine/pairbranch.go:20`, `internal/fabricengine/add.go:229`, `internal/battencli/readopt.go:53-80`

**Issue:** The diff makes `branch_prefix` a machine key (`internal/fabricengine/config.go:49`), so two machines of one hub can name pairs under different prefixes.
Every lookup that goes from a slug to the pair's branch still tries only the current machine's prefix and the bare slug (`pairBranchNames`, `pairbranch.go:20-24`).
When nothing of the pair is local, as on a machine that takes over a run, `PairTaskBranchOrOrigin` misses `a/<slug>` on origin and returns `b/<slug>`.
Its callers are the re-adopt seam's `hubTaskBranch` (`readopt.go:53-60`), the owner archive (`ownerarchive.go:192`) and pair teardown's pull-request step (`pairteardown/pullrequest.go:47`).
The re-adopt seam's `Add` with `AdoptOnly` builds `t.cfg.BranchPrefix + slug` (`add.go:229`), finds neither `b/<slug>` nor its records branch, and stops (`add.go:460`, as the correctness lens read it).
The board entry's premise is that "a pair created on one machine keeps its name when adopted on another".
Before this change a mismatch needed an operator to change the hub-wide prefix, but now it follows from every owner move between machines whose prefixes differ.
The change description lists the `PairTaskBranchOrOrigin` gap under "Open risks" and calls it the same as today's, which understates it, and it does not mention `Add` or re-adopt at all.

### [MEDIUM] slugOfBranch derives the wrong slug from another machine's branch

**Location:** `internal/fabricengine/pairbranch.go:29`, callers `internal/fabricengine/pairparts.go:78`, `internal/fabricengine/cleanup.go:265`, `internal/fabricengine/cleanupremotewarp.go:164,215`

**Issue:** `slugOfBranch` trims only the current prefix.
Under prefix `b/`, the branch `a/foo` yields the slug `a/foo`.
`TaskBranchSlugs` then lists `a/foo` to adopt's fix loop (`adoptcli/fixloop.go:326`) and beam's held slugs (`beamcli/deps.go:280`).
The orphan and remote-warp sweeps take `TeardownLockPath(hub, "a/foo")` (`teardownlock.go:43-44`), not the `foo` lock a teardown of that pair holds, so the two are not serialized.
`keptReason` checks archive evidence under the wrong slug; that keeps the branch, so it fails safe.
The diff touches none of these readers.

### [MEDIUM] The review of the other personal keys is declined without a recorded decision

**Location:** `_lym/landing/summary.md` ("The other personal keys stay where they are")

**Issue:** The amended board entry asks for two things: a per-machine or per-user layer for `branch_prefix`, and a review of the other personal keys that gives them the same layer.
The diff builds only the first.
The summary's reasons are accurate: `batten.open_ide` and `landing.pr_reviewer` are category None, and the Slack operator and the pull request's identity are not lym config.
But declining the second goal point narrows the task's scope, and the decision record carries no `Added after Discussion` entry recording that.

## Rejected

- The "When" clause of the board entry, which plans the work after the `adopt-build` beam lands: when a run starts is the dispatcher's call and no change to the diff satisfies it.
- Test economy of the new `TestLoadConfig` subtest: it is the only test that asserts the loaded prefix from a machine override, and the doc-comment line on the local refusal describes the pre-existing loop at `config_test.go:139-165`.
- The repointed `TestDispatchLayers` row: slack's one key is `CategoryNone` (`slackengine/config.go:41-43`), so the hub-wide refusal and its `wantErr` "lym config slack" hold, and fabric's machine path is covered by the generic gate rows.
