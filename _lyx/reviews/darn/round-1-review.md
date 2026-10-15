---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/reedengine/render/types_test.go:28
    summary: the new keepViewer JSON key has no pin for omission when false or for a round trip
    fix: In TestDisplayMarshalsWithoutRetiredKeys, add an assertion that json.Marshal of a Display with KeepViewer false carries no keepViewer key. Extend the round-trip case that follows it so the want Display also sets KeepViewer true, which pins that the field persists and reads back.
---

### [LOW] keepViewer persistence is unpinned

**Location:** `internal/reedengine/render/types_test.go:28`
**Issue:** `Display.KeepViewer` is persisted in `reed.json` as `keepViewer,omitempty`, and `types_test.go` is the file that pins Display's on-disk key contract.
The diff adds no case there, so nothing fails if the tag drops `omitempty` or the field stops round-tripping, and a lost value would silently turn `--focus=false` back into follow-the-run after a reed reload.
No existing test covers this: `TestDisplayMarshalsWithoutRetiredKeys` and `TestDisplayDecodesRetiredKeysIgnored` never set the field.
The fix is an assertion in the existing test, not a new test, per `PATTERN-test-economy`.

## Checked and found sound

- Scope: the diff makes both changes the board entry asks for, the `--focus=false` fix (#628) and the launch-line fix (#627).
  It stays out of the two items the entry marks "only if the fix needs it", and the decision record's scope matches.
- Zero value: `newAgentWindowSelected` returns early on `Focus`, then on `KeepViewer`, so `Focus` wins in every branch.
  A zero `Display` still reaches the follow rule.
  The loom driver, arbiter and seat spawns build `Display` without `KeepViewer`, and `shuttleengine/open.go` does too.
- Persistence: `shuttleengine/run.go` copies `spec.Display` whole, and reed stores it whole (`strand.go`), so the field is carried wherever `Display` is copied.
  `lyx reed add` leaves it unset, as the summary says.
- `@lyx_unseen`: `openOwnWindowLocked` marks any window whose `selectNew` is false, so a keep-viewer window gets the mark, and the existing tmux-tier tests cover that path.
- Launch key: `launchStrandLocked` sends the key without `-l`, so it goes as a key name.
  An empty key (pwsh) skips the call and types only the payload and `Enter`.
  Every launch and relaunch path goes through `launchStrandLocked`.
  The key is `C-u`, not the interrupt key, so `PATTERN-guarded-interrupt` is untouched.
- Docs: `internal/reedengine/doc.go` names the new flag and the discard key in the same diff, and no `PATTERN.md` entry is called for.
  The `--focus` flag help is updated.
- Change description: each claim, including the renamed tests, matches the diff.
- Test economy: the added cases sit in existing tests (`TestNewAgentWindowSelected`, `TestLaunchStrandLocked_LaunchScript`, `TestRunCmd_EffortAndFocusFlags`, `TestShell_ScriptExtAndDiscardLineKey`), so none duplicates another.
