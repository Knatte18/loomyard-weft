---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/validate.go:126
    summary: "The new validate-description refusal spells the key as the literal string \"landing.describe_forbidden_terms\", which PATTERN-config-key-names forbids: a key named in a message is rendered by configengine's one key-name helper from named constants."
    fix: "In internal/loomcli/validate.go:126, stop spelling the key. Either word the message without a key name, for example \"loom: read the landing config: \"+err.Error() (the LoadConfig error already names its key through keyName), or export from landingshed a function returning keyName(forbiddenTermsKey) and build the message from it."
  - id: F2
    severity: BLOCKING
    class: consistency
    origin: lens:code-affected-set
    location: internal/landingshed/describereview.go#describeReviewSeat
    summary: "The new error fmt.Errorf(\"%s model-spec: %w\", describeReviewKey, err) names the key by its short name describe_review, not in full as landing.describe_review, against PATTERN-config-key-names. The same function renders the full name with keyName for its SkipReason."
    fix: "In describeReviewSeat, replace describeReviewKey with keyName(describeReviewKey) in the resolveModelChoice error, so the message reads \"landing.describe_review model-spec: ...\"."
  - id: F3
    severity: BLOCKING
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/validate.go:123
    summary: "The new loom refusal for forbidden terms that cannot be read, reachable through lym loom validate-description, has no trailing way-forward clause of its own and no test that reaches it and asserts its message, as PATTERN-refusal-way-forward requires of a bound module."
    fix: "In internal/loomcli/validate.go, end the DescriptionTerms error message with a way-forward clause, such as \"; way forward: fix landing.yaml with lym config landing from the prime, then run lym loom validate-description again\". Add a row to the validate-description test in internal/loomcli that wires c.env.DescriptionTerms to return an error, then asserts the message, its way-forward clause and the non-zero exit."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: contracts/stencils/landing/landing-template-describe-review.md:24
    summary: "The reviewer starts on the description as soon as lym shuttle await sees the file, which can be before the description gate passed and before the writer's later rewrites. The amended entry asks that the mechanical gate stay in front of the reviewer. describeEntry's new order puts the description gate ahead only of the writer's own Describe-Review entry, so summary.md:31's claim that the reviewer only reads a text that passed the mechanical check is false (summary.md:57 contradicts it)."
    fix: "Make the gate entry release the reviewer. In describeReview.check (internal/landingshed/describereview.go), write a ready marker on the entry's first call, for example describe-review.ready beside the report through describeReviewPaths. evaluateGate reaches that call only after the row's description gate passed at a writer turn end, while the writer is held idle. Pass the marker's path into the table's values, and have landing-template-describe-review.md await it instead of {{.description_path}}. Remove the marker at a fresh spawn, through a prepareFreshSpawn passed to NewMultiLLMProducerSourced in describeEntry. Then rewrite summary.md:31 and :57 to match."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/describereview.go#describeReviewSeat
    summary: "The reviewer's describe_review_timeout_min (default 15) runs from the table's start, beside the writer's 30, and covers the wait for the description. A writer slower than the reviewer's budget leaves the text unreviewed, and check logs that only at Info. template.yaml:8's \"the longest the writer waits for its report\" is false."
    fix: "Bound the writer's wait from when it starts. Give the reviewer seat the writer's budget, time.Duration(cfg.DescribeTimeoutMin) minutes. In describeReview.check, record when the entry first ran (the moment F4's ready marker is written), and pass once describe_review_timeout_min has elapsed since then with no end marker, logging the unreviewed pass at Warn. Make the reviewer timeout reachable from describeReviewGateEntry for this. Keep template.yaml:8 and docs/overview.md:1438 as they read, since they then hold."
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:72
    summary: "The Merge danger call does not hold. The door is called two-way, but the change pushes outward: PR-Review now overwrites a body edited on GitHub at each poll, and Finalize edits recorded pull requests before it merges. It also adds landing keys that an older binary's strict load refuses once a hub file sets one, which summary.md:65 tells the operator to do. The blast radius, landing, names less than the shedadapters MultiLLMProducer change can break: every seat row, namely Discussion-Write, Plan-Write, Parts, Raddle, the darn writer, Drift-Expect and the Drift-Verify judge."
    fix: "In _lym/landing/summary.md's Merge danger section, call the door one-way, naming the outward pull request edits and the new landing keys that an older binary refuses once set. Call the blast radius with a word that covers every seat row, such as loom, rather than landing."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shedrecipe/entries_describe_test.go#TestDescribeEntry_HappyPath
    summary: "No test pins that describeEntry puts the row's gates ahead of the table's own Describe-Review entry and keeps that entry. The test's table source carries no gate, so reverting to table.Gate = gate, or reversing the order, passes unchanged."
    fix: "In TestDescribeEntry_HappyPath, make the fake DescribeTable return a table whose Gate holds one named entry, then assert the resolved table's gate names, in order, the description entry and that entry."
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shedrecipe/entries_gate.go#resolvePlainGateClosure
    summary: "No test shows the configured landing.describe_forbidden_terms reaching any path that checks a description. Those paths are the row's description gate through Env.DescriptionTerms (and its error path), lym loom validate-description, the beam describer's gate and findings, and the beam recorders. Every test passes nil or a closure returning nil."
    fix: "Add one row per seam with a non-nil pattern and a description line it matches, asserting a forbidden-term finding. Cover: the shedrecipe description gate built from an Env whose DescriptionTerms returns a pattern, plus a row where it returns an error; loomcli validate-description with DescriptionTerms returning a pattern; and beamcli's describeGate and descriptionFindings with a pattern."
  - id: F9
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: contracts/stencils/loom/loom-rubric-code-review.md
    summary: "#255's second What-to-do item, have Code-Review check documents and the pull request body for such names, is not built. The entry's carve-out covers only items 3 to 6 of #225, and no Added after Discussion entry records the omission. summary.md:61 states it is not changed."
    fix: "Add an Also flag bullet to contracts/stencils/loom/loom-rubric-code-review.md. It flags a repository document the diff adds or changes, or the change description, that names the tooling which wrote the change (LoomYard, lym, the board or a board entry, the Sketch, the decision record, another task, a gate or tier of that tooling, or what the parent recorded), unless the repository itself holds those names. Then update summary.md:61."
  - id: F10
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: internal/shuttleengine/hold.go
    summary: "#241's suggested fix is not built. A turn end held for missing output files after an operator-started turn still waits silently until the timeout, with no reminder to the agent and no visible notice. Only the classification half, a resumable Stuck, is built (summary.md:59-60)."
    fix: "In internal/shuttleengine/hold.go, where a held turn end of an operator-started turn currently sends no hold notice, send the agent one reminder naming its missing output files, or show the operator a visible notice in the pane. Add a test that drives an operator-started held turn and asserts the reminder or notice. Then update summary.md:59-60."
  - id: F11
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: contracts/stencils/darn/darn-template-write.md:9
    summary: "#255's first item asks that the Describe step and any step that writes repository documents be told not to name lym, LoomYard, gates or other tasks. The diff tells only the Describe writer and the darn writer, and the darn sentence covers only the change description, not the repository files that writer changes."
    fix: "Add the rule, that a repository file names nothing of the tooling which wrote the change that the repository itself does not hold, to every stencil of a step that writes repository files: the darn writer's code and docs instructions, not only its description paragraph, the implementer (webster) stencil, the burler fixer stencil and the landing merge fixer stencil."
  - id: F12
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/describereview.go#describeReview
    summary: "The sent state lives only in memory, and describeEntry builds a new entry at every call. After a re-attach, a done marker and a findings report from earlier in the session send the same, possibly already fixed, findings to the writer again. GateEntry.PassOnCap's contract asks that a request which must not repeat make its own closure idempotent."
    fix: "Persist the sent state in describeReview.check: write a describe-review.sent file in the review directory before returning the findings, treat its presence as sent, and remove it at a fresh spawn together with F4's ready marker."
  - id: F13
    severity: LOW
    class: scope
    origin: lens:code-scope
    location: internal/landingshed/prreview.go#wake
    summary: "#228 asks to refuse to end a round while the pull request's text and the description differ. The wait still ends a round on stale text in two ways. A continue flag skips syncText and endReview ends it. A transient edit failure is swallowed, and judge can then end it Done on an approval. Finalize sets the text before a non-beam merge, so only the round itself ends unsynced."
    fix: "In PRReview.wake, run syncText for an open pull request whatever rec.Continue says. In judge, keep waiting rather than ending Done or ending the review while the last syncText edit failed transiently, which a flag on prWaitState can carry. Add a row for each path."
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/specs/loom-final-summary-spec.md:56
    summary: "The new Forbidden terms section was inserted ahead of the Merge danger section's closing sentence, \"Publish and Finalize do not check it: they carry the body as it stands.\" That sentence now sits under Forbidden terms, where \"it\" reads as the term check."
    fix: "Move that sentence back to the end of the Merge danger paragraph, before the Forbidden terms heading. Add a sentence of its own to the Forbidden terms section if Publish and Finalize skip the term check too."
  - id: F15
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/landingshed/config_test.go#TestLoadConfig_InvalidKeys
    summary: "No LoadConfig row loads an empty describe_review, the documented way to run no reviewer. Dropping the guard that skips its model-spec parse would fail every such hub at load, and no test would notice."
    fix: "Add a test case that loads the well-formed landing.yaml with describe_review set to an empty string. Assert that LoadConfig returns no error and that cfg.DescribeReview is empty."
  - id: F16
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/landingshed/finalize_pr_test.go#TestFinalize_PullRequest
    summary: "setRecordedText's failed status read has no row, nor does a transient edit failure."
    fix: "Add two TestFinalize_PullRequest rows on a recorded open pull request. In the first, the second Status call is scripted with a non-transient error; want a halt whose reason names \"read the pull request's status\" and no merge call. In the second, the Edit is scripted with context.DeadlineExceeded; want an error and no merge call."
  - id: F17
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/landingshed/describereview_test.go#TestDescribeReviewGateEntry
    summary: "The error paths of the Describe-Review gate entry and of the report gate have no row: an end marker that cannot be read, a report read error other than not-exist, and describeReportGate over a missing report."
    fix: "Add rows to TestDescribeReviewGateEntry: one with a directory at the marker path, and one with a done marker and a directory at the report path; each expects a non-nil error. Add a row to TestDescribeReportGate with no report file that expects an error."
  - id: F18
    severity: NIT
    class: consistency
    origin: lens:code-correctness
    location: internal/landingshed/describereview.go#describeReviewGateAttempts
    summary: "The comment says the second attempt is the margin a re-attached run starts with. Failure counts live in memory and reset to zero at an attach, so no such margin exists."
    fix: "Rewrite the comment to say the entry fails once at most, so one attempt is spent. Drop the re-attach margin claim."
  - id: F19
    severity: NIT
    class: design
    origin: lens:code-correctness
    location: internal/shedadapters/multillm.go#mapOutcome
    summary: "The chair-timeout Stuck pointer drops GateAttempts and GateSpans from chair.Gate. A run held on a pending gate until its deadline then reports no gate attempts or held span, although the Done and gate-failed branches carry them."
    fix: "In mapOutcome's timeout branch, return shedengine.OutputPointer{Reason: reason, GateAttempts: gateAttemptsPointer(chair.Gate), GateSpans: gateSpansOf(chair.Gate)}."
  - id: F20
    severity: NIT
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/config.go#LoadConfig
    summary: "forbiddenTermFindings counts an empty FindString result as no match. A configured pattern whose leftmost match is empty, such as lym| or (x)?lym, therefore admits every line that holds the term."
    fix: "In LoadConfig's describe_forbidden_terms loop, refuse an entry whose compiled pattern matches the empty string (pattern.MatchString(\"\")), with the same way forward as the compile error."
  - id: F21
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/wiring.go:367
    summary: "The wireLightweight comment still says it loads no module config. The new DescriptionTerms closure loads landing.yaml strictly on that path when validate-description runs."
    fix: "Reword the comment at internal/loomcli/wiring.go:367 to say that only validate-description loads a module config on this path, landing.yaml for its forbidden terms, and only when it runs."
