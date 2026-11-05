---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/pairbranch_integration_test.go:20
    summary: No test covers a prefix set over a pair with nothing left locally, so the origin lookup in Remove, RemoveRefusal and DiscardTaskBranch and the branch names the remote teardown job carries are unpinned
    fix: In internal/fabricengine/pairbranch_integration_test.go add one test (tiergate 1, parallel, its own slugs) that makes a pair under the empty prefix, pushes its task and records branches to origin, deletes both worktrees and both local branches, then sets branch_prefix to hanf/. Assert that RemoveRefusal passes, that Remove with remote true deletes the bare-slug task and records branches on origin through the job's carried WarpBranch and WeftBranch, and that RemoveResult.SiblingBranch is the bare records name. Add a second pair left the same way and assert DiscardTaskBranch deletes its bare-slug task branch on origin. Add a row where only the records branch is on origin, for originPairBranch.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/pairbranch.go:33
    summary: Nothing pins PairTaskBranch's own rules, namely that a worktree on a branch outside the pair's names is ignored, and its records-worktree step, local-branch step and fall-through to the current prefix name
    fix: In internal/fabricengine/pairbranch_integration_test.go add a table-driven test of Topology.PairTaskBranch with prefix hanf/ set over a pair made under the empty prefix. Rows are a task worktree switched to an unrelated branch (result is never that branch), a detached HEAD in the task worktree, only the records worktree standing, only a local bare-slug branch left, a prefixed branch and a bare branch both local (the prefixed one wins), and nothing left (result is hanf/ plus slug).
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/pairparts.go:74
    summary: The branch-to-slug steps changed behavior with a prefix set and no test runs any of them with a prefix, though the Acceptance names listing and cleanup
    fix: Add a row with branch_prefix set to the existing test of each step. TaskBranchSlugs (pairparts_integration_test.go) gets a bare branch and a prefixed branch that both list their slug. The remote warp sweep test (cleanupremotewarp_integration_test.go) gets a bare branch with weft evidence classed managed and one without evidence kept as not fabric-managed. The cleanup orphan test gets an orphan weft branch of a bare-slug pair swept under the prefix. The test of resolveAddParent gets a --parent checked out on its bare slug under the prefix and accepted.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-affected-set
    location: internal/fabricengine/remove.go:207
    summary: removePair's nothing-local test omits launchers while refuseWhenNothingRemains counts them, so a launcher-only pair costs an ls-remote the refusal probe did not make, and an origin-only pair is asked twice
    fix: In internal/fabricengine/remove.go extract one helper over pairState that answers whether any local part remains (taskWorktree, siblingWorktree, taskBranch, siblingBranch, launchers), and use it both in refuseWhenNothingRemains at line 563 and in the condition at line 207. Make the origin answer found by the refusal travel to removePair, for example by returning it from an unexported variant of RemoveRefusal that removePair calls, so one Remove asks origin at most once. The documented rule that a pair with any local remnant costs no network call then holds.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/pairbranch.go:66
    summary: For a bare-slug pair with only its origin branch left and a prefix set, PairTaskBranch names the current prefix branch, so the pull request warning and batten's re-adoption records-branch lookup use the wrong name
    fix: Add a Topology method that returns the PairTaskBranch answer unless the local read ended at its last fallback, in which case it returns originPairBranch's name when origin holds one. Use it in internal/pairteardown/pullrequest.go:45 and in hubTaskBranch at internal/battencli/readopt.go:55 where the pair may already be gone. Keep PairTaskBranch itself network-free.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/pairbranch.go:44
    summary: PairTaskBranch returns an error when the records worktree directory is present but its repository root cannot be resolved, so inspectPair now fails and a half-torn pair cannot be removed, where removeSibling tolerated that failure
    fix: In internal/fabricengine/pairbranch.go skip the records-worktree step when RecordsRepoRoot fails and fall through to the local-branch step, instead of returning the error. Add a row for it to the table test of F2.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/pairteardown/pullrequest.go:45
    summary: The pull request lookup and batten re-adoption read the branch through PairTaskBranch, and no test sets a prefix over a live bare-slug pair for either, though the Acceptance names PR lookup
    fix: Add a row to the existing battencli re-adoption test that sets branch_prefix over a live pair made under the empty prefix and asserts the TaskBranch and TaskBranchGone seams of hubReadoptSeams answer with the bare slug. For closePairPullRequest add the same prefix-over-live-pair row to the existing pairteardown test that reaches its gone branch, asserting the branch passed on is the bare slug.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/fabricengine/template.yaml:1
    summary: The branch_prefix description still says the prefix forms the branch name of every pair, though the decision record makes it name only the branches of pairs created after it is set
    fix: In internal/fabricengine/template.yaml line 1 reword the trailing comment to say the prefix is prepended to the slug of each new pair, that a pair keeps the branch name it was made with, that the value can be changed while pairs are live, and that empty means branch equals slug.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/fabricengine/destructivegaps_integration_test.go:721
    summary: The helper's comment still names ownedManagedBranch(l, "") though the diff removed that parameter
    fix: In internal/fabricengine/destructivegaps_integration_test.go line 721 change ownedManagedBranch(l, "") to ownedManagedBranch(l) in the helper's doc comment.
  - id: F10
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/fabricengine/remoteteardown.go:62
    summary: One field comment above WeftBranch describes both branch fields while WarpBranch sits six lines later with no comment
    fix: In internal/fabricengine/remoteteardown.go narrow the comment above WeftBranch to the records branch as the command read it from the pair, since the pair's worktrees are gone when the job runs, and give WarpBranch its own comment above its declaration saying the same of the task branch.
