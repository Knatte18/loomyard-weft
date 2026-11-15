---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: internal/battencli/drive.go:543
    summary: Batten's arbiter seam reads and commits in the hub's prime, whose HEAD and worktree other writers change, so readEdits can report another writer's commits as the arbiter's and ask the parent to keep or revert them
    fix: Leave the wiring in place and defer this finding to the run's parent, who records with lym loom decision add which worktree batten's arbiter edits and commits in. Batten's location is the prime (drive.go:244), so the seam and the prompt's Worktree name the prime and not the task worktree that taskWorktreeLocation(location, slug) returns. Once the parent records the choice, wire shedworktree.ArbiterSeam and PromptInput.Worktree over the worktree it names.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: internal/battencli/drive.go:59
    summary: Batten's arbiter recipe text and docs/overview.md still say batten's arbiter cannot fix, though the diff now gives it the worktree seam and the fix fragment
    fix: In internal/battencli/drive.go, change the last sentence of battenArbiterRecipe to say the arbiter may make one fix of its own and commit it, and otherwise re-launch, goto the halted row or hand on. In docs/overview.md:1545-1546, replace "The arbiter of a loom-engine recipe may make one fix" with "The arbiter may make one fix", and delete "; batten's arbiter edits nothing". In internal/loomcli/doc.go:471, drop "of a loom-engine recipe".
  - id: F3
    severity: MEDIUM
    class: consistency
    location: contracts/stencils/loom/loom-template-arbiter.md:59
    summary: In fix mode the template's What you may change section still limits the arbiter to the repair verbs and nothing else, and to a re-launch, a goto or a hand-on when the recipe names none, which contradicts the fix fragment below it
    fix: In loom-template-arbiter.md, change line 59 to "Run in `{{.worktree}}` only the repair verbs the run's description above names, and what the edit rules below let you." Change line 60 to "A run whose description names none leaves you a plain re-launch, a goto or a hand-on, after any fix the edit rules below let you make." In line 66, change "That is the one edit you make, and nothing beyond it." to "That is the one decision edit you make; the edit rules below govern any other."
  - id: F4
    severity: LOW
    class: consistency
    location: _lym/landing/summary.md:1498
    summary: The change description is stale and claims batten leaves the seam unset, that the spawn snapshot covers only tracked paths, and that the seam has three reads
    fix: In _lym/landing/summary.md, say batten's arbiter now gets the same seam and fix rules, per the Added after Discussion decision. Add Changes, which hashes every uncommitted path including untracked files, to the seam's reads and to what attempt.json records. Name shedworktree as the shared package. In the Tests section, add the untracked-file, read-failure and spawn-time Changes rows of TestCall_Edits and TestHashFileOrDeleted.
  - id: F5
    severity: LOW
    class: scope
    location: internal/shedworktree/shedworktree.go:33
    summary: ArbiterSeam's Changes read has no coverage, so nothing tests that it lists an untracked file, keys by the worktree-relative path or hashes the file joined under the worktree
    fix: Add a test in internal/shedworktree/shedworktree_test.go over a hermetic git pair from the repo's test kit, at the lowest tier that allows git. Have it write an untracked file and modify a tracked file, call ArbiterSeam(location).Changes(), and assert the exact map from slash-separated relative path to SHA-256, with deleted for a removed tracked file. Fold TestHashFileOrDeleted's rows into it.
  - id: F6
    severity: LOW
    class: scope
    location: internal/shedarbiter/call_test.go:584
    summary: fakeWorktree ignores the arguments of IsAncestor and CommitLog, so no test pins that readEdits asks whether the spawn HEAD is an ancestor of the current HEAD and logs from spawn to current
    fix: In fakeWorktree, record the (sha, ref) pair IsAncestor receives and the (from, to) pair CommitLog receives. In TestCall_Edits, for the rows whose after is moved, assert both pairs equal ("aaa", "bbb").
---

### [MEDIUM] Batten's seam is over the prime, which other writers change

**Location:** internal/battencli/drive.go:543

**Issue:** `loadedArbiterDeps` builds the seam from `c.location`, which is the hub's prime location (drive.go:244: "Every path comes from shedrun over the prime's location"), and the prompt's `Worktree` is `location.WorktreePath()`, also the prime.
`readEdits` treats every commit between the spawn HEAD and the HEAD after the arbiter as the arbiter's, and every new uncommitted path as the arbiter's leftover.
On the prime, another batten run's Land pulls the prime, and other sessions may leave files there.
A HEAD that moves forward during an arbiter attempt then lands in `Commits`, forces a goto, and goes into the hand-on question as commits for the parent to "keep or revert".
A stray file from another writer hands on a sound resume.
The arbiter also commits on the prime's branch, whereas loom's arbiter commits on the task branch.
The decision record gives batten "the same worktree seam" but does not say which worktree that seam is over, so this is a choice for the parent, not the fixer.

