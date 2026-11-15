---
round: 3
focus:
  - "Check every entry of testOnlyCodeAllowed in cmd/lym/testonlycode_linux_test.go against PATTERN-test-only-code: does each allowlisted function read its own package's unexported state or write a record only its package writes, with no _test.go file or testkit package able to hold it?"
  - "Check that no function only tests call remains in a production file outside that allowlist, and that any function moved into internal/testkit/shedcheckkit or into a _test.go file has all its callers and header comments updated with no reference to the old location left."
  - "Check that internal/fabricengine/doc.go describes only what production code does: does any sentence present SnapshotWarpSHAForTest, which lives in export_test.go, as a production reader with consumers?"
---

Targets the allowlist exception and the package doc, the two areas round 2 found open.
