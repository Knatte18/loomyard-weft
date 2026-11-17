---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/websterengine/recordbatch.go:619
    summary: A batch failed for a secret file is cleared by Master's batch_failed rung through recover-batch, which builds on the secret-bearing commits, because the named batch-start reset is refused while run.lock is held and no stencil rung routes this failure elsewhere.
    fix: >-
      Add a rung to contracts/stencils/webster/webster-template-master.md beside the record-batch batch_failed rung (line 177):
      when the batch_failed message names a secret file (its reason opens "the batch's commits add a secret file"),
      Master writes outcome stuck to the outcome path with a stuck_reason quoting the message and stops without calling recover-batch.
      Make the reason text a named constant in websterengine that failOnSecretPaths uses, so the rung and the code cannot drift.
      In failOnSecretPaths, word the way forward for the run's parent, who performs it once the run has stopped:
      `lym webster reset --to batch-start --batch NN`, then `lym webster recover-batch NN` (see F6 for the delete step).
      Extend TestRecordBatch_SecretFileFailsBatch to assert the reason constant appears in the error.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/gitrepo/gitrepo.go:453
    summary: AddedFilesBetween compares only the start and head trees, so a secret file one batch commit adds and a later commit deletes passes record-batch while it stays in the branch's history.
    fix: >-
      Replace the net tree diff with a per-commit read: walk the commits reachable from head and not from base,
      and for each commit collect the paths its tree inserts over its first parent's tree (a root commit over the empty tree),
      returning the sorted, de-duplicated union. Rename the method to say so (for example AddedFilesInCommits) and update its doc comment,
      the websterengine Git.AddedPaths doc, addedPaths and websterengine/doc.go's sentence about the start commit lacking the path.
      Add a gitrepo_test.go step that commits a file in one commit and deletes it in the next and asserts the range lists it.
      Make fakeGit.AddedPaths key on the (base, head) pair so TestRecordBatch_SecretFileFailsBatch pins that bs.StartSHA is the base.
  - id: F3
    severity: BLOCKING
    class: consistency
    origin: lens:code-tests
    location: internal/websterengine/recordbatch_test.go:1458
    summary: TestRecordBatch_SecretFileFailsBatch and its subtests, and internal/secretpath's TestLeafInvariant_StandardLibraryOnly, call no t.Parallel and carry no comment naming process-global state, against PATTERN-test-speed's Tests run in parallel rule.
    fix: >-
      Add t.Parallel() after tiergate.Require in TestRecordBatch_SecretFileFailsBatch and as the first call in each of its subtests;
      its fixture is per-test (newRecordFixture over t.TempDir and a fresh fakeGit).
      Add t.Parallel() after tiergate.Require in TestLeafInvariant_StandardLibraryOnly in internal/secretpath/leaf_enforcement_test.go.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/websterengine/recordbatch.go:434
    summary: A batch with a correctness finding fails through failOnCorrectness before the secret check runs, so its secret file is never named and its plain recover-batch way forward keeps the secret in history.
    fix: >-
      In RecordBatch's correctness branch, once headSHA resolves to a non-empty commit, run the secret check first:
      when failOnSecretPaths returns a *BatchFailedError, return it (with the RecordResult the secret path returns) instead of failOnCorrectness.
      Add a row to TestRecordBatch_SecretFileFailsBatch whose fork audit carries a correctness finding and assert the secret failure is returned.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/webstercli/wiring.go:230
    summary: No test reaches either guard with a hub-added pattern through its real wiring, so dropping wireHub's secretPaths loader, the SecretPaths field in record-batch's RecordDeps, or Commit's cfg.SecretPathMatcher() call keeps every test green.
    fix: >-
      Add a row to TestCommit_SecretPath_RefusedAndCommitsNothing whose fixture hub fabric.yaml sets secret_paths to *.key and commits tls/server.key,
      asserting the refusal names "tls/server.key (pattern *.key)".
      Add a webstercli record-batch test over a hub whose fabric.yaml adds a pattern a batch commit's path matches,
      asserting the batch fails naming that pattern, and a row with a malformed secret_paths asserting the fabric.yaml load refusal and its way forward.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/websterengine/recordbatch.go:626
    summary: failOnSecretPaths' doc comment, websterengine/doc.go and the summary claim the batch-start reset leaves each secret file on disk untracked, but the reset moves tracked files back and so removes them, making the way forward's delete step a no-op.
    fix: >-
      Drop the "delete … from the worktree" step from failOnSecretPaths' wayForwardSteps call,
      rewrite its doc comment to say the reset drops the commits and the files they added,
      and change internal/websterengine/doc.go's "deletes the file before recover-batch" and _lym/landing/summary.md's "delete the named files" to match.
      Update TestRecordBatch_SecretFileFailsBatch's "3) lym webster recover-batch 01" expectation to "2) lym webster recover-batch 01".
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/commit_integration_test.go:155
    summary: The refusal's documented guarantee that a refused Commit leaves both indexes as it found them has no assertion; the test checks only HEAD and CommitResult.
    fix: >-
      In TestCommit_SecretPath_RefusedAndCommitsNothing, after the refused call, assert `git diff --cached --name-only` is empty
      in both the warp and the weft checkout, through the package's existing test helpers.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gitrepo/gitrepo.go:422
    summary: NewPathsIn's empty-pathspec early return is unpinned, though without it git ls-files lists the whole checkout and a weft-only commit would be refused on an unrelated warp file.
    fix: >-
      In the gitrepo_test.go NewPathsIn step, while fresh.txt and staged.txt exist, assert NewPathsIn(nil) returns no paths.
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/websterengine/recordbatch.go:518
    summary: The doc's claim that a secret file fails the batch whatever its report says is pinned only for an OK report, so moving the check below the NEEDS_PARENT return passes every test.
    fix: >-
      Add a row to TestRecordBatch_SecretFileFailsBatch whose report has status NEEDS_PARENT and whose commits add a shipped-pattern path,
      asserting ErrBatchFailed and a terminal failed batch.
  - id: F10
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/secretpath/secretpath_test.go:732
    summary: The ShippedPatternWinsOverAdded row adds the same string .env as the shipped pattern, so it cannot detect added patterns being tried before shipped ones.
    fix: >-
      Change the row's added pattern to .e* and keep the want of .env, so a reversed order returns .e* and fails.
