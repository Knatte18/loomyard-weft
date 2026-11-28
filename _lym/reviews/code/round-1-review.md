---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: handler
    location: contracts/stencils/burler/burler-cluster-fan.md:22
    summary: "Both fork briefs narrow the base's ban on writing, editing or deleting any file to files of the target, so a fork may now change any non-target file by shell, such as another worktree's files or a global git config; the throwaway allowance needs only state changes inside repositories the fork creates."
    fix: "In burler-cluster-fan.md:22 and burler-fix-self-check.md:45, replace \"never write, edit or delete a file of the target\" with \"never write, edit or delete a file outside a throwaway repository you created as below\". Keep \"never use a write or edit tool\" and the rest of each sentence as they stand."
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: handler
    location: contracts/stencils/burler/burler-cluster-fan.md:22
    summary: "The briefs allow a throwaway repository created with git worktree add without saying it must be added from another throwaway repository. The audit records a worktree only when it is added from a created repository (cluster.go:274-284). A worktree added from the target also registers itself in the target's .git, and every later commit in it counts as ruleOutsideCreated."
    fix: "In burler-cluster-fan.md:22 and burler-fix-self-check.md:46, change \"with `git init`, `git clone` or `git worktree add`\" to \"with `git init`, `git clone`, or `git worktree add` run from such a throwaway repository\"."
  - id: F3
    severity: LOW
    class: consistency
    origin: handler
    location: contracts/stencils/burler/burler-fix-self-check.md:48
    summary: "The briefs name ln and the redirecting config keys as making every later push count, but not the audit's third such trigger: a git remote change whose directory the audit cannot resolve, such as one run under --git-dir or --work-tree (cluster.go:394-396). A fork may run that on its own throwaway repository and halt the run with its next push."
    fix: "In burler-fix-self-check.md:48 and the matching clause of burler-cluster-fan.md:22, change \"after any `ln`, or any git config key starting ...\" to \"after any `ln`, any `git remote` call run with `--git-dir` or `--work-tree`, or any git config key starting ...\", leaving the key list unchanged."
  - id: F4
    severity: LOW
    class: design
    origin: handler
    location: internal/burlerengine/cluster.go:945
    summary: "On a review round, the revert way forward is now returned for every counted command outside a created repository, including a commit, a branch, a tag or a push from the target. For those it says to inspect the target's working-tree changes, but a commit or ref change leaves no working-tree change to inspect, so the reader is not told to check HEAD and the refs."
    fix: "In violationWayForward's default return at cluster.go:945, word the way forward as \"way forward: inspect the target's working tree, HEAD, branches and tags for what the fork changed, undo it, and resume the run\". Update the revertWayForward constant in cluster_test.go:30 to the same text. Name no denied form, since PATTERN-refusal-way-forward lists git reset --hard."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/burlerengine/cluster.go:941
    summary: "The untouched-push way forward, now reached from the review path, states that no fork wrote a file or ran a state-changing git command in the target's working tree. targetUntouched sees only write and edit tool calls and git calls outside created repositories, not Bash writes such as mkdir or ln. In the incident row (cluster_test.go:318) those ran under /wt/.scratch, a path inside the worktree."
    fix: "At cluster.go:941, replace \"no fork wrote a file or ran a state-changing git command in the target's working tree\" with \"the audit found no write or edit call and no state-changing git command outside a repository a fork created\". Update the matching expected strings in cluster_test.go:33 and cluster_test.go:325."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/burlerengine/cluster.go:338
    summary: "An scp-style push destination with no slash, such as host:repo, fails looksLikePath. It is named under rulePushToRemoteOutside (a remote the fork did not point) instead of rulePushToPathOutside (a URL or path), because isGitURL is tested only after the !looksLikePath return."
    fix: "In pushMutates, test isGitURL(destination) first and return rulePushToPathOutside, before the !looksLikePath branch at cluster.go:338. Add the row {\"push to an scp-style location\", []string{\"git init /tmp/r\", \"git -C /tmp/r push host:repo main\"}, rulePushToPathOutside} to TestMutatingGitCommandsNameTheBrokenRule."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/burlerengine/cluster_test.go#TestMutatingGitCommandsNameTheBrokenRule
    summary: "No row has a command with more than one counted invocation. If runsMutatingGit kept the last counted invocation's rule rather than the first, which its doc and the rule field's doc both state, every test would still pass."
    fix: "Add the row {\"the first counted invocation names the rule\", []string{\"git commit -am x && git init /tmp/r && git -C /tmp/r push\"}, ruleOutsideCreated} to TestMutatingGitCommandsNameTheBrokenRule. Its later push alone would name rulePushNoDestination."
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: _lym/landing/summary.md
    summary: "The change description's Tests section claims \"Every row's way forward is checked on both the review and the fix paths\". Each row runs one path, chosen by tt.fix (cluster_test.go:355-363), and the review-path nested-spawn, write and named-spawn rows set no wantWayForward."
    fix: "Reword the claim in summary.md's Tests section to \"The way-forward check now runs on review-path rows as well as fix-path rows; the review-path commit and ln rows assert it.\""
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/burlerengine/cluster.go:128
    summary: "The mutatingGitCommand type doc still says the invocation runs outside the repositories the fork created. A push counted after ln, a redirect key or an unresolved remote change runs from a created repository, as the type's own pushFrom field documents. The mutatingGitCommands doc at cluster.go:140 also says the command runs a mutating subcommand on a repository the fork did not create."
    fix: "Reword cluster.go:128 to \"The invocation may change state in a repository the fork did not create.\" Reword cluster.go:140 to \"It returns, in order, each command that runs a git invocation that may change state in a repository the fork did not create.\""
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/burlerengine/doc.go:253
    summary: "The package doc's fork-discipline lead still says \"no write of any file; no git command that changes the repository, index, refs or working tree\". The edited section below it and both briefs now allow state changes in a throwaway repository the fork created."
    fix: "Replace doc.go:253-254's two clauses with \"no write or edit tool call, and no write to a file outside a repository the fork created;\" and \"no git command that may change a repository the fork did not create;\", keeping the lines that follow."
  - id: F11
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/burlerengine/cluster_test.go:29
    summary: "The doc comments on revertWayForward and untouchedPushWayForward still call them the fix-round halt's way forward, though a review-path row now asserts revertWayForward and both paths produce the same text."
    fix: "In cluster_test.go:29 and cluster_test.go:32, change \"the fix-round halt's way forward\" to \"a fork-audit halt's way forward\"."
