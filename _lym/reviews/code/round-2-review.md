---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/landingshed/describereview.go#check
    summary: "The Warn that the round's F5 fix added in describeReview.check names the key through keyName(\"describe_review_timeout_min\"), a string literal, not a named constant. The same literal is repeated in config.go's ConfigSpec map and LoadConfig's positive-value list. PATTERN-config-key-names requires a key named in a message to be rendered by the key-name helper from named constants, and PATTERN-code-conventions forbids adding a repeated literal for a name that can go stale."
    fix: "In internal/landingshed/keynames.go, add describeReviewTimeoutMinKey = \"describe_review_timeout_min\" to the const block of landing.yaml key paths, beside forbiddenTermsKey and describeReviewKey. Use it at three sites: in describereview.go's check Warn (keyName(describeReviewTimeoutMinKey)), as the key in config.go's ConfigSpec map entry for describe_review_timeout_min, and as the key of the describe_review_timeout_min row in LoadConfig's positive-value list. Leave the yaml struct tag as it is."
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/landingshed/prreview.go#syncText
    summary: "Commit 8afc7d497 makes a pull request text that cannot be built hold the review open: judge returns nil while textStale is set, so a GitHub approval and the operator's lym loom pr-continue are both refused for as long as the description stays unbuildable, which can be forever. The only message is the one Warn syncText logs per episode. It still reads \"no pull request text could be built, so the pull request keeps its title and body\": it does not say that the review's ending is held, and it names no way forward. PATTERN-refusal-way-forward requires a loom refusal's message to name the way forward, with a test that reaches it."
    fix: "In PRReview.syncText (internal/landingshed/prreview.go), reword the text-build failure Warn so it states that the review stays open and names the way forward. For example: \"landingshed: no pull request text could be built, so the review stays open and the pull request keeps its title and body; way forward: fix the change description so lym loom validate-description passes, and the next poll sets the text and lets an approval or lym loom pr-continue end the review\", keeping the row and cause fields. In internal/landingshed/prreview_wait_test.go's row ADescriptionThatYieldsNoTextHoldsTheContinuedEnding, capture the log and assert that the Warn carries that way-forward clause."
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/landingshed/describereview.go#describeReviewSeat
    carries: F5
    summary: "The fix gave the reviewer the writer's budget, but both runs start at the table's spawn, so the review only gets whatever time the writer leaves. A writer that releases the reviewer at 28 of its 30 minutes leaves the reviewer 2 minutes, and the writer's own 30-minute deadline also ends the turn-end hold at that moment. The deadline then finalizes Done through the entry's Final pass, so the text lands unreviewed and only a log line records it. landing.describe_review_timeout_min therefore bounds nothing once the writer has used more than half of its budget. The amended entry asks that the Describe text be reviewed inside the Describe row."
    fix: "Give the review time on top of the writer's budget. In describeReviewSeat (internal/landingshed/describereview.go), set the reviewer seat's Timeout to time.Duration(cfg.DescribeTimeoutMin+cfg.DescribeReviewTimeoutMin) minutes. In DescribeTable (internal/landingshed/describe.go), set table.Seats[0].Timeout, the lead's, to the same sum, so the writer's deadline does not end the hold before describe_review_timeout_min has run. Update the describe_timeout_min and describe_review_timeout_min comments in internal/landingshed/template.yaml, the DescribeTimeoutMin and DescribeReviewTimeoutMin field docs in config.go, and the reviewer bullet and design decision in _lym/landing/summary.md to state the sum. Extend TestDescribeTable_ReviewerSeat to assert both seats' timeouts equal the sum, using a literal value."
  - id: F4
    severity: LOW
    class: design
    location: internal/landingshed/beamdescribe_test.go#TestReplaceBeamDescription
    carries: F8
    summary: "Of the paths F8 named, the beam recorders remain untested with a non-nil pattern. TestReplaceBeamDescription passes nil for forbidden in every row, so ReplaceBeamDescription could ignore the parameter and still pass. Nothing exercises hubLandDeps in internal/beamcli/deps.go either: it is the code that loads landing.yaml and passes cfg.ForbiddenTerms() to RecordBeamDescription, ReplaceBeamDescription and the DescriptionFindings seam, and no test fails if that forwarding is dropped. The change description claims the check runs in lym beam describe's recorders."
    fix: "In internal/landingshed/beamdescribe_test.go, add a forbidden field to TestReplaceBeamDescription's table and a row whose draft is valid but holds a line one pattern matches. Assert a forbidden-term refusal naming the line, no commit, and the description at path left as it stood. In internal/beamcli, add a test that writes landingshed.ConfigTemplate() as the hub's landing.yaml (as TestBeamDescribeDeps_CarriesTheForbiddenTerms does). It drives hubLandDeps' description recorder and its DescriptionFindings seam over a description holding \"lym\", and asserts the forbidden-term refusal and finding."
  - id: F5
    severity: LOW
    class: scope
    location: contracts/stencils/darn/darn-template-write.md:55
    carries: F11
    summary: "The rule now reaches every brief that commits code, but not the beam describer. contracts/stencils/beam/beam-template-describe.md writes the beam's pull request title and body and never tells the agent to keep the tooling's names out. The amended entry (#255) asks that the Describe step be told not to name lym, LoomYard, gates or other tasks in a pull request body. The beam describer learns the rule only from its gate's refusal, and only for the terms the patterns cover."
    fix: "In contracts/stencils/beam/beam-template-describe.md, add the rule to the description-writing instructions: the reader knows the repository and nothing of the tooling, so the description names nothing of it that the repository itself does not hold (LoomYard or lym, the board or a board entry, the task, the Sketch or the decision record, the parent or an orch, the scoper, a gate, a tier or a perch), never points at the scope by its numbering, and states the requirement itself. Word it as landing-template-describe.md does."
