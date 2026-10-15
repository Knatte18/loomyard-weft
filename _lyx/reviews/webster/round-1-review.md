---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/shedrecipe/entries_gate.go:189
    summary: the new RelevantPath requireAbsRoot guard on the loom branch of the discussion gate resolver has no test reaching it
    fix: add a row to internal/shedrecipe/entries_gate_test.go (a new subtest beside "discussion with a history seam" in TestResolveGateSpec_ValidOneEntryPerVocabularyName, or a new row in TestResolveGateSpec_Rejections built over newTestEnv) that blanks env.RelevantPath on a loom run, with env.DiscussionHistory nil, calls resolveGateSpec with a gates list naming "discussion", and asserts the error names "RelevantPath"; a second case with a relative RelevantPath is optional. Extend an existing table rather than adding a top-level test, so the shedrecipe test-budget row is untouched.
  - id: F2
    severity: LOW
    class: consistency
    location: contracts/specs/specs.go:30
    summary: the LoomDiscussionSpec doc comment still describes the spec as the decision-record and support-log contract and says every writer of "the two files" writes against it, but the spec now pins three files
    fix: reword the two comment lines above `var LoomDiscussionSpec` to name the decision record, the support log and the relevant list, and say every writer of the three files writes against the format, e.g. "LoomDiscussionSpec is the decision-record, support-log and relevant-list format contract's shipped-default content." and "Every writer of the three files writes against its format, and internal/discussionparser checks it."
---

### [MEDIUM] RelevantPath guard in the discussion gate resolver is untested

**Location:** internal/shedrecipe/entries_gate.go:189
**Issue:** Card 4 adds a fail-loud `requireAbsRoot(entry, "RelevantPath", env.RelevantPath)` to `resolvePlainGateClosure`'s loom branch, and the card lists no test for it.
`newTestEnv` and `FullEnv` now always set `RelevantPath`, so no existing row blanks it.
`TestResolveGateSpec_Rejections` and the other `entries_gate_test.go` tests reach no `RelevantPath` refusal, so deleting the guard leaves every test green.
The guard is a behavior the diff adds with no coverage, and a new refusal in a bound module needs a test that reaches it under `PATTERN-refusal-way-forward`.
The sibling `DecisionRecordPath` and `SupportLogPath` guards of the same case are equally unreached today, but that does not excuse the new one.
Because it is a wiring guard no on-disk state reaches, it needs no `refusal-spec.md` row.

### [LOW] LoomDiscussionSpec doc comment still says two files

**Location:** contracts/specs/specs.go:30
**Issue:** Card 1 retitles the discussion spec to cover the decision record, the support log and the relevant list, and the spec now pins three files.
The embed's doc comment in `contracts/specs/specs.go` still calls it the decision-record and support-log contract and speaks of "the two files".
The card did not list `specs.go`, so this is a missed call site rather than a deviation from the card.

## Verified without findings

- `discussionparser.CheckRelevant` (card 3) matches the format the discussion spec (card 1) pins: optional first-line H1 with text, top-level `- ` entries only, backticked path, rejection of absolute, `~`, drive-letter and `..` paths on both separators, ` — ` plus a non-blank reason, one finding per defective line and one no-entry finding.
  A missing file returns nil findings and a nil error, any other read error returns the error with nil findings, and a present malformed list yields findings.
  `TestCheckRelevant` carries every row the card lists, and the discussionparser test-budget row went up by one.
- `loomshed.ValidateLoomDiscussion` is the only loom-side caller of `discussionparser.Validate` besides `ValidateDarnDiscussion`.
  `NewDiscussionGate` and `lyx loom validate-discussion`'s non-darn branch both call it.
  `CheckRelevant` runs after a `Validate` finding, and both errors pass through as is.
  Production callers of `NewDiscussionGate` are the one resolver site, and the darn gate and verb branch never read the list.
- Stencil edits read against their cards' Intent:
  the Discussion template changes Step 5, "What you may write" and both re-prompt places and keeps the spec pointer;
  `twill-relaunch.md` names `{{.relaunch_files}}` while keeping "every reviewed file" in the read-back sentence;
  the Plan template makes the decision record the only source of design and scope and the relevant list a head start that adds no scope, keeping the STOP rule and the no-support-log, no-board rule.
  No rule outside the cards was dropped, weakened or contradicted.
- The final wiring-test fix edits the existing chair-outputs assertion in `TestWire_DiscussionTableEvaluatesToExpectedShape`; it adds no new test.
- The gate tests over `discussionparser`, `loomshed`, `loomcli`, `loomengine`, `shedrecipe`, `twilldirective`, `loomrecipe`, `contracts/...` and `cmd/lyx` pass.
- The new doc comments follow the repository's conventions; line breaks are left to `lyx loom lint-comments`.