---

### [MEDIUM] The briefs' file ban is narrowed beyond what the throwaway allowance needs

**Location:** `contracts/stencils/burler/burler-cluster-fan.md:22`, `contracts/stencils/burler/burler-fix-self-check.md:45`

**Issue:** At the base, both briefs said never to write, edit or delete any file.
The rewrite says never to write, edit or delete "a file of the target".
Stating the throwaway allowance needs only permission to change state inside repositories the fork creates.
The narrowed wording also permits shell writes to every non-target file: another worktree of the hub, the operator's home, a global git config.
The audit never sees a shell write, so the brief is the only guard on those.
The board entry asked to state the allowance with its limits, not to widen what a fork may write.

### [MEDIUM] `git worktree add` is allowed without saying where from

**Location:** `contracts/stencils/burler/burler-cluster-fan.md:22`, `contracts/stencils/burler/burler-fix-self-check.md:46`

**Issue:** The audit treats a worktree as fork-created only when `git worktree add` runs inside a repository the fork created (`cluster.go:274-284`).
The brief allows "a throwaway repository you create outside the target with ... `git worktree add`" and never says it must come from a throwaway.
A fork that adds a worktree from the target changes the target's `.git/worktrees` and gets an unrecorded worktree.
Its first commit there halts the run as a state-changing command outside every created repository.
The earlier clause, which limits the target to reading git commands, arguably forbids this.
The allowance sentence still reads as permitting it.

### [LOW] The briefs omit the unresolved-remote-change push trigger

**Location:** `contracts/stencils/burler/burler-fix-self-check.md:48`

**Issue:** `recordRemote` makes every later push count after a `git remote` change whose directory it cannot resolve, such as one under `--git-dir` (`cluster.go:394-396`).
The briefs list `ln` and the redirecting config keys, but not this trigger.
A fork can hit it on its own throwaway repository without breaking any rule it was told.

### [LOW] On a review round, the revert way forward misses commits and ref changes

**Location:** `internal/burlerengine/cluster.go:945`

**Issue:** The review path now ends every touched-target violation with "inspect the target's working-tree changes, revert what the fork wrote".
On a review round, the typical counted command is `git commit`, as in the row at `cluster_test.go:300-311`, or a branch, a tag or a push.
None of those leaves a working-tree change, so the way forward leaves out what actually changed: HEAD and the refs.

