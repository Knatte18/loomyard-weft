```yaml
lens: fix-affected-set
outcome: reported
```

No findings that BLOCK or ship wrong behaviour. Two LOW findings, one MEDIUM.

- **MEDIUM — F3. The fix's new tier-0 test breaks the parallel rule it was meant to enforce.**
  - Changed text: `internal/webstercli/secretpaths_wiring_test.go`, `TestWire_HubLoadsSecretPathsFromFabricYAML` and its subtests.
  - Neither the test nor its subtests call `t.Parallel()`, and none carries a comment naming process-global state. They use no `t.Setenv`, `t.Chdir` or shared fixture; each subtest builds its own `t.TempDir()` hub.
  - This is the same defect F3 flagged in the original tests.
  - The tier-1 `TestRecordBatchCmd_SecretPaths` is exempt because it calls `t.Setenv("FABRIC_SKIP_GIT", ...)`. It should still say why it is not parallel.

- **LOW — F6/F1. The `record-batch` help text for the secret failure is stale.**
  - Changed text: `internal/webstercli/recordbatch.go:80-81`, the long help.
  - It says only "fails the same way; its error names the way forward".
  - That is still true, but it omits that this way forward belongs to the run's parent and that Master stops with outcome stuck. The `websterengine/doc.go` and Master stencil rewrites now state both.
  - A reader of the CLI help would assume Master can follow it.

- **LOW — F1/F2. `recordbatch.go:8` and `:533` were left unchanged.**
  - `recordbatch.go:8` says "It fails a batch whose commits add a secret file." and `:533` says "A secret file the batch committed sits in the branch's history..." Both remain correct.
  - Neither mentions the new ordering ahead of a correctness finding; only the `RecordBatch` doc at `:281` and the inline comment at `:452` do.

Sets that checked out clean:

- **F2:** no `AddedFilesBetween` reference is left in any `.go` or `.md` file outside `_lym`. All three layers use the new name: `gitrepo`, the `websterengine` wrapper and the fake `Git`.
- **F6:** no "on disk untracked", "delete the named files" or "deletes the file" wording remains in the repo. The remaining hits are unrelated.
- **F1:** `failOnSecretPaths` has two callers, and both pass the new arguments.
- **Stencil:** `webster-template-master.md` carries no `lym-stencil:` hash marker, so the new rung needs no regeneration.

Calls made:
1. Grep for stale `AddedFilesBetween` or net-tree wording, stale F6 and F1 phrases, and `failOnSecretPaths`, `SecretFileFailureReason` and stencil references. My context lacked a repo-wide search of the unchanged sites.
2. Grep for `t.Parallel` and test function names in the two new `webstercli` test files. My context had them only as written, not as a check.
3. Read `internal/webstercli/recordbatch.go` lines 72-90 and the head of the stencil. I had read neither, and needed the help text and the stencil's marker.
