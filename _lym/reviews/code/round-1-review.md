---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: assist:assist
    location: internal/fabricengine/clone_singlerepo_integration_test.go:596
    summary: The new expected messages spell the owned board branch main and the namespace root refs/lym/ as literals, against PATTERN-code-conventions.
    fix: "In the BoardWithNoUpstreamIsNeverPushed table, build the OriginHoldsNoBoard fragment with fabricengine.RecordsNamespaceRoot in place of the literal refs/lym/, and build the two OriginHoldsTheSeed board-branch fragments with fabricengine.OneRepositoryBoardBranch in place of the literal main in branch.main.remote and branch.main.merge, as namespaceBoardRef already does at the top of the file. Keep the surrounding wording literal."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/pushclassified.go:223
    summary: An origin board that descends from this seed, after a joiner pushed to it, is called another clone's board, and the way forward to remove the hub discards any local board commits origin lacks.
    fix: "Before the ancestry check in refuseUnseededBoard, fetch origin's board ref into the records store so originTip is present locally. Then treat origin's board as this seed's whenever it shares history with the local tip (git merge-base of originTip and localTip succeeds), naming the upstream settings way forward, and keep the another-board refusal for an origin board with no common history. Update the doc comment's three-state list and the doc.go paragraph to match, and add a row to BoardWithNoUpstreamIsNeverPushed that seeds origin, drops the upstreams, advances origin's board with a commit made in another clone, and asserts the upstream settings way forward."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/clone_singlerepo_integration_test.go:579
    summary: Three refusal branches have no row; the strict-ancestor seed case, the unreadable origin, and a committed local board facing an origin commit absent locally (the OriginHoldsAnotherBoard row reaches only the unborn-board branch).
    fix: "Add rows to BoardWithNoUpstreamIsNeverPushed: one that commits on the seeded board after dropping the upstreams and asserts the upstream settings way forward, then a successful push after setting them; one that points remote.origin.url of the records store at a missing path and asserts the refusal names a rerun once origin answers, with no failed-push record; and one that commits on the CloneHub board before putting an unrelated commit on origin's board ref and asserts the join way forward."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/destroy.go:1783
    summary: A board seed push that ended unanswered and was not withdrawn is reached only when origin holds another clone's board, yet its new wording omits that and the cause still names run the same clone again, which the clone refuses since origin holds a board.
    fix: "In pushCloneSeed's board branch, when withdrawUnansweredSeedPush returns withdrawn false with a nil error, word the cause as origin holding another hub's board, and route it to seedPushCause's join way forward (run lym fabric clone <url> to join that hub, or use another origin) instead of the rerun. Add a row to TestCloneSingleRepo_SeedPushKilledBeforeOriginsAnswer whose post-receive hook moves the board ref to another commit before killing the client, and assert the other board survives and the error names the join."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/destroy.go:1787
    summary: The board failure states the seed returns the records branch before the restore runs, so a failed restore leaves an error that claims the restore and then reports it failed.
    fix: "In pushCloneSeed, drop the clause the seed returns the records branch to what origin held before from the board cause at destroy.go:1787, and in the fail closure append the records branch was returned to what origin held before only when restoreSeededRecordsBranch returns nil, keeping the existing also-failed wording otherwise. Update the BoardLanded row's expected fragment to match."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/pushclassified.go:220
    summary: The CurrentSHA, isAncestorOrEqual and PrimeRecordsBranchKey read errors in refuseUnseededBoard are returned without the refusal's context or a way forward.
    fix: "In refuseUnseededBoard, wrap each of the three errors with the refused prefix and the phrase that origin's board could not be compared with this board, followed by a way forward naming a rerun once the records store at j.path reads; for the PrimeRecordsBranchKey read, name the key and the records store path."
  - id: F7
    severity: NIT
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/pushclassified.go:219
    summary: An unborn board puts the sentinel no commits where a commit belongs, so the message reads this board at no commits does not contain.
    fix: "In refuseUnseededBoard, keep localTip empty for gitrepo.ErrNoCommits and word that case's refusal as origin holds another board at <tip> and this board holds no commits, with the same join way forward."
---

### [BLOCKING] Owned branch and namespace literals in the new test expectations

**Location:** internal/fabricengine/clone_singlerepo_integration_test.go:596

