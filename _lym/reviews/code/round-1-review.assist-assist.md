---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/beam/beam-template-describe.md:37
    summary: The describer stencil restates the shared final-summary format.
    fix: Remove the heading syntax and Merge danger format instructions from the numbered list. Keep the reference to loom-final-summary-spec.md and the beam-specific requirements to rewrite the end result whole and list every member pull request.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:code-tests
    location: internal/dobbyengine/describe_test.go:93
    summary: Description tests bypass the daemon scheduling path that enables the feature.
    fix: Extend the existing Run scheduling tests with BeamDescribe dependencies and observable agent starts and records. Assert no describer starts under an anchor-move or adopt hold, that release starts it, and that a later tick records a valid draft. Exercise Run rather than directly calling countLandings and describeStep.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/dobbyengine/describe.go:224
    summary: A persistent draft read error prevents timeout settlement and every later rewrite.
    fix: In settleDescribe, retain transient read errors before the deadline but settle an expired attempt even when draftFindings fails. Stop its strand, clear that attempt while preserving later landings, and report the read error as the timeout cause. Extend the existing unreadable-draft scenario past its deadline and assert a later landing can start a new rewrite.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/dobbyengine/describe.go:230
    summary: A PR-update failure is reported as if the replacement description was never recorded.
    fix: Carry recording success separately from a subsequent PR-update error through recordDescription, rewriteDescription and BeamDescribeDeps.Record. Have settleDescribe report that the new description was recorded but the PR update failed, with the existing pr update recovery instruction. Reserve the old-description-stands notice for failures before replacement, and correct the seam comment accordingly.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/beamdescribe.go:197
    summary: A failed write can truncate the old description before the rollback path is reached.
    fix: Replace the direct truncating write in ReplaceBeamDescription with the repository's atomic-file replacement mechanism, preparing the full replacement before changing the destination. Keep the existing commit-failure restoration. Cover a replacement-write failure and assert the old description remains intact and commit is not called.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/dobbyengine/describe.go:231
    summary: A transient notice transport failure permanently loses the rewrite failure notice.
    fix: Persist the failure notice as owed when settling a refused or timed-out rewrite, and clear it only after Notice succeeds. Retry delivery without rerunning the record operation or consuming newer landings. Extend the existing refusal and timeout scenarios with a failed delivery followed by successful delivery.
  - id: F7
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: contracts/stencils/beam/beam-template-describe.md:29
    summary: The reading instructions omit reading every member pull request.
    fix: Add an explicit reading instruction for every member pull request, using a supported read path or supplying its contents in the prompt. Keep the separate instructions to read the beam commits and diff. Do not treat the shortened rendered commit list as the member PR contents.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/beamcli/dobbydeps_integration_test.go:67
    summary: The wiring test never exercises the returned describer recorder or checks its configured gate and model.
    fix: Extend the existing hub dependency test to verify nondefault configured describer model and timeout, the returned adapter's description gate, and recording through the returned BeamDescribe.Record callback. Reuse the existing facts and recording fixture instead of adding another helper-only test.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/dobbyengine/describe_test.go:148
    summary: No assertion protects existing busy work when a describer settles.
    fix: Extend the existing passing-draft settlement scenario with busy already true and assert settlement leaves it true. Prefer observing the resulting busy/backoff behavior through the existing loop harness. This must fail if describeStep assigns waiting directly instead of preserving the prior busy value.
  - id: F10
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/landing/summary.md#Merge danger
    summary: The two-way door claim omits the automatic outward PR updates and the new state-record fields.
    fix: Change the Merge danger door to one-way and explain that reverting code does not restore PR text already edited or undo recorded description updates. Expand the blast-radius assessment to account for beam descriptions and PR publication as well as dobby scheduling.
---

Reviewed cce04199925a2dc6eee25716d8112c0649d98d40..9558b0da6a8ebf97afc887f45eeab938e7387ebc.
The decision record and board entry contain the same goal, with no amendments.
All required lenses returned, and their reports are saved beside this file.

### [BLOCKING] Keep the format in its owning contract

**Location:** contracts/stencils/beam/beam-template-describe.md:37

**Issue:** The stencil repeats heading syntax and the Merge danger structure after pointing at the shared spec.
PATTERN-stencil-ownership requires a producer pointer and forbids restating the format.
The constraint explicitly makes such violations BLOCKING.
Beam-specific content requirements can remain without duplicating the artifact structure.

### [BLOCKING] Exercise the feature through daemon scheduling

