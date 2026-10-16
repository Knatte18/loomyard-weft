---
round: 2
ledger:
  - key: d3-created-dir-bound-misses-custom-and-member-glyph
    rounds: [1]
    status: resolved
  - key: d5-refusal-spec-step-interrupted-row-not-in-scope
    rounds: [1]
    status: resolved
  - key: d5-bound-leaves-resume-and-reseed-unstated
    rounds: [1]
    status: resolved
  - key: d5-no-restep-rationale-covers-decode-only
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-miss-d2-d7-and-test-unstated-behaviour
    rounds: [1]
    status: resolved
  - key: d5-root-cause-sentence-belongs-in-support-log
    rounds: [1]
    status: resolved
  - key: d5-decode-error-way-forward-waits-on-unhealing-file
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: d5-new-stop-causes-keep-transient-class
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: d3-rename-destination-has-no-acceptance-criterion
    rounds: [2]
    status: open
    class: scope
    severity: NIT
---

All six round-1 keys are resolved: the round-2 review confirmed D3's created-directory set, the `step interrupted` row in scope, the resume and reseed bound, and the acceptance criteria for D2 and D7.
Round 2 opened three new non-gating keys, all on D5 and D3: the decode-error way forward, the class of the new stop causes, and a missing Rename acceptance case.
