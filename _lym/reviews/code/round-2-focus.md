---
round: 2
focus:
  - "internal/runplan/postbatch.go PostBatchChecks and withoutDroppedCreates: when a decision record drops a Create target declared through a plan handle, does every check that sees the batch's cards, including BindHandles and its bind-count mismatch, treat the dropped declaration as absent, and does a test with a two-handle card cover it?"
  - "internal/runplan/postbatch.go PostBatchChecks and its callers in internal/websterengine (record-batch, recover-batch) and internal/weaverengine: when DroppedCreates fails on a read error or a malformed Added after Discussion heading, does the returned error name the record path and a way forward, and does a test reach that path?"
  - "internal/websterengine/recordbatch.go failCardNotDone and createTargetWayForward: is the --drops-create route offered only for a finding that dropping a Create target can actually clear, and does a rename-not-done finding get a way forward that works?"
  - "internal/burlerengine/prompt.go priorFindingsBlock and roundcheck.go namedByAmendment: does a pending amendment from an earlier snapshot still reach the reviewer's prompt, and does the amendment exemption admit only plan card files that an amending entry names as a whole token?"
  - "internal/burlerengine/roundcheck.go newFindings: is the origin webster exemption from the changed-file rule bounded to the one halt finding the prompt asks for?"
  - "contracts/stencils/burler (cluster fan and assist consolidation) and contracts/stencils/parent/parent-directive-parent.md: does the consolidation text preserve origin webster on the halt finding, and does the parent's fold rule keep a dropped Create across decision replace and a re-recorded decision add?"
  - "internal/shedadapters/burlerreview.go, internal/runplan/postbatch_test.go, internal/websterengine/recordbatch_test.go, internal/websterengine/template_test.go: do tests show the halt reason reaches only the Plan-Burler-Review profile, an undropped Create target still blocks beside a dropped one, a decision record that drops a Create is honoured by the record-batch and weaver paths, and the none-rendering of the parent decisions section is still asserted?"
  - "internal/websterengine/doc.go, internal/loomcli/decision.go Long text, internal/burlercli/validate.go, internal/discussionparser/append_test.go: do the package doc and help text state the rules as the code now holds them, does validate-review derive its plan run_subdir from the recipe instead of a second literal, and does a test the redundancy report lists as contained carry a keep marker where it pins what its covering test does not?"
---

The focus lists the sites the fixes touch, so the round-2 review checks them against the answer key as the diff leaves them.
