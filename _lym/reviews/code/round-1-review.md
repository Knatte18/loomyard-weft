---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/landingshed/finalize_test.go#TestFinalize_RecordLandingFailureKeepsDone
    summary: No test pins that a failed post-merge head read fails Finalize when Deps.LandLocally is wired, the path the diff rewires by passing headErr into pushParent.
    fix: "Add a test case in internal/landingshed/finalize_test.go next to TestFinalize_RecordLandingFailureKeepsDone (it holds only Done rows, so add a sibling test or a table with an expected outcome). Use newTestDeps(t) and set deps.LandLocally to a func that records each call. Use a recordingParentMerger with results [{result: fabriclayout.MergeResult{Committed: true}}], headSHA \"landed1\", headErr set to an injected error and preMergeHeadSHA stringPointer(\"parent0\"), so the pre-merge read succeeds and only the post-merge read fails. Build Finalize as the neighbouring tests do and call fz.Call(context.Background()). Assert that the returned error wraps the injected error (errors.Is) and that LandLocally was never called. The injected error must reach only the post-merge read: the fake returns preMergeHeadSHA only while len(calls) == 0, so the step-1b4b read sees no error as long as fz.gatedTip is empty, which it is in these in-package fixtures."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomcli/landingdeps.go#recordHeldLanding
    summary: The parent branch read moved from before the pair push lock and status commit to after them, so a failed read now leaves a committed status checkpoint that no ledger landing holds back. The writer also takes a target rather than the parent branch the Exports line names.
    fix: "Give writeLedgerLanding a branch argument: writeLedgerLanding(parent runpusher.Target, branch, slug, tip, records string) error. Drop its own gitrepo CurrentBranch read and pass branch into pushledger.Landing.ParentBranch. The target stays, since runpusher.RecordLanding needs it. In recordHeldLanding, read the branch first, before fabricengine.HoldPairPushLock, as the base did: branch, err := gitrepo.New(parent.Location.WorktreePath()).CurrentBranch(), returning fmt.Errorf(\"read the branch of the parent pair %s: %w\", parent.Location.WorktreePath(), err) on failure. Then pass it to writeLedgerLanding. Update writeLedgerLanding's doc comment to name branch as the parent branch the landing is recorded on. In TestWriteLedgerLanding_AChainOfOwnTipsReadsAsOwed (internal/loomcli/landlocally_integration_test.go), pass f.branch as the new argument. Update the signature listed in _lym/landing/summary.md, together with its design-decision bullet about taking the target."
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/landingshed/deps.go:174
    summary: The new LandLocally contract and the finalize.go:569 comment call the post-merge head the run's own landing commit. The unchanged Finalize.Call doc says a re-run over an already-landed parent records the parent's head, which after a later landing is another run's commit.
    fix: "Make the new wording match what the code passes, as the base did and as the Call doc (internal/landingshed/finalize.go:280-285) states. In internal/landingshed/deps.go, replace the line \"tip is the run's own landing commit.\" with two lines: \"tip is the parent's head right after the parent-side merge, which is the run's own landing commit on a pass whose merge commits.\" and \"On a re-run over a parent that already holds the run's work it is the parent's head.\" In internal/landingshed/finalize.go:569, replace the comment with \"The parent's head right after the merge, the run's own landing commit on a pass whose merge commits, goes to both landing records.\" Make the same narrowing in the recordLanding and pushParent doc comments (finalize.go:958 and :1029), which call it the run's own landing commit. Change no code."
  - id: F4
    severity: NIT
    class: consistency
    origin: lens:code-tests
    location: internal/loomcli/landlocally_integration_test.go:498
    summary: The with-the-last-landing-commit-unrecorded assertion would also pass if the writer recorded the parent's head as every tip. Its one behaviour, that an unrecorded commit reads as not owed, is already pinned at fabricengine.AheadByOnly by internal/fabricengine/landing_integration_test.go.
    fix: "In TestWriteLedgerLanding_AChainOfOwnTipsReadsAsOwed, drop the first loop over slugs[:2] and the requireAheadByOwed(\"with the last landing commit unrecorded\", false) call. Record all three runs in one loop, then keep requireAheadByOwed(\"with every landing commit recorded\", true) and the rest unchanged."
---

