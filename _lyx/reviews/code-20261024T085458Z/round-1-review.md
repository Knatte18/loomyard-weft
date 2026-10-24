---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/loomcli/decision.go:148
    summary: A --from file with an unknown or misspelt key is not refused; the key is silently dropped, though the decision record and acceptance criteria require the refusal
    fix: In decisionEntries, call decoder.DisallowUnknownFields() right after json.NewDecoder(bytes.NewReader(data)) so a decode error names the unknown key. The existing branch that reports a decode error already appends decisionWayFromFile, so the refusal names its way forward. Add one row to TestDecisionVerb_Refusals in internal/loomcli/decision_test.go, such as from `[{"title":"a","decision":"d","rationale":"r","pattern_enty":"PATTERN-x"}]` with inputRefusal and noDarnReads true, wanting "unknown field", "pattern_enty" and "way forward:".
    origin: lens:code-correctness
  - id: F2
    severity: LOW
    class: design
    location: internal/loomcli/decision.go:150
    summary: decoder.More() is false before a stray ] or }, so a --from file such as an array followed by ] is accepted instead of refused as data after the array
    fix: Replace the decoder.More() test with a second decoder.Decode into an empty struct{} (or json.RawMessage) that must return io.EOF; any other result is the "data follows the array" refusal, keeping decisionWayFromFile. Add a row with a trailing ] to the "data after the array" case in TestDecisionVerb_Refusals.
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: design
    location: internal/loomcli/decision.go:403
    summary: The new loop in beginAmendment that restores the amendment record when a later title fails has no test, since a one-entry call can never fail on a second title
    fix: Add one row to the existing amend-failure tests in internal/loomcli/decision_test.go where a --from batch of two entries with amendsPlan has the fake appendAmendment fail on the second title; assert the amendment record equals its prior bytes (or is absent if it did not exist), nothing is appended and nothing is committed.
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: consistency
    location: internal/discussionparser/doc.go:4
    summary: The package doc names only AppendDecision as the post-Discussion write path and omits the new AppendDecisions, against the rule that module docs land in the same commit
    fix: In internal/discussionparser/doc.go line 4, extend the sentence on AppendDecision to say that AppendDecisions adds several such calls to the record in one write and one discussion check.
    origin: lens:code-affected-set
  - id: F5
    severity: LOW
    class: consistency
    location: internal/loomcli/decision.go:868
    summary: The add help still says --title, --decision and --rationale are required, which --from contradicts until a later paragraph
    fix: In the add command's Long text, change "--title, --decision and --rationale are required." to say they are required unless --from is passed, and name --from in that same sentence.
    origin: lens:code-affected-set
  - id: F6
    severity: LOW
    class: consistency
    location: internal/discussionparser/append.go:61
    summary: The AppendDecisions doc comment is stated in terms of another symbol, AppendDecision, against the self-contained comment rule
    fix: Rewrite the AppendDecisions doc comment so it states the behaviour directly, namely that each entry is added at the end of the record's Decisions section with its PATTERN bullet when set, the record is written once and Validate runs once, and a finding or an error from Validate restores the prior bytes; drop both references to AppendDecision.
    origin: lens:code-pattern-fit
---

### [BLOCKING] Unknown key in the --from file is not refused

**Location:** internal/loomcli/decision.go:148
**Issue:** The decision record's Constraints and its fourth acceptance criterion require an unknown key in the batch file to be refused with a way forward and nothing appended.
`decisionEntries` decodes with a plain `json.NewDecoder` and never sets `DisallowUnknownFields`, and neither `decision.go` nor `decision_test.go` mentions an unknown key.
A misspelt key such as `pattern_enty` or `tittle` is dropped, so the entry is recorded without its PATTERN change or fails with a misleading empty-field message, and the call can still report success.
Neither clause of `PATTERN-refusal-way-forward` is met for this refusal: no message and no test.

### [LOW] Data after the array can pass unrefused

**Location:** internal/loomcli/decision.go:150
**Issue:** `Decoder.More()` returns false when the next byte is `]` or `}`, so `[...] ]` decodes as a valid batch.
The acceptance criterion names a malformed file as a refusal.

### [LOW] Partial-batch amendment restore is untested

**Location:** internal/loomcli/decision.go:403
**Issue:** `beginAmendment` now appends several titles and restores the record when a later one fails.
No test makes the second `appendAmendment` fail, so dropping the restore would pass the suite.

### [LOW] discussionparser package doc omits AppendDecisions

**Location:** internal/discussionparser/doc.go:4
**Issue:** The doc lists `AppendDecision` as the write path and does not mention the batch append added in this diff.

### [LOW] add help says the per-entry flags are required without exception

**Location:** internal/loomcli/decision.go:868
**Issue:** The sentence "--title, --decision and --rationale are required" contradicts the later paragraph saying `--from` replaces them.

### [LOW] AppendDecisions doc comment leans on AppendDecision

**Location:** internal/discussionparser/append.go:61
**Issue:** The comment defines the batch behaviour by reference to `AppendDecision` instead of stating it.

## Rejected

- Whitespace-only `pattern_entry` builds a `PatternChange` with an empty ID (lens:code-correctness, NIT): the one-entry path has the same untrimmed check, so the diff introduces nothing, and the discussion check catches the result.
- One-entry test may lack a `headings` assertion (lens:code-tests, LOW): the lens could not see whether `TestDecisionVerb_AppendsAndCommits` omits it, and `headings` is built from the same slice the batch test pins; speculative.
- Arbiter and bouncer stencils show only the one-entry form (lens:code-affected-set, noted not filed): the diff did not change them and the record asks only for the orch stencil line.
