No findings, other than one stale doc reference noted below.

- **LOW — `claudeengine/doc.go:598`:** the package doc still says the settings and notice render `standingDeniesFor(cfg.RefuseRaw)`. The function now takes `(refuseRaw, lymPath)`, so the cited call no longer exists in that form.
- **Not a finding, `webstercli/smoke_test.go:44`:** it calls a real `engine.Prepare`, which would hit `ErrTestBinary` under the new `LymBinary` resolution. That is a smoke-tier test and the gate did not run it. It is a coverage gap rather than a defect in the change.

What I ran, step by step on one input each, all correct:
- **Claude check:** `pythonAllowedCheck("/stub/lym")` gives `(cd "${CLAUDE_PROJECT_DIR:-.}" && '/stub/lym' shuttle hook-python-allowed) >/dev/null 2>&1`.
- **Claude guard:** `renderShellGuard` puts it in `grep … && ! (…) … && echo deny; true`. A missing project directory fails the `cd`, so the deny stands.
- **Codex guard:** `guardCommand` renders `! '/stub/lym' shuttle … >/dev/null 2>&1 && echo …`, and the stub at the injected path is what the tests run.
- **Notice path:** `buildDenyNotice` and `init` render with `""`, so the check quotes to `''`. That is harmless because only notices are read there.
- **Standalone branch (F2, F4):** on a damaged hub the old `Board != ""` test returns the hub layers and the template default (deny), so row `PythonAllowedInADamagedHub` (want 0) would fail. Returning the hub layers on any `ResolveWithin` success would fail rows 1–3. The no-shuttle-file row prints no JSON, so it proves the line-94 fallback rather than the error branch.
- **Config load failure (F3):** dropping the `LoadConfig` error check makes the unparsable-config row exit 0 and fail.
- **Missing project directory (F8):** turning `&&` into `;` would run the stub and log, failing the row.
- **Gate:** tier 0 and tier 1 passed over the touched packages.
- **Minor:** Claude `Prepare` resolves `LymBinary` after `prompt.md` is written, so a refusal there leaves that file behind. Codex resolves it before any write.

Tool calls:
1. Grepped real `.Prepare(` callers in tests outside the engine packages, to see which test binaries still need the `LymBinary` stand-in.
2. Grepped docs for references to the changed signatures, because the context held no doc text for them.
