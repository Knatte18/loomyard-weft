```yaml
lens: fix-proposal
outcome: reported
```

No findings.

- **F1 (skip the probe under `SkipGit`):** I found no case where it goes wrong. `CommitAnchoredPaths` returns early under `SkipGit` and ignores the path list, so leaving `withFriction` false changes nothing observable. Step 2b still clears and recreates the directory, which is the same as before.
- **F2 (docs):** The new text says every start or step commits whatever the friction directory holds. That matches the probe, which runs on every invocation, and nothing contradicts it.
- **F3 (tests):**
  - The four probe rows match git's `ls-files --cached --others --exclude-standard` behavior. A tracked file deleted on disk and an untracked file give true. An ignored-only directory and an empty directory give false.
  - The tier 1 run passed with 0 failures, so git commit identity works in this package's fixtures.
  - The smoke assertion `len(tracked) != 1` fits one resume note. It passed in the tier 2 run (`re-entry_keeps_friction_notes` PASS, log lines 534–535), so the subtest did run and was not skipped.
- **F4 (test wording and row name):** Pure rename, no effect elsewhere.

Tool calls:
1. Grepped the tier 2 gate log for `TestSmokeStepBootstrapWiring`. My context showed the gate summary (35 passed, 248 skipped) but not whether the smoke subtests were among the passes.