### [MEDIUM] Batten's recipe text and the overview still say its arbiter cannot fix

**Location:** internal/battencli/drive.go:59

**Issue:** `battenArbiterRecipe` ends "The arbiter's only moves are a plain re-launch, a goto to the halted row, or a hand-on."
That text fills `{{.recipe}}` in the same prompt that now carries "## Your one fix", so batten's arbiter reads two contradictory rules.
`docs/overview.md:1546` says "batten's arbiter edits nothing", and line 1545 together with `internal/loomcli/doc.go:471` limits the fix to a loom-engine recipe.
Both contradict the Added after Discussion decision and the wiring in this diff.

### [MEDIUM] The template's What you may change section contradicts the fix fragment

**Location:** contracts/stencils/loom/loom-template-arbiter.md:59

**Issue:** Line 59 says to run "only the repair verbs the run's description above names, and nothing else".
Line 60 leaves a run with no repair verbs "a plain re-launch, a goto or a hand-on".
Line 66 says the decision edit "is the one edit you make, and nothing beyond it".
The fix fragment filled at line 79 then permits code edits, `git add` and `git commit`.
In fix mode the arbiter reads both rules, and the earlier one forbids the fix.
Line 67 and line 113 were updated to defer to the edit rules, but these three lines were not.

### [LOW] The change description is stale

**Location:** _lym/landing/summary.md:1498

**Issue:** The description says "Batten leaves it unset, since its worktree is the prime … its arbiter edits nothing, as before".
After commit 54cdd04d, batten sets the seam.
It describes the seam as "a worktree snapshot, an ancestor check and a commit log", leaving out `Changes`.
It says the snapshot records the "hash of each tracked uncommitted path", but attempt.json now also records `Changes`, which includes untracked files.
It does not mention `internal/shedworktree`.

### [LOW] ArbiterSeam's Changes read is untested

**Location:** internal/shedworktree/shedworktree.go:33

**Issue:** This read is how an untracked file the arbiter leaves gets caught, which is round 2's first focus.
Only `hashFileOrDeleted` is tested.
No test drives `Changes` over a real checkout, so nothing pins the inclusion of untracked files, the relative-path keys that `readEdits` compares with `attempt.Changes`, or the join under `location.WorktreePath()`.
`TestCall_Edits` uses a fake seam and cannot see any of it.

### [LOW] The arguments of the ancestry and log reads are not pinned

**Location:** internal/shedarbiter/call_test.go:584

**Issue:** `fakeWorktree.seam` returns `w.descends` and `w.commits` whatever arguments it gets.
If `readEdits` swapped `IsAncestor(spawned.Head, current.Head)` or `CommitLog(spawned.Head, current.Head)`, the swap would invert the rewritten-history check or empty the commit list against real git, and every row would still pass.

## Focus answers

- Untracked files (focus 1): covered.
  `WorktreeSeam.Changes` reads `fabricengine.CodeWorktreeChanges`, which goes through go-git status and lists untracked, non-ignored files one by one, not as collapsed directories.
  The call records it at spawn (`attempt.Changes`), and `readEdits` compares it again after the arbiter, so a new or edited untracked file hands a resume on.
  A committed new file shows up as a commit through `CommitLog`.
  Gitignored files are not seen, which matches the dirty guard's scope.
  F5 covers the missing coverage of the real read.
- TestCall_Edits read failures (focus 2): each post-arbiter failure has its own row, separate from the spawn-time rows.
  Those failures are the snapshot read, the Changes read, IsAncestor and CommitLog; the spawn-time rows cover `snapshotErr` and `spawnChangesErr`.
  The IsAncestor and CommitLog rows expect the reason "HEAD moved from aaa to bbb and the arbiter's commits could not be read", which names both the failure and the move.
  The post-arbiter snapshot failure cannot know HEAD, and its reason names the failed read.
  The post-arbiter Changes failure row keeps HEAD unmoved.
  This part is not a defect; F6 is a separate gap.
- Fix-mode git rule (focus 3): the fix fragment allows only `git add` and `git commit` on the found branch and forbids every other git command that changes the repository, its index, its refs or its worktree.
  The template adds the push, amend, rebase, reset and branch-switch ban.
  The git rule holds; F3 is a contradiction in the template's earlier section, not in the git rule.

The gate ran green on every changed package: tier 0, 3095 passed, 0 failed.