**Issue:** The `OriginHoldsNoBoard` row expects `origin holds no board under refs/lym/`, and the `OriginHoldsTheSeed` row expects `branch.main.remote` and `branch.main.merge` spelled with the literal `main`.
`PATTERN-code-conventions` says a branch lym owns in its records namespace is one named constant in fabric, never a literal at a use site, and an existing literal is debt a change never adds to.
The same file already builds `namespaceBoardRef` from `fabricengine.RecordsNamespaceRoot` and `fabricengine.OneRepositoryBoardBranch`, and the same row uses it for the merge value.
A violation of a listed constraint is blocking.

### [MEDIUM] An origin board that descends from this seed is called another clone's

**Location:** internal/fabricengine/pushclassified.go:223

**Issue:** After a seed lands and its clone is killed, a joiner can clone origin and push board commits.
Origin's tip then descends from the local tip, and is usually absent from the local store, so `isAncestorOrEqual` (`remoteleftover.go:312-321`) answers false.
The refusal claims "another clone seeded origin", which is false, the very fault the board entry asks to remove.
Its way forward removes the hub, which discards any board commits the local hub made after the seed.

### [MEDIUM] Refusal branches with no test row

**Location:** internal/fabricengine/clone_singlerepo_integration_test.go:579

**Issue:** The `OriginHoldsAnotherBoard` row clones with `CloneHub`, whose board is unborn; commit f7d565cb6 added the `ErrNoCommits` branch for that reason, so the row never reaches `isAncestorOrEqual`'s missing-commit answer.
No row commits on the board after the seed, so a mutant narrowing the ancestor check to tip equality passes every row, though the doc comment states the ancestor case.
No row makes origin unreadable, so the rerun way forward is unreached.

### [MEDIUM] The not-withdrawn board failure still names a looping rerun

**Location:** internal/fabricengine/destroy.go:1783

**Issue:** With the board leased on absence, `withdrawUnansweredSeedPush` keeps the hub when origin holds no board (`destroy.go:1869`), so a nil error with `withdrawn` false means origin's board stands at a third tip, another clone's.
The diff's new wording, "origin took the records branch, and the board's push ended without an answer", leaves that out, and `seedPushCause`'s default names "run the same clone again".
A `--single-repo` clone onto an origin that holds a board is refused (`recordslayout.go:220-222`), so the way forward loops, the fault the board entry names.
No row of the killed-push table reaches this branch.

### [LOW] The restore is claimed before it runs

**Location:** internal/fabricengine/destroy.go:1787

**Issue:** The cause says "the seed returns the records branch to what origin held before", and only then does `fail` run `restoreSeededRecordsBranch`.
When the restore loses its lease, `fail` appends "restoring the records branch … also failed", so one error says both.

### [LOW] Three refusal error paths lose their context

**Location:** internal/fabricengine/pushclassified.go:220

**Issue:** A `CurrentSHA` or `isAncestorOrEqual` failure is returned bare, without saying the board push was refused.
A missing `lym.primeRecordsBranch` surfaces as git's `exit status 1` under "read lym.primeRecordsBranch to name the records branch's upstream", with no way forward.

### [NIT] The unborn-board refusal reads "this board at no commits"

**Location:** internal/fabricengine/pushclassified.go:219

**Issue:** The sentinel string takes the place of a commit in the sentence "which this board at no commits does not contain".

## Rejected

- Assistant: quote the paths and branch names in the printed `git config` commands. Fabric's other way forwards print `git -C <path>` unquoted (`recordslayout.go:91-101`, `destroy.go:1847-1850`), and the branch names are lym's own, so this is the repository's convention, not a defect of this diff.
- Assistant: cover the `(false, nil)` withdrawal result. Merged into F4, whose fix adds that row; on the records side the result changes no wording, so it needs none.
- code-tests: the unborn-board branch has no row. The `OriginHoldsAnotherBoard` row reaches it, since `CloneHub` leaves the board unborn.
- code-tests: the another-board row pins the missing-commit answer. It reaches the unborn branch instead; F3 carries the gap.
- code-correctness F4: an empty `lym.primeRecordsBranch` value prints `branch..remote`. The clone always writes the key with the branch name (`clone.go:391`), so the state cannot arise from lym's own writes.
