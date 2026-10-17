---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/gatecli/cli.go:30
    summary: No test pins that Command passes shuttleengine.DefaultAwaitCap as the slot wait
    fix: Add one row to an existing gatecli test (package gatecli, or via export_test.go) that asserts the cap Command() hands newCommand equals shuttleengine.DefaultAwaitCap, for example by comparing the gate test verb's help text, which prints slotWait, against shuttleengine.DefaultAwaitCap.String(). Add no new top-level test.
    origin: lens:code-tests
---

### [LOW] The shared-cap wiring in Command is unpinned

**Location:** internal/gatecli/cli.go:30
**Issue:** The board entry requires the slot wait to take the cap from the one shared definition.
`Command` does this with `newCommand(defaultGoBinary, defaultSamplePeriod, shuttleengine.DefaultAwaitCap)`.
The integration helper builds `newCommand(goBinary, time.Second, time.Second)` directly, so no test reaches `Command`'s argument.
Replacing the constant in `Command` with a literal, or with another value, breaks nothing.

## Rejected

- Message-prose pin in the busy-hub row (code-tests, LOW): the `strings.Contains` check on the re-run message is the old row's way-forward check carried over, on a runtime message that `PATTERN-refusal-way-forward` requires to name the way forward. It is not a stencil's static prose.
- `cmd/testtiming` `takeGateSlot` has no test for the new bound (code-tests, LOW): it is a one-line swap of the wait bound inside a slow-boundary wrapper that has no seam for the acquire. Pinning it would need a new seam, which the rubric's test economy does not ask for.
- Cancelled-acquire route prints no envelope (focus directive): `acquireSlot` returns 1 with no envelope when the parent context is cancelled, as before. A cancelled acquire is not the cap passing, so the directive's "every route" applies to the cap-passed path only. The deadline route, the default error route and the success route each behave correctly.

## Verdicts on the focus directive

- Cap and envelope: `Command` passes `shuttleengine.DefaultAwaitCap`, the single definition. The cap-passed path prints `ok: true`, `slot_free: false`, the holders and a message, and exits `SlotBusyExit` (75). The cap passing is checked after the parent-cancel case, so a cancelled acquire is not misread as the cap.
- Retired key: `cli_wait_sec` is in `RetiredKeys` and out of `Config`, the template, `Limits` and `decodeConfig`'s checks. A `TestLoadConfig` row loads a file that still sets it. A repo-wide search finds it only in the intended spots (`config.go`, `config_test.go`, `gateslot/doc.go`), and finds no `CLIWait` or `CLIWaitSec`.
- Shell guard and golden: the added sentence matches the await help's words, "in a turn that does nothing else, so the prompt cache stays warm between calls". The golden diff is the same 128 insertions of that sentence and nothing else.
- Stencils and docs: the implementer and verify-fix stencils change only the busy-answer description and add the re-run wording. No other rule is dropped or weakened. `doc.go`, `docs/overview.md` and the cap's comments match the code.
- Change description: each claim checks out against the diff, including the `cmd/testtiming` wait and the fixtures that no longer write `cli_wait_sec`.
- Test economy: the cap-passed envelope, the retired-key load and the shell-guard sentence are each covered once, by an edited row or assertion. The diff adds no top-level test.