---

### [BLOCKING] validate-description's refusal spells a config key literally

**Location:** internal/loomcli/validate.go:126

**Issue:** The message is built as `"loom: read landing.describe_forbidden_terms: "+err.Error()`. PATTERN-config-key-names says a key named in a message is rendered in Go by configengine's one key-name helper, from named constants. The constant `forbiddenTermsKey` already exists in `internal/landingshed/keynames.go`.

### [BLOCKING] describeReviewSeat names describe_review by its short name

**Location:** internal/landingshed/describereview.go#describeReviewSeat

**Issue:** `fmt.Errorf("%s model-spec: %w", describeReviewKey, err)` renders `describe_review`, not `landing.describe_review`. PATTERN-config-key-names requires the full name. The same function already uses `keyName(describeReviewKey)` for the seat's SkipReason.

### [BLOCKING] New loom refusal has no way forward and no reaching test

**Location:** internal/loomcli/validate.go:123

**Issue:** When `c.env.DescriptionTerms()` fails, validate-description exits with the wrapped LoadConfig error. That error carries a way forward for some failures, such as the regex and below-1 checks, but not for a model-spec parse error. PATTERN-refusal-way-forward binds loom modules: each refusal names its way forward, and a new refusal lands with a test that reaches it and asserts its message. No loomcli test reaches this one.

