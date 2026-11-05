```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the `FollowTemplate` change that parses the base only when `haveBase` is true.

No findings.

- **Scoping:** the `err` declared inside the `if haveBase` block is local to it. The later `_, templateRoot, err :=` and `doc, existingRoot, err :=` still declare `err` fresh, because `templateRoot` and `doc` are new. It compiles, and the gate was green.
- **`haveBase` false, corrupt base:** `baseRoot` stays nil and no error is returned. The old `if !haveBase { baseRoot = nil }` is gone, and the result is the same for every other input.
- **`haveBase` true, base `- not\n- a mapping\n`:** the base is parsed, and the error wraps `ErrInvalidBase` with the same text as before.
- **Caller:** `configsync.go:1046` passes `templateBasePresent && !fileAbsent`, so its `errors.Is(..., ErrInvalidBase)` branch is still reached whenever the base matters.
- **Existing corrupt-base tests:**
  - The configsync row "a template base that is not a YAML mapping…" has a present hub file (`hubFile: template`), so `haveBase` is true and it still fails as expected.
  - The hubreconcile row "a template base that is not a YAML mapping names the base and no config command" ran against a hub that is not shown to have its file absent, so I assume `haveBase` is true. I did not check that.
- **Added row:** "a base that is not a mapping is not read while the file has no base" fails with the defect in place. The old code would return `ErrInvalidBase`, and the table loop `t.Fatalf`s on any error. It also asserts no report entries and an unchanged file.

Tool calls:
1. `grep` of the configsync, hubreconcile and configcli tests for corrupt-base rows. My context did not show which tests reach a corrupt base.
2. `Read` of `ensure_integration_test.go` lines 975–1015. I needed the hubreconcile row's fixture.
3. `Read` of `configsync_test.go` lines 925–1025. I needed the configsync rows' `hubFile` and `baseFile` values.