---

### [BLOCKING] Master clears a secret-file failure through recover-batch, keeping the secret

**Location:** `internal/websterengine/recordbatch.go:619`, `contracts/stencils/webster/webster-template-master.md:177`

**Issue:** `failOnSecretPaths` names `lym webster reset --to batch-start --batch NN` first.
`PlanReset` refuses every target except report-head while `run.lock` is held (`internal/websterengine/reset.go`, the `active && to != ResetToReportHead` branch).
Master calls `record-batch` from inside the run.
Master's `batch_failed` rung tells it to run `recover-batch` at once, and the head-mismatch rung allows no reset other than report-head.
`recover-batch` re-drives the batch on top of the commit that holds the secret file and runs no secret check, so the guard costs one failed record and the secret stays on the branch.
PATTERN-refusal-way-forward's "No denied recovery" section bars a way forward its receiver cannot perform.

### [BLOCKING] The batch check reads the net tree delta, not each commit

**Location:** `internal/gitrepo/gitrepo.go:453`

**Issue:** `AddedFilesBetween` runs `object.DiffTree` between the start and head trees.
A batch whose first commit adds `.env` and whose second deletes it shows no insert, so `record-batch` records it, and the file sits in the branch's history and goes out with the next push.
The board entry asks for a refusal of "a commit that adds a path matching" the list, which is a per-commit property.

### [BLOCKING] New tests skip t.Parallel without naming shared state

**Location:** `internal/websterengine/recordbatch_test.go:1458`, `internal/secretpath/leaf_enforcement_test.go:12`

**Issue:** Under PATTERN-test-speed's "Tests run in parallel" rule, a test calls `t.Parallel` unless it touches process-global state, and a comment names that state.
Neither new test does either.
Neither touches global state: the record fixture is per-test, and the leaf scan reads source files.
Older siblings of both tests skip `t.Parallel` too, but the rule binds these new ones.

### [MEDIUM] A correctness finding hides the secret-file failure

**Location:** `internal/websterengine/recordbatch.go:434`

**Issue:** The correctness branch returns through `failOnCorrectness` before the secret check at line 518.
Its way forward is plain `recover-batch`, so a batch that both trips an audit finding and commits a secret file never names the file, and recovery builds on it.

### [MEDIUM] Hub-added patterns are not tested through either guard's wiring

