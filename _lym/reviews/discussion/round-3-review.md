---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:172
    summary: "D7 step 4 bounds only test starts by redundancy_budget_min, so coverage builds and the step's own gate slot wait have no stated limit, and the stated cost in the rationale understates the worst case"
    fix: "In D7 step 4 state what redundancy_budget_min counts (measured from the step's start, builds included) and that no coverage build or test starts once it is spent, with the packages left over listed as not judged with that reason. State how the step takes its gate slots (Constraints already require them) and that the wait for them is unbounded as D5 states for the verb, or give the limit. Then restate the rationale line on cost (line 199) as a worst case that holds under those limits, for example the budget plus one build under redundancy_build_timeout_min plus one redundancy_test_timeout_sec, and name the slot wait as outside it. Keep the never-changes-the-result rule as it is."
  - id: F2
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:130
    summary: "D6 starts a follow-up refresh when the last refresh reported unbuilt stale packages, but no decision says through which channel the dobby reads that report from a child it waits on"
    fix: "In D5 or D6 name the one channel by which the verb tells its caller that the cap left stale packages unbuilt (a distinct exit code, or a line the dobby parses from the verb's output, or a record the verb writes) and say how the dobby reads it. Add that a refresh that is killed, or whose channel cannot be read, counts as reporting none left, so only a prime HEAD change starts the next one. Name the channel in the D6 acceptance bullet so a test can reach it."
  - id: F3
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:1
    summary: "D1 moves the spawn scan and the eligibility rules into internal/testcover but not the keep-marker scan that D7 step 2 also needs, and that scan sits in the cmd/testtiming main package, which cannot be imported"
    fix: "In D1 add that the scan for `//testtiming:keep` markers and their reasons, now in cmd/testtiming/keep.go, moves into internal/testcover with the spawn scan, and that the tool keeps reporting a kept candidate under Kept without a change to its classification while D7 step 2 drops it from judging. Name the keep scan beside the spawn scan in the D1 rationale sentence that says a main package cannot be imported."
  - id: F4
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md:74
    summary: "D4 hashes each Go file in the package directory while D5 compares the Go files the directory tracks at HEAD, so a gitignored Go file makes the two sets differ and the package stale on every call"
    fix: "In D4 change the file hash line to say the map holds a content hash of each Go file the package directory tracks at HEAD, the same set D5 compares and D5's dirty check covers. No other text changes."
  - id: F5
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:93
    summary: "D5 reads the tracked Go file set at HEAD, the modified and untracked files and HEAD itself in the prime worktree, but names no home for those git reads while D7 routes its own reads through fabricengine"
    fix: "In D5 state that the verb reads the tracked Go file set at HEAD through the same fabricengine reader D7 step 1 adds (CodeDirAt with CodeFileAt at the revision HEAD), and the modified or untracked check through the fabric status read, so no git runs outside internal/fabricengine. Add PATTERN-fabric-boundary to the Constraints bullets beside the other PATTERN entries the feature touches."
---

### [MEDIUM] D7's step has no stated bound on builds or its slot wait

**Location:** `_lym/discussion/decision-record.md` D7 step 4 (line 172 to 176) and D7 Rationale (line 199 to 200).
**Issue:** The focus asks whether every wait the step can block on has a stated limit or is stated as unbounded.
D5 answers it for the verb: a slot wait that is stated unbounded, a build under `redundancy_build_timeout_min`, a test under `redundancy_test_timeout_sec` plus a grace, and a call total.
D7 step 4 gives the builds and tests the same per-item limits.
Its only total is `redundancy_budget_min`, which it defines as the point after which "the step stops starting tests".
A coverage build runs once per package holding a kept test, and up to `redundancy_max_tests` tests can sit in that many packages.
The record does not say whether the budget counts builds or whether a build starts after the budget is spent.
Read literally, the step's worst case is one `lym` build plus one coverage build per package at ten minutes each, before any test, and the budget limits none of that.
The rationale states the cost as "one coverage build per touched package ... and those tests' run time up to `redundancy_budget_min`", which presents the budget as the run-time bound.
The step also holds gate slots (the Constraints require it), and the record says nothing of how it takes them or how long it waits.
The gate runs the step after the tiers have released theirs, so it is a fresh wait.
D7's rule that the step never changes the gate's result holds, but the gate call can be held for a duration the record does not state.
D6's orphan bound rests on D5's stated time bound and holds.
The step's timeout and grace wait for a test whose descendant holds its output pipe are stated in D5 and cited in D7 step 4, so those hold.

### [LOW] D6's follow-up refresh rests on a report channel the record does not name

