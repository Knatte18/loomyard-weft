```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Behaviour → pinning test:**

1. The template ships `remote_control: false`. Pinned by the `TestLoadConfig_RemoteControl` row "the template is off with the cross-machine address", which expects `false` from `model: opus\n` (`internal/orchengine/config_test.go:12` in the diff).
2. An explicit `remote_control: true` turns it on and keeps the template's name. Pinned by the row "switched on keeps the template's name".
3. An empty name loads while the switch is off by default. Pinned by the row "off with an empty name loads".
4. An empty name is refused while the switch is explicitly on. Pinned by the row "on with an empty name is refused". The row now sets `remote_control: true` explicitly, so the refusal check (`internal/orchengine/config.go:134`) is still reached. No assertion was widened or removed.
5. The control-character refusal does not depend on the switch. The unchanged row "a control character is refused" still reaches `config.go:131` under the new default.
6. `FlagRemoteControlDefault` flags a file with no `remote_control` and leaves its bytes unchanged.
   - Pinned by the `OtherKeysOnly` and `HeaderOnly` rows of `TestFlagRemoteControlDefault` (`reinterpret_test.go:108-109`).
   - They assert unchanged bytes, an empty `rewrote`, and exactly one flagged line containing the literal way forward.
   - `HeaderOnly` decodes to a nil map, the same path as an empty file or a null document, so those cases are covered.
7. A file that sets the key, true or false, is neither changed nor flagged. Pinned by the `SetOn` and `SetOff` rows.
8. A document that is not a mapping returns a parse error. Pinned by the `NotAMapping` row.
9. The orch module row registers `RemoteControlOffReinterpretationID` with `FlagRemoteControlDefault` (`internal/configreg/configreg.go:161`). **No test pins this.**
   - The only test touching `Reinterpretations` swaps in a fake slack reinterpretation (`internal/hubreconcile/reinterpret_integration_test.go:85`).
   - No test reads the registered orch entry or runs the real walk over an orch.yaml.
   - Dropping the `Reinterpretations:` field from the orch row fails no test, and hubs would silently never be flagged.
   - The gate module's registration (`configreg.go:150`) has no test either, so this follows the existing practice.

Expected values in both tests are literals (`wayForward`, the booleans and names), never computed by the code under test. No test is tautological.

**Findings:**

- **LOW · design** · `internal/configreg/configreg.go:161`. The orch row's registration of `orch-remote-control-off-by-default` → `FlagRemoteControlDefault` has no test that fails when it is removed.
  - Evidence: the grep shows no test referencing `RemoteControlOffReinterpretationID` or the real modules' `Reinterpretations`. The only such test, `internal/hubreconcile/reinterpret_integration_test.go:85`, replaces them with a fake.
  - The same gap exists for the gate row (`configreg.go:150`), which is why this is LOW.
  - Fix: add a row or assertion to an existing configreg registry test, checking that the orch module carries this ID.

Tool calls:
- Grepped the test files for `Reinterpretations`, `RemoteControlOffReinterpretationID` and `reinterpreted-` to see whether any test pins the registry wiring; the manifest held only non-test reads of that wiring.