---

### [MEDIUM] No test of a prefix set over a pair with nothing left locally

**Location:** internal/fabricengine/pairbranch_integration_test.go:20
**Issue:** `TestBranchPrefixChange_KeepsLivePairsManaged` keeps the local branches, so `originPairBranch`, `firstOnOrigin`, the origin fallback in `DiscardTaskBranch` and the branch names the remote teardown job carries never run under a changed prefix. The Acceptance names remove and teardown; only the local-remnant path is pinned.

### [MEDIUM] PairTaskBranch's own rules are untested

**Location:** internal/fabricengine/pairbranch.go:33
**Issue:** The decision record's safety rule, that a worktree checked out on an unrelated branch is never read, has no row. The records-worktree step, the local-branch step and the fall-through to the current prefix name have none either, because the one test leaves both worktrees standing and the first step always answers.

### [MEDIUM] Branch-to-slug steps are untested with a prefix set

**Location:** internal/fabricengine/pairparts.go:74
**Issue:** `TaskBranchSlugs`, the `keptReason` and `cleanupOrphan` slug derivations and the `--parent` check of `Add` changed behavior when a prefix is set. Their tests use the empty prefix, so the changed behavior, and the dropped refusal of a branch lacking the prefix, is asserted nowhere.

### [LOW] Remove asks origin where the refusal probe did not, and twice for an origin-only pair

**Location:** internal/fabricengine/remove.go:207
**Issue:** The condition in `removePair` leaves out `launchers`, which `refuseWhenNothingRemains` counts as a local part. A launcher-only pair therefore makes an `ls-remote` the probe skipped, against the doc claim that a pair with any local remnant costs no network call. An origin-only pair is asked once in the refusal and again in `removePair`.

### [LOW] An origin-only bare-slug pair is named with the current prefix

**Location:** internal/fabricengine/pairbranch.go:66
**Issue:** `PairTaskBranch` is network-free and ends at the current prefix's name, so with only an origin branch left the callers that run after the local parts are gone get the wrong name. The torn-down pull request path calls no GitHub API in that state, so only its warning names the wrong branch. Batten's re-adoption builds the records branch name from it and looks up `hanf/<slug>-weft` instead of `<slug>-weft`.

### [LOW] PairTaskBranch errors on an unresolvable records repository

**Location:** internal/fabricengine/pairbranch.go:44
**Issue:** `inspectPair` now calls `PairTaskBranch`, which returns the `RecordsRepoRoot` failure whenever a records worktree directory exists. `RemoveRefusal`, `Remove`, `PairParts` and `DiscardTaskBranch` then all fail for a half-torn pair. Before the diff `inspectPair` never resolved that root, and `removeSibling` tolerated the failure.

### [LOW] Pull request lookup and re-adoption have no prefix test

**Location:** internal/pairteardown/pullrequest.go:45
**Issue:** The branch these two read changed from `cfg.BranchPrefix + slug` to `PairTaskBranch`. The only prefix-bearing re-adoption test sets its prefix from the start with no live bare-slug pair, and no pairteardown test asserts the branch the gone path passes on.

### [LOW] branch_prefix template description is stale

**Location:** internal/fabricengine/template.yaml:1
**Issue:** `doc.go` was updated, but the config template, where an operator reads what `branch_prefix` means, still says the prefix forms the branch name and does not say a pair keeps its branch name or that the value can change with pairs live.

### [LOW] Stale signature in a test helper comment

**Location:** internal/fabricengine/destructivegaps_integration_test.go:721
**Issue:** The comment names `ownedManagedBranch(l, "")`, a signature the diff removed.

### [NIT] Field comment covers a field declared elsewhere

**Location:** internal/fabricengine/remoteteardown.go:62
**Issue:** The comment above `WeftBranch` says "WeftBranch and WarpBranch", but `WarpBranch` is declared after `DeleteWeftOnOrigin` with no comment of its own.

## Rejected

- Fork finding that `slugOfBranch` and `TaskBranchSlugs` now class a branch lacking the prefix, such as `main`, as a slug (lens:code-correctness): the decision record's Chosen fix says these steps accept a branch with the current prefix and one without. `keptReason` still needs weft evidence before a branch is managed, the destructive gate still bounds a deletion, and the only consumer of the listing numbers past an extra branch.
- Fork note that `RemoveResult.SiblingBranch` is set when no sibling existed (lens:code-correctness): the field is read only when the origin delete failed, so no behavior differs.
- Own check of the `RemoveResult.SiblingBranch` field and the `fabriccli` change as unlicensed scope: the record's Chosen fix removes every re-derivation of the branch from the current config on a live-pair path, and the `fabriccli` error rebuilt the name that way.
- Own check of a `Pattern-Departure:` need: none of the four commits touches a PATTERN rule, and `PATTERN.md` and `pattern/` declare no change for this diff.
- Fork check of test economy: `pairbranch_integration_test.go` repeats no existing test, and the signature-only edits to existing tests pin nothing new, so no redundancy finding stands.
- Own check of the summary's claim that origin is asked only when nothing local names the branch: it holds except for the launcher case, which is finding F4.
