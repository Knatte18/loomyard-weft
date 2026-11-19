---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/loomcli/prfindings_integration_test.go:37
    summary: The regression test bypasses the required public command surface
    fix: Replace the direct prReviewSendBackEdge invocation with clihelp.ExecuteIn over Command() and the fixture worktree, invoking pr-findings with valid blocking findings and a seeded pull-request record. Assert the successful send-back envelope and stored record, retaining real landing construction. Replace the helper test rather than adding a duplicate.
    origin: lens:code-tests
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/loomcli/prfindings_integration_test.go:24
    summary: The fixed pair starting state bypasses the required template fixture
    fix: "Copy hubforge.Shape{Anchor: \".\", Pairs: []string{hubforge.TemplatePairSlug}}, use hubforge.TemplatePairSlug for the fixture paths, and remove the per-test AddPair call and custom slug. If the custom slug is necessary, build it in a package-level hubforge.Registration and use CopyRegistered instead."
    origin: lens:code-affected-set
---

### [BLOCKING] Exercise the command connection

**Location:** internal/loomcli/prfindings_integration_test.go:37

**Issue:** `PATTERN-test-speed` requires an unexported helper to be tested through the public surface.
The new test calls the helper directly, while `TestPRFindingsVerb` injects its edge.
Both would pass if the command reverted to its old unwired closure while the extracted helper remained correct.
The affected-set lens independently reported the same defect; it is consolidated here.

### [BLOCKING] Reuse the pair template

**Location:** internal/loomcli/prfindings_integration_test.go:24

**Issue:** `PATTERN-test-isolation` requires fixed-slug pair starting states to come from registered templates.
The new test copies an empty hub and adds its fixed pair on every execution.
`hubforge.Shapes` already supplies a pair-carrying shape, so this setup needs no custom pair construction.

## Review evidence

The production method fills landing dependencies before shed construction and preserves error propagation.
Run and step already wire those dependencies; status and pause leave their shed builder unset.
PR-Review session and record-watch seams remain deferred closures.
Sampler wiring opens a process-lifetime record watch, but no correctness defect was established for this command.
The changed documentation and implementation trace to the board goal.

The integration assertion distinguishes the missing wiring by inspection of Finalize's constructor.
Its positive budget matches the template's Code-Review budget.
Execution could not be confirmed: the foreground tier1 gate failed before testing because the sandbox denied writing
the gate-holder record under the hub's `_bolt/.lym/gate` directory.
This infrastructure limitation is not a finding against the diff.
