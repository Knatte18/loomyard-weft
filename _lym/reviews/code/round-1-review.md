---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/fabriccli/fabric.go:377
    summary: The new cleanup --help text spells the records namespace root refs/lym/ as a literal in fabriccli, which PATTERN-single-repo-hub forbids outside fabric.
    fix: "In the cleanup command's Long help in internal/fabriccli/fabric.go, build the sentence that names refs/lym/ from fabricengine.RecordsNamespaceRoot, as the clone --single-repo flag help at fabric.go:107 already does, instead of the literal. In internal/fabriccli/doc.go, reword \"commands that move it under `refs/lym/`\" to \"commands that move it under the records namespace\" so the package spells no namespace."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-affected-set
    location: internal/battencli/catchup.go:224
    summary: fetchRunForCatchUp wraps the new FetchPairTips layout error as ErrOriginUnreachable, so batten run tells the operator origin is unreachable and drops the refspec way forward.
    fix: "In fetchRunForCatchUp, wrap only a fetch failure as dobbyengine.ErrOriginUnreachable. Return the layout error FetchPairTips now gives unchanged, so catchUpFailure's default branch prints it with its refspec way forward. To tell them apart, export a sentinel from fabricengine (for example ErrRecordsLayout, wrapped by originNamespaceAt's records-store refusals) and test it with errors.Is. Update the function's doc line \"A fetch that fails reads as an unreachable origin\" to say a records layout it cannot read is returned as is. Add a row that unsets the records store's remote.origin.fetch and asserts the refusal names the refspecs, not an unreachable origin."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/codetipsettle.go:24
    summary: The O2 layout reads added to fastForwardWeft, FetchPairTips and ReturnToPreviousTransition have no covering test; only resolveMergeSources is pinned.
    fix: "Add rows that unset the records store's remote.origin.fetch and call fabricengine.ForgetOriginNamespaceForTest, the way TestPull_RecordsStoreWithNoNamespacedFetchRefspecFails in pull_integration_test.go does. In codetipsettle_integration_test.go, assert FetchPairTips and ReturnToPreviousTransition return an error naming \"config --replace-all remote.origin.fetch\" and that the records store's tracking refs did not move. In parenttips_integration_test.go, assert FastForwardPair still moves the code side while the weft HEAD stays where it was."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabriccli/fabric.go:903
    summary: The plain archive tag listing is report-only, yet its ls-remote or layout-read failure fails the whole cleanup --remote verb after --apply has already deleted branches.
    fix: "In runCleanupWithFlags, on a PlainArchiveTagsOnOrigin error set fields[\"plain_archive_tags_error\"] to the error text and go on, instead of returning errWithRecordFields. Document the key in the cleanup help and fabriccli/doc.go beside plain_archive_tags. In PlainArchiveTagsOnOrigin, treat only errors.Is(urlErr, gitrepo.ErrRemoteNotConfigured) as no origin, and return any other RemoteURL error."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/clone.go:125
    summary: For a hub killed during a --single-repo seed clone, RefuseHubWithoutBolt names a plain one-URL --reset clone, which fails because origin holds no board yet.
    fix: "In RefuseHubWithoutBolt, when the records store is one-repository, name both re-runs: the join `lym fabric clone <code-url> --into <parent> --reset` when origin holds the board, and `lym fabric clone --single-repo --shortname <shortname> <code-url> --into <parent> --reset` when it holds none. Fill the shortname from the hub's recorded shortname when it can be read, else a <shortname> placeholder. Update the doc comment and the test's expected message to match."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/clone.go:124
    summary: RefuseHubWithoutBolt drops an originNamespaceAt error and falls through to the two-repository branch, naming `clone <code-url> <code-url>` for a one-repository store whose refspecs were lost.
    fix: "In RefuseHubWithoutBolt, when originNamespaceAt(recordsRoot) returns an error, return a refusal that carries that layout error (its refspec way forward) together with the --reset clone, and spell the clone in the one-repository form. Never build the two-URL form from it."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabriccli/reconcile_missingbolt_integration_test.go:105
    summary: Several new behaviours have no test row. These are the two-repository refusal's command, the two-argument existingHubError, the plain_archive_tags envelope field with its empty list on a two-repository hub, the held-lock refusal naming the given rerun, and cancel passing its own command.
    fix: "Turn TestCommand_HubWithoutItsBoltNamesTheResetClone into a table with a two-repository hubforge row whose literal expectation is `lym fabric clone <records-url> <code-url> --into <container> --reset` for both reconcile and a two-argument CloneHub. In a fabriccli cleanup --remote test, assert plain_archive_tags is [] on a two-repository hub. In teardownlock_integration_test.go's held-lock row, pass a non-empty rerun and assert the error text names it."
  - id: F8
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md#O1
    summary: The summary says the O1 refusal adds --single-repo for a one-repository hub, but RefuseHubWithoutBolt builds CloneOptions without SingleRepo and the test expects no --single-repo.
    fix: "In _lym/landing/summary.md's O1 section, describe the refusal's command as the code builds it once F5 is settled: for a one-repository hub, the code store's URL alone, with --single-repo --shortname only for the seed case."
  - id: F9
    severity: LOW
    class: consistency
    origin: handler
    location: "_lym/landing/summary.md#Merge danger"
    summary: The two-way call leaves out that the diff adds a rerun key to the pending-teardown record and the teardown child's job, both read by other binaries, and the blast radius fabric omits battencli's cancel and cleanup.
    fix: "In _lym/landing/summary.md's Merge danger section, name the new rerun key on the pending-teardown record and the teardown job. Say why it stays two-way: the key is omitempty, readers are tolerant, and an older binary ignores it and prints the described re-run. Widen the blast radius to fabric and batten (cancel and cleanup's discard)."
  - id: F10
    severity: NIT
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/clone.go:87
    summary: cloneResetCommand drops ForceBootstrap, so a --force-bootstrap clone into an existing hub is told to re-run a command the bootstrap guard refuses.
    fix: "In cloneResetCommand, append --force-bootstrap when opts.ForceBootstrap is set, before the URLs."
