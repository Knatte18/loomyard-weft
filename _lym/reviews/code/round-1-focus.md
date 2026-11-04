---
round: 1
focus:
  - "internal/driftengine/accepted.go `testFiles` and `requireBaselineTest`: settle whether the name match (path relative to the profile directory, with or without extension) can admit a name that is not a baseline test or refuse a real one, e.g. a name that matches only a file's stem across extensions, a dotfile, or a name with a directory part."
  - "internal/driftcli/cli.go `acceptTest` with `driftengine.CheckAccept` and `Accept`: settle whether the acceptance capture, the check and the scope-record write run in an order and under a lock that leave no half state, such as a staged `AcceptingDir` left behind when the capture fails or `Accept` refuses, or two accepts sharing `AcceptingDir`."
  - "internal/driftengine/verbs.go `Accept` against `admitAcceptance`: settle whether the refactor keeps every earlier refusal (wrong row, running, withdrawn by agent, caller without ruling, no scope) with the same message and order, and whether `keepAcceptedOutput` inside `UpdateScope` can leave kept output without a matching scope entry or the reverse."
  - "internal/driftengine/verify.go `keepAcceptedProfile` with `buildAcceptedProfile`: settle whether the accepted profile now records each test's output as it was at acceptance (wanted item 2), whether the compare against it at HEAD handles tests accepted before this change or re-accepted after `Withdraw`, and whether `AcceptedProfileHead` is cleared or kept consistently."
  - "Scope against the decision record's amended Goal: `lym drift scope --skip` must stay unchecked, and nothing else of Drift changes. Settle whether the removed seams and fields (`CaptureAccepted`, `RunAcceptedCapture`, edits in `internal/battencli`, `internal/loomcli`, `internal/shedrecipe`, `internal/testkit/envkit`) are all consequences of the replacement and license no other behavior change."
  - "Tests in `internal/driftengine/verbs_test.go`, `verify_test.go` and `capture_test.go`: settle whether each new behavior (unknown-name refusal naming test and directory, output kept at acceptance, drift after acceptance halting Drift-Verify, withdraw dropping kept output) has coverage, and whether any new test repeats an existing one."
  - "Doc and comment compliance: `internal/driftcli/doc.go`, `internal/driftengine/doc.go`, the `accept` Long text and changed doc comments state the new behavior and carry no stale reference to the removed capture seam."
---

Seed focus from the decision record's amended Goal (accept check only, accepted profile kept at acceptance time) and a read of the verbs, accepted-profile and verify diffs.