**Location:** `_lym/discussion/decision-record.md` D6 (line 130) with D5's last paragraph (line 116 to 118).
**Issue:** D6 starts another refresh when "the last refresh reported stale packages the `--max` cap left unbuilt".
D5 says the verb "exits listing" its skipped and failed packages "apart from the count of stale packages the `--max` cap left unbuilt".
The dobby starts the child, waits on it and holds no other link to it.
Nothing says whether the dobby learns the count from an exit code, from output it captures, or from a record.
The child is started as a detached process in the pattern the dobby code follows, with no stdio, so a listing printed on exit reaches nobody.
Without a named channel D6's loop control (start again on leftovers only) cannot be implemented from the record, and a plan writer may pick "non-zero exit means leftovers", which also fires on a failed build.
The record already bounds the rest of the loop correctly: skipped packages and failed builds never start another refresh, and each call that leaves packages unbuilt has built at least one.
A killed refresh (for example by the anchor move's `EndSparing` of the hub owner) reports nothing, and the dobby it restarts under starts fresh and refreshes on its first tick.

### [LOW] The keep-marker scan is not placed in a package the gate step can import

**Location:** `_lym/discussion/decision-record.md` D1 (lines 21 to 35) and D7 step 2 (line 170).
**Issue:** D7 step 2 drops tests marked `//testtiming:keep`, and D1 says what moves into `internal/testcover`: the runner, profile parsing, containment, the map, the eligibility rules and the spawn scan.
The scan for keep markers is `scanKeeps` in `cmd/testtiming/keep.go`, in the `main` package.
`internal/impactset` reads the same marker for placement checks but does not return the reasons.
So the spawn scan and the eligibility rules are placed, which settles that half of the focus question, and the keep scan is the one piece the step calls that D1 leaves in an unimportable package.
The compiler catches this at plan time, which is why the severity is LOW.
D1 also says the eligibility rules move "unchanged", while D7 step 2 makes a keep marker an exemption that the tool today applies to the report only, so the record should say both.

### [LOW] D4 and D5 name different sets of Go files

**Location:** `_lym/discussion/decision-record.md` D4 (line 74) and D5 (lines 89 to 92).
**Issue:** D4 stores "a content hash of each Go file in the package directory".
D5 calls a package stale when "the Go files its directory tracks at HEAD, as a set or in the content of one of them, differ from what its map recorded".
D5's dirty check also covers only modified and untracked files, and `git status` does not report a gitignored file, such as a generated one.
A package directory holding a gitignored Go file records that file in D4's map and never finds it in D5's tracked set, so the sets differ after every rebuild and the package is stale again on the next call.
Each such call spends a build and a place against `--max`.
D6 does not start refreshes for it, since D6 triggers only on a HEAD move or leftovers, so the cost stays within calls the operator or a HEAD move starts.

### [LOW] D5's git reads at HEAD have no home under the fabric boundary

**Location:** `_lym/discussion/decision-record.md` D5 (lines 89 to 99) and Constraints (lines 232 to 243).
**Issue:** `PATTERN-fabric-boundary` allows only read-only SHA and status verbs outside `internal/fabricengine`.
D5 needs the set of Go files a directory tracks at HEAD, which is a tree listing and not one of those verbs.
D7 step 1 places its equivalent reads in `fabricengine.CodeDirAt` (new) and `fabricengine.CodeFileAt` (new).
D5 leaves its own reads unplaced, and the Constraints list names other PATTERN entries but not this one.
A plan writer can reach the right answer from D7, so the severity is LOW.

## Rejected

- Focus 1, whether a package with no test file, no `tier0` or `tier1` test, or a failed build stays stale on every call.
  Not a defect after the round-2 fix.
  D5 maps such a package with its file hashes and no tests and counts it against `--max`, so it is not stale again at the same files, and a failed-build map is retried only after a HEAD move.
  D6 starts only on a HEAD change or on leftovers, never on skipped packages, so it cannot start a refresh on every tick.
  A later changed package is not left unbuilt: each HEAD move starts a refresh, and the cap leftovers start follow-ups.
  The one residue is the channel that F2 names.
- Focus 2, whether the stop route ends a running test-map child.
  It holds under the documented contract of `runproc.EndRole`.
  `EndRole` ends the live records of the given role under the owner and the processes below them, spares other roles, and skips only the caller and its ancestors.
  A child registered under `runproc.HubOwner` with the `test-map` role is a record `EndRole(HubOwner, test-map)` selects, and its descendants, whether in their own groups or not, are reached through the process walk.
  The `EndRole` of the prime dobby's own role reaches the child only through the process walk below the dobby's record, which `end.go` does not document as a guarantee.
  The new call makes the stop hold by the documented contract, which is the round-2 F2 fix, and it did not regress.
  A child started in the pattern the code already uses (detached, waited on, registered with `ChildRecord`) leads its own group, which is what the group signal needs.
- Focus 3, whether D7 step 6 records the same set D8 and the repeat-call skip compare.
  It does.
  Step 6 records the function hash of every test step 1 returns, before steps 2 to 4 drop any, and D8 and the repeat-call skip both compare step 1's output against that set.
  A diff that also changes a `tier2` test therefore matches, as the D7 and D8 acceptance bullets say.
- Round-2 items re-read for regression: the `runproc.EndRole` route, the per-test process group and grace wait, `redundancy_build_timeout_min`, the rename rule in D7 step 1 and the spawn scan's move are all stated and none regressed.