---

### [BLOCKING] Cleanup help spells the records namespace outside fabric

**Location:** `internal/fabriccli/fabric.go:377`

**Issue:** The cleanup `--remote` help added for O4 writes `refs/lym/` as a literal in `fabriccli`.
PATTERN-single-repo-hub's background states that no package other than fabric spells `refs/lym/`, and the base `fabric.go` had no such literal.
Its clone flag help builds the prefix from `fabricengine.RecordsNamespaceRoot`.
`hardencli` spells the prefix too, but that is existing debt, and PATTERN-code-conventions forbids adding another literal.

### [MEDIUM] Catch-up misreports the new layout error as an unreachable origin

**Location:** `internal/battencli/catchup.go:224`

**Issue:** `FetchPairTips` now returns the records layout refusal (diff `codetipsettle.go`), and `fetchRunForCatchUp` wraps every error it gets as `dobbyengine.ErrOriginUnreachable`.
`catchUpFailure` (`catchup.go:64-67`) then replaces the text with "origin is unreachable … run lym batten run <slug> again once origin answers", so the refspec way forward O2 exists to name never reaches the operator.
This path is new: before the diff, a records store with no fetch refspec fetched without error.

### [MEDIUM] Three of O2's four layout reads are untested

**Location:** `internal/fabricengine/codetipsettle.go:24`

**Issue:** `TestOneRepositoryMerge_RefusesARecordsStoreThatLostItsFetchRefspecs` pins only `resolveMergeSources`.
The reach report shows no mapped test reaching `fastForwardWeft` or `ReturnToPreviousTransition`, and no settle or fast-forward test unsets the refspecs.
Each read could be deleted and every test would still pass.

