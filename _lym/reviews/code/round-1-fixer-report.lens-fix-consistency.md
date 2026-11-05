```yaml
lens: fix-consistency
outcome: reported
```

No findings.

- **What I checked:** the changed text of every fix against its twins.
  - The start guard's doc comments, the `Unreleased` field comment, `releaseCommandText` and the restored test row all name draft and discussed together.
  - The `boardengine/doc.go` run-lock paragraph, the `checkRunLocks` doc comment and `endsAsSketch` all state the same net-change rule.
  - The `sketchFinish` comment and the `loomcli/doc.go` paragraph describe the same refusal-only fallback.
  - All three `checkRunLocks` callers in `board.go` and `replay.go`, plus the one test caller, pass the vocabulary.
  - No remaining text still uses the old lowercase `legacyStatusDiscussed`.
- **Out of scope, not a finding:** `boardengine/doc.go:317` and the draft-layer text at `doc.go:204` and `layer.go:27` don't mention `ErrSketchRefused` or a kept `discussed` task. The fix didn't make either worse.

Tool calls:
1. One `grep -rn` across the repo's `.go`, `.md` and `.yaml` files for the guard wording, `MakeSketch` and "passes whole". It checked for stale twins of the changed text, which my context didn't hold for files outside the edited set.
