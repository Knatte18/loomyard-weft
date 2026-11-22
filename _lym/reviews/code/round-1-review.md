---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/burlerengine/cluster.go#recordRemote
    summary: "The per-root remote model marks a remote as leading into a fork repository while git pushes elsewhere: set-url --push is not told apart, a plain set-url turns an outside push URL back to safe, a worktree's remote change is keyed under the worktree instead of the shared config, and a git remote call under --git-dir, --work-tree or an unknown directory is ignored."
    fix: "In internal/burlerengine/cluster.go, make remote tracking conservative. (1) Once a remote of a root is marked as not leading in, keep it so: in setRemote never turn false into true, so a plain set-url after a set-url --push outside stays unsafe. (2) In recordRemote, treat set-url --push (and any set-url carrying --push) as setting the remote to leadsIn && the remote's current value. (3) When git remote add/set-url/rename/remove runs with an unknown directory (invocation.dirKnown false, which covers --git-dir and --work-tree) set repos.pushRedirected = true. (4) Key remotes by the repository whose config a worktree shares: record, in record() for a worktree add, the source root the worktree came from, and have recordRemote and pushMutates use that source root as the remotes key. Add rows to TestFirstMutatingGitCommandScopesToForkRepositories for each: set-url --push outside then plain set-url inside then push origin; remote set-url outside run from a worktree of the pushing repository; git --git-dir=/tmp/r/.git remote set-url origin /srv/target.git then git -C /tmp/r push origin; each must return the push command."
    origin: lens:code-correctness
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/burlerengine/cluster.go:451
    summary: "The redirect check misses git's documented --config-env=<name>=<envvar> spelling, so a push whose remote or url. key is overridden that way passes; an include.path or includeIf key that pulls in a redirecting config is not flagged either."
    fix: "In parseGitInvocation, before the gitValuedGlobalOptions case, match a word with the prefix --config-env= and run namesPushRedirectKey on the part after it, setting invocation.redirectsPush; keep the separate-word form as is. Add include. and includeif. to gitPushRedirectKeyPrefixes. Add rows to TestFirstMutatingGitCommandScopesToForkRepositories: a push to a recorded leading-in origin under git -C /tmp/r --config-env=remote.origin.pushurl=E push origin main, and one under -c include.path=/tmp/x; both must return the push command."
    origin: lens:code-correctness
  - id: F3
    severity: BLOCKING
    class: design
    location: internal/burlerengine/cluster.go#leadsInto
    summary: "A relative push destination or remote URL is resolved against the directory the git call ran in, but git resolves it from the repository's top level, so git -C r/x push .. passes as a push into r while git pushes into the directory above r, which can be the target."
    fix: "In leadsInto, resolve a relative p against the created root that contains invocation.dir (repos.rootOf(invocation)), not against invocation.dir; when invocation.dir lies in no created root, a relative p leads nowhere. recordRemote and pushMutates then inherit the fix through leadsInto. Add rows to TestFirstMutatingGitCommandScopesToForkRepositories: git init r && mkdir r/x && git -C r/x push .. HEAD:refs/heads/x, and git init r && git -C r/x remote add origin .. && git -C r push origin HEAD; both must return the push command, while git -C r push ../remote.git after git init --bare remote.git still passes."
    origin: lens:code-correctness
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/burlerengine/cluster.go#forkViolation
    summary: "The fix-round halt claims more than the audit knows: a push from a fork repository to a destination the audit could not place yields 'no fork ... changed the target's repository', which is false when that destination is the target, and when a non-git violation comes first the push is dropped and the text says no state-changing git command ran at all."
    fix: "In forkViolation, compute pushFrom over every fork's mutatingGitCommands, not only from the git violation, so any untouched audit with a push carries the first push's source. In fixViolationWayForward, change the push form to say that no fork wrote a file or ran a state-changing git command in the target's working tree, that the push from <pushFrom> went to a destination the audit could not place, and that the way forward is to check that destination is neither the target's repository nor one of its remotes, then resume the run; drop the claim that the target's repository is unchanged. Update the TestAuditClusterRound push row's wantWayForward to the new text and add a row with a nested spawn in fork a and git init /tmp/r && git -C /tmp/r push origin main in fork b, expecting the push form."
    origin: lens:code-correctness
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/burlerengine/cluster.go:210
    summary: "For git clone, namesPushRedirectKey scans every argument, so cloning a source or into a directory named remote.git, url.x or branch.x sets pushRedirected and every later push counts, which halts the issue's own scratch probe when it clones its bare remote.git."
    fix: "In gitInvocationMutates, for subcommand clone, run namesPushRedirectKey only on the values of -c and --config (the word after a separate -c or --config, and the part after --config=), not on the source or directory operands. Add a row to TestFirstMutatingGitCommandScopesToForkRepositories: cd /tmp/p && git init --bare remote.git && git clone remote.git work && git -C work commit --allow-empty -m x && git -C work push origin HEAD, which must return no command, and one with git clone -c remote.origin.pushurl=/srv/t.git remote.git work then the same push, which must return it."
    origin: lens:code-correctness
  - id: F6
    severity: MEDIUM
    class: design
    location: internal/shedadapters/burlerfix.go#archiveReport
    summary: "A failure exit that finds no fixer report records nothing, so a later call over the same review either tells its fixer nothing although earlier sessions worked the review, or replays an older call's halt reason and report as if it were the latest."
    fix: "In archiveReport, record the rerun even when archived is \"\": keep the reason (or the cancellation reason) and set Report to the report of the record already kept for the same digest, or \"\" when there is none, so recordRerun always holds the latest halt. Apply the same rule in Call's attempt loop: when attempt 2 archives nothing and rerun came from recordedRerun, replace its Reason with the died-attempt reason while keeping its Report. Add rows to TestBurlerFixProducer_Call_TellsARerunFixerWhy: two died attempts without a report, then a second Call, whose first Fix must get a non-empty Reason naming the two died outcomes and an empty Report; and a halted call with a report followed by a halted call without one, then a third call that must get the second halt's reason with the first call's archived report."
    origin: lens:code-correctness
  - id: F7
    severity: MEDIUM
    class: design
    location: internal/shedadapters/burlerfix_test.go#TestBurlerFixProducer_Call_TellsARerunFixerWhy
    summary: "No test observes the pre-attempt archive's record (burlerfix.go:287-296): mutation survivors at :295 and :296 show that dropping or negating recordRerun there passes every test, and the attempt-1 leftover-report reason is never asserted."
    fix: "Add a row to TestBurlerFixProducer_Call_TellsARerunFixerWhy where attempt 1 writes its report and dies, and attempt 2 dies without writing one, so the call ends on the two-died exit; then make a second Call and assert its first Fix gets Reason \"attempt 1's session ended with outcome \\\"died\\\" before the round finished\" with Report the stamped archive path (or the reason F6's fix gives, keeping that Report). Add a row that plants the canonical round-1 fixer report after acceptance with a not-complete round shape the producer still fixes, if one exists, asserting Reason \"that session never finished the round\"; if none exists, remove that switch case instead."
    origin: lens:code-tests
  - id: F8
    severity: MEDIUM
    class: design
    location: internal/burlerengine/cluster.go#pushMutates
    summary: "The push exemption trusts lexical paths only: a symlink made by ln inside a fork repository can point at the target's repository or remote, and a clone or init recorded at an anchored path (a failed clone into an existing directory, or an ancestor of the target) makes pushes and commits there pass."
    fix: "In runsMutatingGit, treat a segment whose first word is ln (bare or path-prefixed) as able to redirect a push: set repos.pushRedirected = true. In record, refuse an anchored operand that is an ancestor of, or equal to, a directory an earlier cd in the transcript entered without having created it, and refuse a clone directory that the transcript already named as an existing directory (cd or -C into it before the clone). Add the symlink limit to the summary's Known gaps alongside the lexical note. Add rows: git init --bare /tmp/b.git && ln -s /srv/target.git /tmp/b.git/t && git init /tmp/r && git -C /tmp/r push /tmp/b.git/t main, which must return the push command."
    origin: assist:assist
  - id: F9
    severity: LOW
    class: design
    location: internal/burlerengine/cluster.go#pushMutates
    summary: "A push from outside every fork repository to a path inside one passes even with -u or --set-upstream, which writes branch.<name> keys into the pushing repository's config, the target's when the fork pushes from the target."
    fix: "In pushMutates, when from is \"\" and invocation.args holds -u, --set-upstream or --recurse-submodules (with or without =value), return mutates true. Add a row to TestFirstMutatingGitCommandScopesToForkRepositories: git init --bare /tmp/b.git, then git push -u /tmp/b.git main from the starting directory, which must return the push command."
    origin: lens:code-correctness
  - id: F10
    severity: LOW
    class: design
    location: internal/shedadapters/burlerfix_test.go#TestBurlerFixProducer_Call_Cancellation
    summary: "The cancellation reason archiveReport records (burlerfix.go:145, a mutation survivor) is never asserted; the cancellation row checks only the archive count."
    fix: "In the row 'during a failing attempt the report is archived', after Call returns, read the round-1 snapshot with readRoundSnapshot and assert FixerRerun.Reason equals \"the run was cancelled while the fixer ran\" and FixerRerun.Report is the stamped archive path filepath.Join(runDir, \"round-1-fixer-report-20260816T151326Z.md\")."
    origin: lens:code-tests
  - id: F11
    severity: LOW
    class: design
    location: internal/burlerengine/cluster_test.go#TestFirstMutatingGitCommandScopesToForkRepositories
    summary: "Several guard branches the diff adds have no row: innermost-root choice for nested fork repositories, a branch. redirect key, git clone -c or --config with a redirect key, the --name=value form of namesPushRedirectKey, attached --origin=<name>, git remote remove or rm, and set-url --delete."
    fix: "Add one row per branch to TestFirstMutatingGitCommandScopesToForkRepositories: git init /tmp/r && git init --bare /tmp/b.git && git clone /tmp/b.git /tmp/r/sub && git -C /tmp/r/sub push origin HEAD (passes); git -C /tmp/r config branch.main.remote x before a path push (counts); git clone --config=url.x.insteadOf=y /tmp/b.git /tmp/w then a path push (counts); git clone --origin=up /tmp/b.git /tmp/w && git -C /tmp/w push up HEAD (passes); remote add then git remote remove origin then push origin (counts); remote add inside then set-url --delete origin then push origin (counts)."
    origin: lens:code-tests
  - id: F12
    severity: LOW
    class: design
    location: internal/shedadapters/burlerfix.go#archiveWholeRound
    summary: "Retiring a round's review, fixer report and acceptance keeps the snapshot's fixer_rerun record, so a later accepted review of the same round with identical bytes is told the retired review's halt and report."
    fix: "In archiveWholeRound, after archiving the files, clear the record with updateRoundSnapshot(p.runDir, round, func(rec *roundSnapshot, _ bool) { rec.FixerRerun = nil }), logging a failed write like recordRerun does. Extend TestBurlerFixProducer_Call_GateFailedAndChangedReviewArchiveTheWholeRound to plant a FixerRerun record first and assert it is nil afterwards."
    origin: assist:assist
  - id: F13
    severity: LOW
    class: consistency
    location: _lym/landing/summary.md:84
    summary: "The Merge danger section calls a two-way door with blast radius burler, but the diff adds the fixer_rerun key to the tracked round snapshot record that other binaries read, which the rubric names as a change a two-way call must not cover."
    fix: "In _lym/landing/summary.md under ## Merge danger, name the record-format change: the new fixer_rerun key of round-<n>-snapshot.json, read through internal/state by older and newer binaries; set the door to the call that change requires and widen the blast radius to name the burler review and fix producers and the bouncer, which read that record."
    origin: assist:assist