### [LOW] The untouched-push way forward claims more than the audit can see

**Location:** `internal/burlerengine/cluster.go:941`

**Issue:** `targetUntouched` counts write and edit tool calls and git calls outside created repositories (`cluster.go:900-911`).
The message says no fork wrote a file "in the target's working tree", which a Bash `mkdir`, `ln` or redirect can falsify unseen.
The regression row's commands run under `/wt/.scratch`, inside the worktree path.

### [LOW] An scp-style destination is named as a remote

**Location:** `internal/burlerengine/cluster.go:338`

**Issue:** `host:repo` holds no `/` and opens with no `.`, so it fails `looksLikePath` and returns `rulePushToRemoteOutside` before `isGitURL` is tested.
The command still counts; only the rule named in the halt is wrong.

### [LOW] No test pins that the first counted invocation names the rule

**Location:** `internal/burlerengine/cluster_test.go#TestMutatingGitCommandsNameTheBrokenRule`

**Issue:** `runsMutatingGit` keeps the first counted invocation's rule (`cluster.go:235-237`), as its doc and the `rule` field's doc state.
Every row in the rule test counts at most one invocation per command, and the multi-invocation row at `cluster_test.go:132-141` asserts only "git-mutating command".

### [LOW] The change description overstates the way-forward test coverage

**Location:** `_lym/landing/summary.md`

**Issue:** It says every row's way forward is checked on both paths.
Each row runs one path (`cluster_test.go:355-363`).
The review-path rows for a nested spawn, a write and a named spawn (`cluster_test.go:276-298`, `327-336`) assert no way forward.
No row is checked more loosely than at the base.
At the base, the suffix check ran only on fix rows with `wantErr`, and every such row still sets `wantWayForward`.

### [LOW] `mutatingGitCommand` and `mutatingGitCommands` docs say the command runs outside created repositories

**Location:** `internal/burlerengine/cluster.go:128`

**Issue:** The incident shows a push from a created repository being counted.
The type's own `pushFrom` field documents that case, and the new violation message reads "may change a repository it did not create".
The two unedited docs still say the invocation runs outside, or on, a repository the fork did not create.

### [LOW] The package doc's fork-discipline lead contradicts the new allowance

**Location:** `internal/burlerengine/doc.go:253`

**Issue:** The lead still forbids any file write and any state-changing git command.
The section's own later lines (`doc.go:266-267`) and both rewritten briefs allow both inside a throwaway repository.

### [NIT] Test constants still describe themselves as fix-round only

**Location:** `internal/burlerengine/cluster_test.go:29`

**Issue:** `revertWayForward` is now asserted on a review-path row, and the review path produces the same text.

## Rejected

- The code-scope lens raised no finding. Its trace holds: the rule naming and the review-path way forward build the first ask, the two brief rewrites build the second, and the third ask, which only asked for consideration, is answered in the change description's design decisions.

## Focus verdicts

- The briefs against the audit: a throwaway under `.scratch` inside the working tree is forbidden by the brief ("outside the target") but allowed by the audit. A stricter brief is harmless, so that is no finding. `git remote add` before a push is allowed by both and leads in when it points into a created repository. Every push-rule constant except the unresolved-remote trigger (F3) is either stated or follows from "only to another such repository, by its path or by a remote you pointed there". `git worktree add` is F2.
- The READ-ONLY sentences: the git-command prohibition is kept for the target and every repository the fork did not create. The file prohibition is narrowed (F1).
- The review-path way forward: the error is wrapped with `%w`, so `*forkViolationError` stays in the chain. No caller classifies it, and shedadapters still classifies only `ErrClusterForksMissing` and `ErrHalfNotStopped` (`burlerreview.go:458-466`, `655-668`). The halt classification is unchanged. The wording is F4 and F5.
- Rule attribution: the first redirecting call wins across calls and across commands. A single call that names a key and changes a remote names the key rule, since the key is checked before `recordRemote`. Within one command, the first counted invocation names the rule, which is untested (F7). The scp-style label is wrong (F6).
- Doc comments: F9 and F10.
- `TestAuditClusterRound`: no base row is checked more loosely. The description's claim is F8.
- Merge danger: the diff changes only an error message and two briefs. No other binary parses that message, no config meaning changes, and nothing is deleted or pushed. Two-way with blast radius burler holds.
