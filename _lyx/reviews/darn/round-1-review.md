---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: contracts/stencils/orch/orch-template-role.md:161
    summary: The board policy still sends a darn run's decision by message to the Darn-Write session, though the board entry says the board policy drops the separate message step.
    fix: In contracts/stencils/orch/orch-template-role.md, rewrite line 161 (the sentence after "Only when the verb refuses because Discussion-Write is running ...") so a darn run's decision, once Darn-Write has committed its record, is recorded with the verb alone and needs no message of the orch's own, since the verb types the notice line into the live agents' panes. Keep line 160 as it is, because the verb still refuses while Discussion-Write owns the record.
    origin: handler
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/loomcli/decision.go:446
    summary: The signing split in decisionNoticeClient (program client for an orch or unnamed caller, strand client for a named non-orch caller) has no test, so a swapped branch passes the suite.
    fix: In internal/loomcli, add rows to a table-driven test of the claim choice, reached through an export_test.go seam or a returned claim kind, for the callers hub:orch, an empty name, an unparsable name and hub:task-a:arbiter, each asserting which kind of client decisionNoticeClient builds. Extend an existing test if one already covers the creel client constructors.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: design
    location: internal/loomcli/decision.go:446
    summary: A caller whose strand name parses as a non-orch strand but who runs in no strand pane gets a strand claim that creel refuses, with no program fallback, so every notice fails.
    fix: In the notify closure in preRun and decisionNoticeClient, when the strand client's post fails with creelengine.ErrStrandClaimOutsidePane, post once more through creelcli.NewProgramClient(target, loomProgram), as orchcli's notifyPrime does. Add a row for it to the test of F2.
    origin: lens:code-correctness
  - id: F4
    severity: LOW
    class: consistency
    location: internal/loomcli/decision.go:483
    summary: The help text edit also rewrites two existing paragraphs unrelated to the notice, dropping the sentence that bounds the append to a validated record and the clause that the bullet is checked with the record.
    fix: In the decision add Long text, restore the dropped sentence "That bounds the append to a record the writer validated and has not touched, and to an entry that reached the fabric." and the clause ", the bullet is checked with the record" unless they are what keeps the text under the long-help cap. If the cap forces a cut, state in the change description which cap applies and trim only the notice paragraph.
    origin: handler
---

### [MEDIUM] Board policy keeps the separate message step for a darn run

**Location:** contracts/stencils/orch/orch-template-role.md:161
**Issue:** The board entry's Open item says the Fixing a run section and the board policy both drop the separate message step. The diff changes only line 109, in Fixing a run. Line 161 in the board policy still reads that a darn run's decision goes by message to the Darn-Write session once Darn-Write has committed its record. After this diff the verb succeeds on a darn run and notifies that session itself, so the message is the step the entry asked to drop. Line 160 stays correct, since the verb still refuses while Discussion-Write owns the record.

### [MEDIUM] The signing split has no test

**Location:** internal/loomcli/decision.go:446
**Issue:** The summary and the record's D3 claim a notice signs as the LOOM program from an orch pane or an unnamed shell and as the caller's own strand otherwise, because creel refuses a program claim from a non-orch pane (internal/creelengine/identity.go:57-67). The tests stub notify wholesale, so no row reaches decisionNoticeClient. Swapping its branches, or making the arbiter's every notice fail, passes the suite.

### [LOW] No program fallback when a strand claim is refused outside its pane

**Location:** internal/loomcli/decision.go:446
**Issue:** A caller whose LYX_STRAND_NAME parses as a non-orch strand but who runs in no strand pane, such as an operator shell with a leaked variable, gets a strand claim that creel refuses with ErrStrandClaimOutsidePane (identity.go:71-76). orchcli's notifyPrime falls back once to the program in that case. Here each notice fails and is reported under notice_errors; the verb still exits 0, so the effect is a lost notice only.

### [LOW] Help text trimmed beyond the notice

**Location:** internal/loomcli/decision.go:483
**Issue:** The summary says the help was condensed to stay under the long-help cap. The diff removes two statements about the darn bound and the pattern bullet check that have nothing to do with the notice. I found no cap defined in the code to check the claim against.

## Rejected

- Two live run records sharing a strand name would be posted to twice (code-correctness NIT): no code path produces a duplicate name.
- The empty notified assertion goes through fmt.Sprint and cannot tell an empty list from null (code-tests NIT): notifyLiveRuns always returns non-nil slices, so the criterion holds.
- Receiver selection notifies every live shuttle run of the worktree and not one producer strand: the focus asked for a verdict, and the decision record's D1 gives the reason, with the entry's single strand not derivable from Go's records. Filter checked against the code: tackler, arbiter, caller and unparsed strands are skipped, and the darn confirmation gate runs before the post. Not a defect.
- Failure semantics and refusal paths: every refusal returns before notifyLiveRuns, and a failed listing or post leaves exit 0 with the entry committed. Not a defect.