---

The review range is `43fbea87e2d5c57e5b9fee1fd97de36fceddf54b..HEAD`, four commits.
Both board items have implementations, and every change traces to them.
The rerun reason (#176) reaches the fixer prompt correctly for the paths it records.
The push exemption (#177) has holes, though: ordinary git commands let a push leave the fork's own repositories and still pass.
That is the boundary the board asks the guard to keep.

### [BLOCKING] Remote tracking marks a remote safe while git pushes elsewhere

**Location:** `internal/burlerengine/cluster.go#recordRemote`

**Issue:** `recordRemote` sets one boolean per remote name, keyed by the created root the call ran in.
`git remote set-url --push origin /srv/target.git` followed by `git remote set-url origin /tmp/b.git` leaves `origin` marked as leading in.
Git still pushes to the push URL, `/srv/target.git`.
A worktree shares its source repository's config, but the worktree's remote change is recorded under the worktree's own root (cluster.go:165, 265).
A `git --git-dir=/tmp/r/.git remote set-url origin /srv/target.git` has `dirKnown` false, so `recordRemote` returns at cluster.go:315.
The earlier `true` for `origin` then survives, and `git -C /tmp/r push origin` passes.

### [BLOCKING] The attached --config-env form bypasses the redirect check

**Location:** `internal/burlerengine/cluster.go:451`

**Issue:** Git documents `--config-env=<name>=<envvar>`.
`gitValuedGlobalOptions[args[i]]` looks up the whole word, so the attached form falls through to the bare-option case and `redirectsPush` stays false.
The assistant confirmed with git that this form rewrites a push URL.
An `include.path` key can load a file that holds a redirecting `url.` or `remote.` key, and `gitPushRedirectKeyPrefixes` has no entry for it.

### [BLOCKING] Relative push paths resolve from the wrong directory

**Location:** `internal/burlerengine/cluster.go#leadsInto`

**Issue:** `leadsInto` resolves `p` against `invocation.dir` (cluster.go:390).
Git moves to the repository's top level before its transport opens a local path.
With `git -C r/x push ..`, the audit resolves `..` to the created root `r`, while git pushes into the parent of `r`.
In the issue's own layout, a fork repository sits directly under the target, so that parent is the target.
A relative URL given to `git remote add` from a subdirectory has the same gap through cluster.go:324.

### [MEDIUM] The halt claims the target's repository is unchanged

**Location:** `internal/burlerengine/cluster.go#forkViolation`

**Issue:** A push from a fork repository to a destination the audit could not place still leaves `targetUntouched` true.
The halt then says no fork "changed the target's repository".
The new `TestAuditClusterRound` row pushes to `/srv/target.git` and expects that sentence.
When a nested spawn is reported first, `pushFrom` stays "", so the halt says no state-changing git command ran at all.
That drops the instruction to check where the push went.

### [MEDIUM] Cloning a repository named remote.git sets the redirect flag

**Location:** `internal/burlerengine/cluster.go:210`

**Issue:** For `clone`, `namesPushRedirectKey(invocation.args)` scans the source and directory operands too.
The lowercased word `remote.git` starts with `remote.`, which marks every later push as redirected.
`remote.git` is the name the issue's own probe uses for its bare repository, so cloning it still halts the round.

### [MEDIUM] A failure exit with no report records no reason

**Location:** `internal/shedadapters/burlerfix.go#archiveReport`

**Issue:** `archiveReport` returns at burlerfix.go:141 when no report was archived.
After two died attempts that wrote no report, the next call's fixer gets a zero `FixRerun`, although earlier sessions may have edited the target.
When an older record for the same digest exists, the next fixer gets that older halt reason, not the latest one.
Within one call, an attempt 2 that archives nothing also keeps a reason that `recordedRerun` read from an earlier call (burlerfix.go:292).

### [MEDIUM] The pre-attempt archive's record is untested

**Location:** `internal/shedadapters/burlerfix_test.go#TestBurlerFixProducer_Call_TellsARerunFixerWhy`

**Issue:** The mutation report lists survivors at burlerfix.go:295 and :296.
The died-attempt row reads only the in-memory `rerun` that the second `Fix` receives, never the stored record.
`highestCompleteRound` treats a round holding a review, a fixer report and an acceptance as complete, so a canonical report is rarely present at attempt 1.
That leaves the "that session never finished the round" case close to unreachable, and no test asserts it.

### [MEDIUM] Lexical ownership admits symlinks and failed clones

**Location:** `internal/burlerengine/cluster.go#pushMutates`

**Issue:** A destination under a created root can be a symlink made with `ln` to the target's remote, and `rootContaining` accepts it.
`recordClone` records its directory whether or not git created a repository there.
`record` also accepts an anchored ancestor of the target.
The summary's Known gaps says the audit is lexical but does not name symlinks.

### [LOW] A push with -u from the target's own repository passes

**Location:** `internal/burlerengine/cluster.go#pushMutates`

**Issue:** When `from` is "" and the destination is a path into a fork repository, the push passes.
`-u` then writes upstream keys into the config of the repository the push ran from.

### [LOW] The cancellation reason is never asserted

**Location:** `internal/shedadapters/burlerfix_test.go#TestBurlerFixProducer_Call_Cancellation`

**Issue:** Mutation survivor :145 shows that no test reads the reason archiveReport records on cancellation.

### [LOW] Several new guard branches have no row

**Location:** `internal/burlerengine/cluster_test.go#TestFirstMutatingGitCommandScopesToForkRepositories`

**Issue:** These branches are reached by no row:
- nested roots, which decide the `remotes` key;
- a `branch.` key;
- `git clone -c` and `--config`;
- the `--name=value` key word;
- the attached `--origin=`;
- `git remote remove` and `rm`;
- `set-url --delete`.

### [LOW] Retiring a round keeps its rerun record

**Location:** `internal/shedadapters/burlerfix.go#archiveWholeRound`

**Issue:** `archiveWholeRound` does not archive the snapshot, and `recordRoundStart` keeps an existing snapshot.
The digest key is the only guard, so a re-accepted review with identical bytes inherits the old reason.

### [LOW] Merge danger leaves out the snapshot format change

**Location:** `_lym/landing/summary.md:84`

**Issue:** The rubric says a `two-way` call is a finding when the diff changes a record format another binary reads.
The new `fixer_rerun` key is such a change, and the section names neither the change nor the readers of that record.

## Rejected

- Affected-set F1 (fix_test.go:354's blanket revert check) is rejected: its only row is a write violation, for which the revert text is correct, so nothing is wrong today.
- Assistant F6 (fork stencils still forbid scratch git) is rejected: the board asks only that the guard stop halting on git in a fork's own repositories, and the stencils' stricter read-only rule is policy the guard backs up, not a contradiction.
- Tests F1's claim that the attempt-1 leftover report is "the exact #176 scenario" is rejected: in #176 the report was already under a timestamped name, which the failure-exit record now covers. The uncovered branch is folded into F7.
- Correctness F7's `cloneDirectoryName` result for `--bare` was rated acceptable by that lens and is not raised: it errs toward counting a command, except in the anchored-ancestor case F8 covers.
- The scope lens raised no findings: every change traces to #176 or #177 or is forced by one, and both Root-Cause trailers name defects the diff touches.
