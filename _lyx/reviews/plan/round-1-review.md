---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/04-loom-gate-and-verb.md:15-16
    summary: Card 4 sets RelevantPath in loomCLI.wire and loomCLI.wireLightweight but plans no assertion of it, so an unset path on the lightweight wiring that validate-discussion runs on would silently skip the relevant-list check with no test failing
    fix: In card 4, add `internal/loomcli#TestWire_DefaultConfig` and `internal/loomcli#TestWireLightweight_FillsThePathsWithoutLoadingAnyConfig` to the Edit list after `internal/loomcli#loomCLI.wireLightweight`, and add a Tests bullet saying each gains one assertion, beside its existing SupportLogPath assertion, that `c.env.RelevantPath` equals `loomengine.DiscussionRelevant(location)` (`loc` in TestWire_DefaultConfig). This overlay passes `lyx loom validate-plan --overlay` with no new finding.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/04-loom-gate-and-verb.md:63
    summary: IOFault_RelevantIsDirectory asserts the message contains relevant.md, which the old decision-record prefix also produces because the os read error names the path, so the row cannot fail on the prefix change it is meant to pin
    fix: 'In card 4''s TestValidateDiscussionCmd bullet, change IOFault_RelevantIsDirectory''s assertion to want the message to contain `loom: validate discussion: ` (the new prefix, which the old `loom: validate discussion record <path>: ` prefix does not contain), and say the relevant path itself comes from the wrapped read error.'
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/plan/04-loom-gate-and-verb.md:61
    summary: Card 4 says TestValidateDiscussionCmd gains two rows and then lists three
    fix: In card 4's TestValidateDiscussionCmd bullet, change "gains two rows" to "gains three rows".
---

### [MEDIUM] The new Env field's assignment on both loomcli wire paths is untested

**Location:** `_lyx/plan/04-loom-gate-and-verb.md:15-16`

**Issue:** Card 4 has `loomCLI.wire` and `loomCLI.wireLightweight` set `RelevantPath: loomengine.DiscussionRelevant(location)` but names no test that reads it.
`TestWire_DefaultConfig` (`internal/loomcli/wiring_test.go:208`) and `TestWireLightweight_FillsThePathsWithoutLoadingAnyConfig` (`internal/loomcli/wiring_test.go:729`) assert every sibling path field, `SupportLogPath` included.
`validate-discussion` runs on `wireLightweight` (`internal/loomcli/cli.go:300`, `internal/loomcli/arm.go:190`).
If that assignment were dropped, `CheckRelevant("")` would meet a not-exist read and return no finding, so the verb would silently stop checking the list: the gate and verb would diverge, breaking `PATTERN-gate-self-check-parity`, while every planned row still passes, since the parity and verb rows build their `Env` through `decisionRecordFixture`, not the wire paths.
On `wire`, a dropped assignment only surfaces at run time, as the `requireAbsRoot` wiring refusal.

### [LOW] The I/O envelope row does not pin the prefix change

**Location:** `_lyx/plan/04-loom-gate-and-verb.md:63`

**Issue:** Card 4 changes the verb's I/O error prefix from `loom: validate discussion record <decision record path>: ` to `loom: validate discussion: ` so a failing relevant-list read is not attributed to the decision record.
`Validate` returns `os.ReadFile`'s error unwrapped (`internal/discussionparser/validate.go:71-79`), and card 3 has `CheckRelevant` do the same, so the message names `relevant.md` through the `*PathError` under either prefix.
The planned assertion, that the message contains `relevant.md`, therefore passes against the unchanged code and does not reach the change the lens review says it ties down.

### [NIT] Row count disagrees with the rows listed

**Location:** `_lyx/plan/04-loom-gate-and-verb.md:61`

**Issue:** "`TestValidateDiscussionCmd` gains two rows:" is followed by `Findings_RelevantMalformed`, `Darn_MalformedRelevantIgnored` and `IOFault_RelevantIsDirectory`.

## Rejected

- Test economy on card 4's three stray-line rows: not a defect.
  The new `TestNewDiscussionGate` subtest is the only test that pins the combined-findings order, with `Validate`'s findings first.
  `Stuck_RelevantMalformed` pins gate-verb parity, comparing verdicts only (`cliVerdict`, `internal/loomcli/parity_test.go:68`).
  `Findings_RelevantMalformed` pins the verb's exit code and findings text, the split `TestValidateDiscussionCmd`'s `//testtiming:keep` reason records ("the parity test compares verdicts alone") and the existing `Findings_SupportLogAbsent`/`Stuck_SupportLogMissing` pair follows.
  `TestCheckRelevant`'s table covers the check's cases; none of the three re-tests them, since each uses one stray line as a probe.
- Test economy on cards 5 and 6: not a defect.
  Card 6 asserts the lead's `Outputs` and `Values["relevant_path"]`, both values.
  Card 5's `finishes writing ` anchor locates the rendered `RelaunchFiles` value, the same way the existing twill assertions anchor values with "per `Step 5`" and "`lyx loom validate-discussion` per `Step 6`" (`internal/loomengine/discussiontable_test.go:433-434`).
  The anchor is needed because card 6 renders the same paths elsewhere in the prompt.
- Fidelity: D1-D6 and every Constraint are carried.
  `ReviewedFiles` stays unchanged (card 5).
  `resolveModelChoice` and the seat literal's shape are left alone (card 6).
  `CheckRelevant` sits outside `Validate` (card 3).
  The `plan.go` header (card 7) and `discussionparser/doc.go` (card 3) are edited.
  The new I/O prefix, `docs/overview.md` and the PATTERN parity pair are edits that this design forces: the old prefix misattributes a relevant-list read error, CLAUDE.md requires docs to land with a CLI behavior change, and the PATTERN pair entry would otherwise name the wrong function.
  `ValidateLoomDiscussion` is the "one package function" D3 and the parity pattern call for, mirroring `ValidateDarnDiscussion`.
- Card order: cards 4-7 use only `DiscussionRelevant` (card 2) and `CheckRelevant` (card 3).
  Card 5 edits `TestDiscussionTable` before card 6, and card 5's anchor is written to stay unique once card 6 adds the path to Step 5.
- Verify coverage: `go test ./...` and `go test -tags integration ./...` cover every targeted package.
  The smoke tests card 4 edits are `tmux`-tagged (`internal/loomcli/smoke_gate_test.go:1`, `smoke_parentreview_test.go:1`), which `go vet -tags tmux ./...` compiles.
  The order matches the template's (`loom-template-plan.md:193`).
- Re-sign arrows: only `NewDiscussionGate` changes its signature, and it carries the arrow.
  `loomCLI.wire`, `loomCLI.validateDiscussionCmd` and `Directive` keep theirs, and `Env` and `Values` are struct types gaining a field.
  The file-path Edit targets are doc files whose Intent names no member to target instead.
- Attack surface and reachability: card 3's path-rule rows each name an input (`/`, `\`, `~`, `C:/x`, `a/../b`, `a\..\b`, and `..foo` passing) and reach `CheckRelevant` directly.
  Card 4's `requireAbsRoot` on `RelevantPath` is a wiring guard in `refusal-spec.md`'s sense (`contracts/specs/refusal-spec.md:31`), which takes no per-row test.
- Granularity of card 4: one unit.
  The `NewDiscussionGate` re-sign forces the resolver, `Env`, test-env builders and its five test callers onto one card, and parity needs the gate and the verb to change together.
