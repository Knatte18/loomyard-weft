```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

My task was to rerun the sets behind the review's findings and check that the fix covers every member.

- **F1 (argument scans after a script-reached command word):** `grep` for `commandPositionPattern`, `shellArgumentPattern`, `goWordPattern`, `goArgumentsPattern`, `goSubcommandPattern` and `ghMutationScanPattern` in `rows.go` shows the old unbounded argument helpers are gone. The only rows still built on the unscoped `commandPositionPattern` are python (`rows.go:276`), rm (`rows.go:286`) and refuse-raw (`rows.go:556`). None of the three scans argument words, so none can run past a script's closing quote. Every row that does scan arguments now goes through `commandRowPattern`. That covers sed, perl and tee, the sleep flags, headless claude, adopt-take-hold, the go test/build/vet and `-tags` alternatives, and every publish alternative (git flags, the gh repo flags, `notWordFrom`, `ghArgumentsPattern`, `mutationScan`).
- **F3 (docs that still list `bash -c` as a bypass):** The `grep` over `.go` and `.md` files (excluding `docs/research`, `_lym` and `.lym`) leaves no stale mention. The remaining `doc.go` hits at 681, 687 and 853 are the intended read-the-script text, "nested `bash -c`", and the unrelated `RunPluginHook`. Two other hits are about other guards: `bash -c` is named as a bypass in `websterengine/audit.go:334`, `websterengine/doc.go:380`, `fabricengine/refscanner.go:264` and `planparser/verifymodule.go:76`. They don't describe the shell-guard rows, so they are out of scope.
- **F5 (heredoc lines and double-quoted expansions inside a script):** The heredoc-operator branch is in both script reach patterns, so every row built on `commandPositionPattern` or `scope.position` reads it, and in `redirectWritePattern`'s script branches. The single-quoted-script double-quoted-expansion alternative was added to raw-go. The raw-go env-name alternatives still stop at `<<`, but the top-level ones do the same, so script and top level agree there.

Not checked: the `codexengine` goldens. The tier-0 gate across `claudeengine`, `codexengine`, `configcli` and `orchcli` passed in the parent's run.

Calls:
1. `grep` for `bash -c` / `sh -c` in `claudeengine/doc.go`, `codexengine`, `docs`, to find leftover bypass mentions; my context held only the lines that were edited.
2. `grep` for the old helper names in `rows.go`, to list which rows are still unscoped; my context showed the pre-edit file.
3. `grep` across `.go` and `.md` files for other copies of the "behind `bash -c`" and "after do or then" wording, to find stale twins outside `doc.go`.
