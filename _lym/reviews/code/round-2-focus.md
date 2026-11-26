---
round: 2
focus:
  - "internal/gatecli/counters.go, observeSummary and the VSTest counter around it: where does an open `Total tests:` summary end? Check lines interleaved from another project or stderr, a missing `Total time:` line, an invocation boundary with and without a results file, a one-line summary and a failure block running into a summary, for counts that are lost, doubled or attributed to the wrong invocation, and for failure text that takes in summary lines."
  - "internal/gatecli/tierrun.go, withTimeoutMin and its callers there and in internal/gatecli/copies.go: does every bound taken from a tier's timeout_min count wall-clock time, end its goroutine and report its cause correctly, and do the tests fail if either the wall-clock or the monotonic reading is dropped?"
  - "contracts/stencils/burler/burler-template-fixer.md, the text on what the fixer does when `lym gate test` refuses: for each kind of refusal planning can return, does the text lead the fixer to the right step (follow the way forward itself, or defer), without contradicting the rest of the stencil?"
  - "internal/gatecli/keynames.go, internal/gateslot/keynames.go and the refusals in internal/gatecli/tiers.go and internal/gateslot/tiers.go: do the refusals the diff adds or changes render config key names as PATTERN-config-key-names and PATTERN-code-conventions require, and do tests pin the rendered keys?"
  - "internal/gatecli/tiers.go, planTierRuns and takeCallerFilters: how is each shape of a caller `--filter` after `--` handled, including a trailing one with no value, a repeated one and a command with its own --filter, and do the doc comment of planTierRuns and the refusal list in internal/gatecli/doc.go match what it refuses?"
---
