```yaml
lens: fix-code-correctness
outcome: reported
```

Confirmed: that existing row sets `WritePaths` to `fx.Deps.OutcomePath` and expects the batch to fail. So the `CorrectnessFindingToo` row does exercise the correctness branch, and there is no vacuous pass.

The only finding stands: a **LOW** one, that no test ties the stencil's hardcoded rung text to `SecretFileFailureReason`.