### [LOW] A failed report-only listing fails cleanup --remote

**Location:** `internal/fabriccli/fabric.go:903`

**Issue:** `PlainArchiveTagsOnOrigin` only reports, but its error returns through `errWithRecordFields` after the weft and warp sweeps have already run their deletions under `--apply`.
An origin that cannot be listed now turns the verb into an error even when every deletion succeeded.
The cleanup doc says a kept or skipped remote step leaves the exit code alone.
Separately, `PlainArchiveTagsOnOrigin` reads every `RemoteURL` error as a missing origin.

### [LOW] Killed seed clone is told to run a join that fails

**Location:** `internal/fabricengine/clone.go:125`

**Issue:** A `--single-repo` seed pushes only after `CloneHub` returns, so a hub killed before `_bolt` has an origin with nothing under the namespace.
The named `lym fabric clone <code-url> --into <parent> --reset` then takes the one-argument two-repository path and fails with no recorded warp binding (`clone.go` one-argument branch).
The join case works: `checkResetTarget` accepts the hub.

### [LOW] Layout error swallowed into a two-URL command

**Location:** `internal/fabricengine/clone.go:124`

**Issue:** When `originNamespaceAt(recordsRoot)` errors, the `else if` branch reads the records store's origin URL, which on a one-repository hub is the code URL.
It names `lym fabric clone <code-url> <code-url> --into … --reset`, and the layout error that explains the state is lost.

### [LOW] New behaviours without a test row

**Location:** `internal/fabriccli/reconcile_missingbolt_integration_test.go:105`

**Issue:** The new test drives only a one-repository hub.
`warpbinding_clone_integration_test.go:467` checks only the substring "hub already exists".
No test reads `plain_archive_tags`.
`teardownlock_integration_test.go:342` passes an empty rerun and checks only `errors.Is`.
The reach report shows no test reaching `hubCancelDeps`.

### [LOW] Summary claims --single-repo in the O1 refusal

**Location:** `_lym/landing/summary.md#O1`

**Issue:** The summary says the refusal names the clone "with `--single-repo` and the code store's URL alone for a one-repository hub".
The code sets `opts = CloneOptions{WeftURL: opts.WarpURL}`, which carries no `SingleRepo`, and the test expects `lym fabric clone <url> --into <container> --reset`.

### [LOW] Merge danger leaves out the record key and battencli

**Location:** `_lym/landing/summary.md#Merge danger`

**Issue:** The diff adds a `rerun` JSON key to `PendingTeardown`, which `lym fabric status` and `prune` of any binary on the hub read, and to the job written to the teardown child.
The rubric makes a two-way call over a record another binary reads a finding.
The change is additive and tolerant, so two-way may well hold, but the section has to say so.
The blast radius "fabric" also leaves out the `battencli` cancel and cleanup changes.

### [NIT] Reset re-run drops --force-bootstrap

**Location:** `internal/fabricengine/clone.go:87`

**Issue:** `cloneResetCommand` echoes every clone option but `ForceBootstrap`.
A bootstrap clone into an existing hub is told to re-run without it, and the guard `writeRecord && !probe.WeftLooksLikeWeft && !opts.ForceBootstrap` then refuses that re-run.

## Rejected

- code-tests NIT, the expected message is built from `fabricengine.BoltDir` and `BoltDirName`: those supply the path and the directory name, not the message the code composes. The message's wording and order are literal in the test, so it is not tautological.

## Notes

- The operator ordered the Codex assistant reviewer stopped after it produced nothing. Its strand was removed, and this review is consolidated without it.
- Checked and found sound: O3 leaves `pushLocalArchiveTags` unchanged and names the push in both helps and in `DropLocal`'s doc, per the parent's decision. O5 carries `Rerun` through the job, the child's JSON and the pending record, and a record an earlier binary wrote falls back to the description. The `Root-Cause:` trailer names defects the diff removes. The diff declares no PATTERN changes.