prior:
  - id: F1
    status: resolved
    answer: "validate.go's message names landing.yaml and no key; the LoadConfig error it wraps names its key through keyName."
  - id: F2
    status: resolved
  - id: F3
    status: resolved
    answer: "The message ends with a way-forward clause, and TestValidateDescriptionCmd/UnreadableForbiddenTerms asserts it and exit code 1."
  - id: F4
    status: resolved
    answer: "check writes describe-review.ready on its first call. evaluateGate reaches that call only after the description entry passed. The reviewer awaits ready_path, and ResetDescribeReview removes the record at a fresh spawn."
  - id: F5
    status: open
    answer: "The fixed-differently reason is accepted for how the wait is bounded. What remains open is that both runs share the spawn time, so the review only gets what the writer leaves; carried as F3."
  - id: F6
    status: resolved
  - id: F7
    status: resolved
  - id: F8
    status: open
    answer: "Rows now cover the shedrecipe gate, validate-description and the beam describer; the beam recorders' forwarding is still untested, so this is carried as F4."
  - id: F9
    status: resolved
  - id: F10
    status: resolved
  - id: F11
    status: open
    answer: "Every code-committing brief carries the fragment; the beam describer's pull request body stencil does not. Carried as F5."
  - id: F12
    status: resolved
    answer: "The sent record is written through GateResult.Reprompted once the re-prompt is delivered, and TestDescribeReviewGateEntry covers rebuild, reset and undelivered rows."
  - id: F13
    status: resolved
    answer: "syncText runs under the continue flag, and judge holds both an approval and the continue flag while textStale is set; the hold's missing way forward is the new F2."
  - id: F14
    status: resolved
  - id: F15
    status: resolved
  - id: F16
    status: resolved
  - id: F17
    status: resolved
  - id: F18
    status: resolved
  - id: F19
    status: resolved
  - id: F20
    status: resolved
  - id: F21
    status: resolved
---

### [BLOCKING] A key named in the review's timeout Warn is a repeated literal, not a named constant

**Location:** `internal/landingshed/describereview.go#check`

**Issue:** Commit fd21b914e added `keyName("describe_review_timeout_min")` to the Warn logged when the review's time runs out.
The key path is a string literal, and the same literal also appears as the `ConfigSpec` map key (`internal/landingshed/config.go:158`) and in `LoadConfig`'s positive-value list (`config.go:240`).
`keynames.go` already declares a constant for every other landing.yaml key path this package's messages cite, including `forbiddenTermsKey` and `describeReviewKey` from this change.
PATTERN-config-key-names requires a named key to be rendered "from named constants", and PATTERN-code-conventions says never add another repeated literal.

### [BLOCKING] An unbuildable pull request text holds the review open with no way forward