### [MEDIUM] The reviewer is not behind the mechanical gate

**Location:** contracts/stencils/landing/landing-template-describe-review.md:24

**Issue:** The reviewer and the writer start together. The reviewer's prompt runs `lym shuttle await {{.description_path}}`, which returns once the file exists (`internal/shuttlecli/await.go`), and the reviewer then reads the text. That can happen before the description passes `ValidateDescription`, and before the writer rewrites it. The writer's Describe-Review entry hands those findings over whatever the text has become.

The amended entry says "The mechanical gate stays in front of it". `describeEntry`'s new order only orders the writer's own turn-end checks. The change description's claim at `_lym/landing/summary.md:31`, "so the reviewer only reads a text that passed the mechanical check", is not borne out, and line 57 of the same file contradicts it. The same claim appears in commit 2fcc25f7's message.

### [MEDIUM] The reviewer's budget runs from the row's start

**Location:** internal/landingshed/describereview.go#describeReviewSeat

**Issue:** The seat's Timeout is `describe_review_timeout_min` (15 by default), counted from the table's start. That budget includes the reviewer's wait for the writer, who has 30.

If the writer takes more than about 15 minutes, the reviewer times out before it reads anything. `check` then passes on the non-done marker and logs the unreviewed pass only at Info. The text lands unreviewed, which defeats the entry's item 1.