**Location:** internal/dobbyengine/describe_test.go:93

**Issue:** The harness directly invokes countLandings and describeStep.
Deleting the new invocation from tick would disable automatic descriptions without failing these tests.
The existing move and adopt-hold tests configure the tidy agent, not the describer.
PATTERN-test-speed requires behavior tests at the module surface rather than only its helpers.
The gap also leaves the new scheduling-hold behavior unchecked.

### [MEDIUM] An unreadable draft never reaches timeout cleanup

**Location:** internal/dobbyengine/describe.go:224

**Issue:** A directory at DraftPath, or a persistent permission error, makes draftFindings return an error.
settleDescribe returns before checking the deadline on every tick.
Describe remains set after expiry, preventing subsequent member landings from starting another attempt.
The error also returns from tick before tidyStep and landBeam.
The existing read-error test checks retention only before expiry.

### [MEDIUM] Separate record failure from publication failure

**Location:** internal/dobbyengine/describe.go:230

**Issue:** The real Record callback invokes rewriteDescription and then prUpdate.
recordDescription commits the replacement first.
A later push or GitHub edit failure therefore returns an error after the new summary has been recorded.
settleDescribe still says it was not recorded and that the previous description stands.
This contradicts both the actual state and the new Record seam's error contract.
The manual describe path already words this partial success accurately.

### [MEDIUM] Preserve the summary when writing the replacement fails

**Location:** internal/landingshed/beamdescribe.go:197

**Issue:** os.WriteFile truncates the destination before writing its contents.
A quota or I/O failure after truncation returns before the commit-failure restoration block.
The old summary can therefore become empty or partial on a failed rewrite.
The goal requires failed rewrites to leave the old description standing.

### [MEDIUM] Keep failed notices owed until delivery

**Location:** internal/dobbyengine/describe.go:231

**Issue:** Both the refused-record and timeout paths clear the attempt before calling d.post.
That helper logs a Notice failure and discards it.
No state remains from which another tick can retry delivery when creel recovers.
The new package documentation and landing summary promise an orch notice for these failures.
A delivery retry must not repeat a successful recording operation.

### [MEDIUM] Read the member PRs, not only their commit summaries

**Location:** contracts/stencils/beam/beam-template-describe.md:29

**Issue:** The goal requires reading the current summary, beam commits, and every member pull request.
The stencil requests the summary, board entry, commit list, diff, and member commit messages.
RenderBeamChanges supplies a number, title, timestamp, and shortened body text rather than the PR contents.
The instruction to list every PR in the output does not supply the missing reading step.

### [MEDIUM] Test the returned production wiring

**Location:** internal/beamcli/dobbydeps_integration_test.go:67

**Issue:** The test checks only a nonnil Record and positive Timeout.
A no-op recorder, missing gate, or wrong positive timeout would pass.
Other new tests call the facts, recording, and gate helpers directly, bypassing their production connection.
This is a coverage gap under the rubric, not a demonstrated failure of the current wiring.

### [MEDIUM] Pin preservation of existing busy work

**Location:** internal/dobbyengine/describe_test.go:148

**Issue:** The current assertion starts with busy false and a describer still waiting.
It does not test settlement while another source already made the daemon busy.
The reported mutant removing the prior busy operand therefore survives.
Assigning waiting alone would let a completed describer clear another active run's backoff protection.

### [MEDIUM] Correct the merge-danger account

**Location:** _lym/landing/summary.md#Merge danger

**Issue:** The summary calls the change two-way with a dobby-only blast radius.
The new daemon path commits descriptions, pushes the beam through prUpdate, and edits remote PR text.
It also adds fields to a state record other binaries can read.
The rubric expressly rejects a two-way claim for outward effects and record-format changes.
A code revert does not undo a PR body already published.

## Review checks

The manual edit-and-record route remains present.
The current landing notice and orch instructions no longer request a per-landing manual update.
The overview, package docs, registry entries, and parity producer are included in the diff.
The tests' prompt assertions concern substitutions and directives, not static prose.
No new real-time test wait was found.
Mutation survivors at describe.go:166 and :282 affect diagnostic logging and warrant no separate finding.
The removed follow-up test expectations implement the requested behavior change rather than relaxing a retained check.

The targeted build passed: go build ./internal/dobbyengine ./internal/beamcli ./internal/landingshed.
No test suite was run in this review, and no target file was changed.
The test lens's wiring and busy coverage findings are kept at MEDIUM under the rubric's coverage criteria.
The public-surface test violation remains BLOCKING under the explicit repository constraint.