**Location:** `internal/webstercli/wiring.go:230`, `internal/webstercli/recordbatch.go:114`, `internal/fabricengine/commit.go:141`

**Issue:** `TestLoadSecretPathMatcher` covers the loader, and `TestRecordBatch_SecretFileFailsBatch` covers a matcher handed in directly.
Nothing reaches the path from `fabric.yaml` to either guard.
`TestCommit_SecretPath_RefusedAndCommitsNothing` uses shipped patterns only, and no `webstercli` test mentions secret paths.
The new `fabric.yaml` load refusal in `record-batch` is never reached either.

### [LOW] The reset does not leave the secret file on disk

**Location:** `internal/websterengine/recordbatch.go:626`

**Issue:** `lym webster reset` moves the worktree's HEAD, index and tracked files back to the target (`internal/webstercli/reset.go`, its long help).
A file the dropped commits added is tracked at HEAD and absent at the batch start, so the reset removes it.
The comment's "leaves each file on disk untracked" is therefore false, and the way forward's delete step names files that are already gone.
`internal/websterengine/doc.go` and `_lym/landing/summary.md` repeat the claim.

### [LOW] The untouched-index guarantee is unasserted

**Location:** `internal/fabricengine/commit_integration_test.go:155`

**Issue:** `refuseSecretPaths` documents that a refused `Commit` stages nothing.
The test checks HEAD and `CommitResult` only, so a refusal moved after `git add` would still pass.

### [LOW] NewPathsIn's empty-pathspec return is unpinned

**Location:** `internal/gitrepo/gitrepo.go:422`

**Issue:** `Fabric.Commit` calls `NewPathsIn` with an empty warp list on every weft-only commit.
Without the early return, `git ls-files --others --exclude-standard -z --` lists every untracked file in the checkout.
No test calls it with an empty list.

### [LOW] Report-status independence of the batch check is unpinned

**Location:** `internal/websterengine/recordbatch.go:518`

**Issue:** The doc comment says the batch fails whatever its report says.
Only an OK report is exercised.
A NEEDS_PARENT report returns at line 544, so moving the check below that return would record a secret-carrying batch as waiting, with no test failing.

### [LOW] The shipped-first row cannot fail

**Location:** `internal/secretpath/secretpath_test.go:732`

**Issue:** The row's added pattern equals the shipped `.env`, so `Match` returns `.env` in either order.
The row therefore does not pin the documented "shipped patterns first".

## Focus verdicts

- **secretpath matching.** The shipped list matches the entry's five patterns against the base name at any depth.
  `.env.example` and `id_rsa.pub` pass, and slash patterns match the whole path.
  Matching is case-sensitive, which is no defect (see Rejected).
- **Only new paths, and nothing left behind.** Fabric refuses only untracked or staged-added paths, before any lock or staging.
  The record-batch reasoning is in F1, F2 and F4.
- **NewPathsIn.**
  - On an unborn branch, `git diff --cached --diff-filter=A` lists the staged file; I probed it in a scratch repository.
  - `--no-renames` reports a staged rename as an add.
  - Each git error is wrapped and refuses the commit, so the read fails closed.
  - Multi-commit batches are AddedFilesBetween's concern, in F2.
- **Hub config.** `secret_paths` merges with the shipped list and never replaces it.
  A malformed pattern surfaces as `configengine.ErrInvalid` naming the key.
  The key is closed to the local layer.
- **Scope.** Every changed file serves the two guards or the hub patterns.
  The PATTERN bullets for fabric-boundary and leaf-packages land in the diff.

## Rejected

- Case-insensitive matching (lens:code-correctness, LOW): the entry names the patterns literally, and git records each path as typed. A hub that wants `.ENV` adds it in `secret_paths`.
- An unpinned rename row for AddedFilesBetween (lens:code-tests, LOW): F2 replaces that method and its contract, so the row is moot.
- An unpinned base argument (lens:code-tests, LOW): folded into F2's fix, which keys the fake on the (base, head) pair.
- A Master stencil rung missing for the new failure (lens:code-affected-set, BLOCKING): the same defect as F1, merged there.
- A doc comment contradicting the reset verb (lens:code-affected-set, MEDIUM): the same defect as F6, merged there.
- A false positive from a parent merge inside the batch range (handler): reconcileReportHead tolerates a parent merge only above the report head, so start..report-head holds the batch's own commits.