`template.yaml:8` calls the value "the longest the writer waits for its report". That holds only as an upper bound: what the writer waits is whatever is left of the 15 minutes.

### [MEDIUM] Merge danger: the two-way door and the landing blast radius do not hold

**Location:** _lym/landing/summary.md:72

**Issue:** The rubric makes a two-way call a finding when the diff pushes anything outward. This diff does:
- PR-Review now edits the pull request's title and body on GitHub at each poll whenever GitHub holds other text, which overwrites a hand edit.
- Finalize edits a recorded pull request before it merges it.

The diff also adds three landing keys. `summary.md:65` asks the operator to set `landing.describe_forbidden_terms: []` in the hub file. Once that is set, a binary without the key refuses the strict landing load, per PATTERN-config-layers, "Keys the template lacks". A revert therefore leaves the hub broken until the key is pruned.

The blast radius `landing` is too narrow. The `MultiLLMProducer` timeout mapping changes every seat row, among them Discussion-Write, Plan-Write, Parts, Raddle, the darn writer, Drift-Expect and the Drift-Verify judge.

### [MEDIUM] describeEntry's new gate order has no test

**Location:** internal/shedrecipe/entries_describe_test.go#TestDescribeEntry_HappyPath

**Issue:** `table.Gate = append(slices.Clone(gate), table.Gate...)` is what keeps the reviewer's entry in the writer's gate, behind the description gate. The happy-path test's table carries no gate of its own and asserts only one `description` entry. Dropping the table's entry, or reversing the order, keeps every test green.

### [MEDIUM] The configured forbidden terms are never shown to reach a check

**Location:** internal/shedrecipe/entries_gate.go#resolvePlainGateClosure

**Issue:** Every caller test passes `nil` patterns, and `envkit.FullEnv` returns `nil, nil`. The tests that do so are `dobbydeps_test.go`, `beamdescribe_test.go`, `parity_test.go` and the shedrecipe gate tests. So nothing fails when the forwarding from `landing.describe_forbidden_terms` is dropped. That applies to the Describe row's gate, `validate-description`, the beam describer's gate and findings, and the beam recorders.

### [MEDIUM] Code-Review does not check documents or the PR body for the tooling's names

**Location:** contracts/stencils/loom/loom-rubric-code-review.md

**Issue:** #255's second What-to-do item asks Code-Review to check documents and the pull request body for such names, so the finding comes before approval. The diff leaves the rubric unchanged. The entry's opening line defers only items 3 to 6 of #225's Idea. The change description names the omission (`summary.md:61`), but nothing licenses it.

### [MEDIUM] #241's held-turn reminder is not built

**Location:** internal/shuttleengine/hold.go

**Issue:** The bundled #241 note's suggested fix has two parts:
- remind the agent, or hand the pane back with a visible notice, when a turn end of an operator-started turn is held for missing outputs;
- classify the resulting timeout as resumable.

The diff makes only the second part. The silent hold that cost about 28 minutes in the trace is unchanged (`summary.md:59-60`).