**Location:** `internal/landingshed/prreview.go#syncText`

**Issue:** When `pullRequestText` fails (the description no longer parses, or `completeSummary` fails), `syncText` sets `textStale` and `judge` returns nil before the continue and approval branches.
The wait then keeps refusing the operator's `lym loom pr-continue` and an approval on GitHub until the file is fixed, which may never happen, since nothing tells anyone.
The only signal is the Warn logged once per episode, which still says the pull request "keeps its title and body".
It does not say that the review's ending is held, and it names no way forward.
That breaks PATTERN-refusal-way-forward for a loom refusal.
The focus asked whether a wait added for the stale text can fail to end: it can, and it does so silently.

### [MEDIUM] The review only gets the time the writer leaves of its own budget

**Location:** `internal/landingshed/describereview.go#describeReviewSeat`

**Issue:** The reviewer seat's `Timeout` is `describe_timeout_min` (30).
The writer's run also ends at 30 minutes, and both start at the table's spawn.
The reviewer is released only when the writer first ends a turn on a text that passes, so all the review gets is the writer's leftover budget.
The writer's deadline also ends the turn-end hold.
`classifyDeadlineExpiry` finalizes Done while the description exists, and the entry's `Final` closure passes.
A writer that uses most of its budget therefore lands its description unreviewed, and `describe_review_timeout_min` never comes into play.
The focus asked whether the time budget covers a writer that uses its whole budget: it does not.

### [LOW] The beam recorders' forbidden-term forwarding is untested

**Location:** `internal/landingshed/beamdescribe_test.go#TestReplaceBeamDescription`

**Issue:** `TestReplaceBeamDescription` passes `nil` for `forbidden` in every row.
`hubLandDeps` in `internal/beamcli/deps.go` loads landing.yaml and forwards `cfg.ForbiddenTerms()` to `RecordBeamDescription`, `ReplaceBeamDescription` and `DescriptionFindings`, but no test drives it.
If any of those forwards were dropped, no test would fail.

### [LOW] The beam describer is not told to keep the tooling's names out of the pull request body

**Location:** `contracts/stencils/darn/darn-template-write.md:55`

**Issue:** `contracts/stencils/beam/beam-template-describe.md` writes a beam's pull request title and body.
Unlike `landing-template-describe.md`, it says nothing about the tooling's names.
#255 asks for the Describe step to be told this rule for a pull request body.
The carry rules keep the location on F11's file; the fix site is the beam stencil.

## Rejected

- The reminder in `internal/shuttleengine/hold.go` is typed at the first held end of an operator-typed turn, and once per run. That is not a misfire. `sendVerified` presses Enter only into a box that holds exactly the sent text. The line tells the agent to defer to the operator. The turn the reminder starts is notified to the parent as any other turn if it ends held. `remindOfMissingOutputs` skips covered, usage-limited, background-waking and unstarted-prompt runs, and a failed send clears the flag so the reminder is retried.
- The reviewer reads only a text that passed the mechanical gate. `evaluateGate` stops at the first failing or pending entry, and every recipe's Describe row has a must-pass `description` entry ahead of `Describe-Review`, so the ready record is written only after a pass. At a fresh spawn, the seat engine's `clearAssistantFiles` removes the stale marker and report, and `ResetDescribeReview` removes the ready and sent records.
- Findings are sent once across a re-attach. The sent record is written from `handleGatedBoundary`, only after the re-prompt was delivered, and a rebuilt entry passes once the record exists.
- Tests of the forbidden-term forwarding and the gate order fail if the forwarding or the order is dropped. The shedrecipe gate test, `TestValidateDescriptionCmd`, `TestBeamDescribeDeps_CarriesTheForbiddenTerms`, `TestWire_DefaultConfig`, `TestWireLightweight_*` and `TestDescribeEntry_HappyPath`'s gate-order assertion each check this. The beam recorders are the exception, raised as F4.
- No commit in the range carries a `Root-Cause:` trailer, although the record includes bug reports (#228, #241, #244). This round is incremental, so it can raise only BLOCKING findings on changed hunks, and a missing trailer is not one.
- The package tests of landingshed, shedrecipe, summaryparser, beamcli and shedadapters pass (`lym gate test`, tier0: 3077 passed, 0 failed).