### [MEDIUM] A failed head read with LandLocally wired is untested

**Location:** `internal/landingshed/finalize_test.go#TestFinalize_RecordLandingFailureKeepsDone`

**Issue:** The diff turns the second head read in `pushParent` into a `headErr` passed down from `landUnderLock` (`internal/landingshed/finalize.go:1051-1053`).
The change description claims a failed read "still fails the call when one is" wired.
No test reaches that branch.
`finalize_unlocked_test.go:389-393` injects `headErr` but wires no `LandLocally`, and it fails at the step-1b4b read (`finalize.go:507-510`), which carries the same message.
The one test that wires `LandLocally` (`finalize_test.go:202`) sets no `headErr`.
If the `headErr` check were dropped, `LandLocally("")` would write an empty tip into the ledger and no test would fail.

### [LOW] The parent branch read now follows the status commit

**Location:** `internal/loomcli/landingdeps.go#recordHeldLanding`

**Issue:** At the base, `recordHeldLanding` read the parent pair's branch before `HoldPairPushLock` and `commitRunStatus`, so a failed read left nothing behind.
Now `writeLedgerLanding` reads it after the status checkpoint is committed.
A failure there returns with that commit made and no ledger landing holding it back.
Once the lock is released, a pair push can then send it to origin ahead of the landing, which the lock and PATTERN-pair-remote's held-records rule exist to prevent.
A re-step recovers only if no pair push ran in between.
The `oldestHeldRecords` and `RecordLanding` failures already leave this state, so the diff widens an existing window rather than opening a new one.
The amended entry's Exports line also fixes the writer as "taking the parent branch", and `writeLedgerLanding` takes a `runpusher.Target` and reads the branch itself.
Taking the branch as an argument fixes both.

### [LOW] The new landing-commit contract overstates the re-run case

**Location:** `internal/landingshed/deps.go:174`

**Issue:** `Deps.LandLocally` now says "tip is the run's own landing commit", and the comment at `finalize.go:569` says the post-merge head is that commit.
On a re-entered Finalize over a parent that already holds the work, the merge commits nothing.
That holds after a crash past `LandLocally` that leaves the landing owed, while another run lands on top.
The head is then the later run's commit, and the unchanged `Finalize.Call` doc (`finalize.go:280-285`) says so: "the landing is recorded again with the parent's head".
The behaviour matches the base, which the amended entry accepts ("for a lone run is the commit it lands today").
The new wording claims more than the code does, though, and contradicts the `Call` doc.

### [NIT] The unrecorded-commit assertion cannot fail on the behaviour under test

**Location:** `internal/loomcli/landlocally_integration_test.go:498`

**Issue:** With all three merges made before any record, a writer that used the parent's head as the tip would give runs one and two run three's commit.
The parent would still read as not ahead by owed tips alone, so the check passes either way.
What it pins, that an unrecorded commit is not owed, is already `fabricengine.AheadByOnly`'s tested behaviour.

## Rejected

- code-tests finding 2 (no test of a failed parent branch read in `writeLedgerLanding`): once F2 restores the read before the commit, the failure leaves nothing behind, as at the base, which had no such test. It is not a behaviour the diff adds.
- code-scope F1 (the writer takes a target, not the parent branch): the same change as F2 fixes it, so it is folded into F2.
- code-correctness finding 2 (the comment at `finalize.go:569` is false on a re-run): this describes the same defect as code-affected-set finding 1, kept as F3. It is lowered to LOW because the code matches the base and the amended entry, and only the wording is wrong.

Holistic checks with no finding:

- The single head read feeds `RecordLanding` and `LandLocally` the same commit. Under the parent lock, the base's two reads could not differ on a lone run.
- The pull-request-merged path and the `PushSkipped` and nil-seam paths keep their base outcomes. The one extra unconditional head read is harmless.
- The standing-landing rewrite keeps the tip given and keeps the earlier records commit through `oldestHeldRecords` under the entry's two conditions. The new test's re-record of run one pins this, as the acceptance criterion asks.
- The change description's claims hold. The `two-way` door fits: no record format, config meaning or outward push changes. The landing blast radius covers the diff.
- No repository document outside `_lym` describes the ledger landing's tip, so none is left stale. The commit range carries no trailers and needs none.