### [MEDIUM] Only two writer stencils carry the vocabulary rule

**Location:** contracts/stencils/darn/darn-template-write.md:9

**Issue:** #255's first item covers the Describe step "and any step that writes repository documents". The diff adds the rule to the Describe stencil, and to darn's paragraph about the change description only. No stencil of a step that writes repository files carries it: neither the darn writer's code and docs instructions, nor the implementer, nor the fixers.

### [LOW] The findings hand-off repeats after a re-attach

**Location:** internal/landingshed/describereview.go#describeReview

**Issue:** `describeEntry`'s table source builds a fresh `describeReviewGateEntry` at every Call. So after a probe re-attaches a live writer, `sent` is false again. A `done` marker and a findings report from before the interruption then re-send findings the writer has already handled.

`shuttleengine.GateEntry.PassOnCap`'s doc asks that a request which must not repeat make its own closure idempotent. `summary.md:55-56` records this as a known limit, but the code does not follow that contract.

### [LOW] A PR-Review round can still end while the texts differ

**Location:** internal/landingshed/prreview.go#wake

**Issue:** `syncText` runs only when `!rec.Continue`, so `pr-continue` ends the review without a final compare. A transient edit failure is logged and the wake goes on to `judge`, which can end Done on an approval. Finalize's `setRecordedText` sets the text right before a non-beam merge, so what lands is correct. But #228's "refuse to end a round while the two differ" is not met for these paths.

### [LOW] A Merge danger sentence moved under Forbidden terms

**Location:** contracts/specs/loom-final-summary-spec.md:56

**Issue:** The sentence "`Publish` and `Finalize` do not check it: they carry the body as it stands." closed the Merge danger paragraph. It now follows the new section's last line, so its "it" reads as the forbidden-term check.

### [LOW] An empty describe_review is never loaded in a test

**Location:** internal/landingshed/config_test.go#TestLoadConfig_InvalidKeys

**Issue:** `LoadConfig` skips the model-spec parse for an empty `describe_review`. That empty value is the stated way to run no reviewer, and no LoadConfig row exercises it.

### [LOW] setRecordedText's failure paths have no rows

**Location:** internal/landingshed/finalize_pr_test.go#TestFinalize_PullRequest

**Issue:** Only a failed edit that halts is pinned. The status read that `setRecordedText` adds before every non-beam merge can fail, and so can a transient edit; neither has a row.

### [LOW] The review gates' error paths have no rows

**Location:** internal/landingshed/describereview_test.go#TestDescribeReviewGateEntry

**Issue:** No row covers an unreadable end marker, a report read error other than not-exist, or `describeReportGate` over a missing report.

### [NIT] The attempts comment claims a re-attach margin

**Location:** internal/landingshed/describereview.go#describeReviewGateAttempts

**Issue:** `GateEntry.PassOnCap`'s doc says the count lives in memory for one shuttle run, so an attach starts from zero. The second attempt is no margin for a re-attached run.

### [NIT] The timeout Stuck drops the chair gate's attempts and spans

**Location:** internal/shedadapters/multillm.go#mapOutcome

**Issue:** The new timeout branch returns `OutputPointer{Reason: reason}`. A chair held by a pending Describe-Review entry until its deadline then reports no gate attempts or held span.

### [NIT] A pattern that can match empty silently passes lines

**Location:** internal/landingshed/config.go#LoadConfig

**Issue:** `forbiddenTermFindings` treats `FindString(line) == ""` as no match. Take a pattern such as `lym|`: its leftmost match is always empty, so it never refuses a line.

### [NIT] wireLightweight's comment says it loads no config

**Location:** internal/loomcli/wiring.go:367

**Issue:** The new `DescriptionTerms` closure strict-loads `landing.yaml` on the lightweight path when validate-description runs. Only the later line about validate-description was updated.

## Rejected

- No fork finding was rejected. The code-scope lens's MEDIUM on #228's round ending is kept at LOW: Finalize now sets the recorded pull request's text before a non-beam merge, so the landed text follows the description; only the round's own end is unsynced.
- Handler candidate, not filed: TestDescribeTable_ReviewerSeat runs without t.Parallel. Its sibling TestDescribeSpec_ComposesSpec, which uses the same newDescribeInputs, is serial too, and I found no reason either way, so a PATTERN-test-speed violation is not established.
- Handler candidate, not filed: the default patterns leave out bare task, gate, tier, parent and board. The entry's own "read first which words the check can match without refusing a repository's own legitimate names" licenses narrowing them, and the Describe stencil states the full rule.
