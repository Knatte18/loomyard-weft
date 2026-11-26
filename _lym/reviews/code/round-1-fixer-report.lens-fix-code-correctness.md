```yaml
lens: fix-code-correctness
outcome: reported
```

fix-code-correctness: checked the changed code in `anchor.go`, `driftcli/cli.go`, `gate_test.go`, `anchor_integration_test.go` and `store_test.go` for correctness, callers and tests.

no findings

- **`checkCwdAnchorGate` and `AnchorGateError`:** a cwd of `home/user/repo/sub` with anchor `.` yields `Refusal`, then the hint with the quoted anchor path. `errors.Is(err, ErrCwdOutsideAnchor)` still holds through `Unwrap`, so `preflight.ResolveMode` and the `driftcli` branch behave as before. An exact match still returns an untyped nil.
- **`runCaptureChild`:** a gate refusal is swapped for `Refusal` and keeps the lasting-fault mark. `ErrNotAGitRepo` and other errors fail `errors.As` and pass through unchanged.
- **Tests:** the integration assertion would fail if the child reported the full text, and the gate log in the context shows it ran and passed. The subpath `wantMessage` row would fail if the way forward named `worktreePath`.

No further tool calls made.
