```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I looked for the test row that would fail if it broke.

**Per behaviour, the test that pins it**

1. **The guard reports a production string literal that names a template key bare** (`internal/configreg/keynames_enforcement_test.go:179-206`). Only the guard's run over the live tree pins this. `allowlist.RequireNoStale` (line 205) fails if detection stops matching the nine allowlisted underscored names in their files. That is an indirect canary for the underscored-name path only. No row feeds a known-bad literal and expects a report.
2. **Skip rules in `bareKeyNames`** (lines 115-137): the `.`/`%` prefix, the `--set`/`--unset` prefixes, the trailing colon after trimming `` ` ``/`"`/`\`, and the one-word name counting only as "the `<name>` key". None of these has a row.
3. **The whole-literal skip** (lines 194-196). None.
4. **`isKeyNameCall` skipping the arguments of `*KeyName`/`*keyName` calls** (lines 141-151, 165-166). None.
5. **`stringLiterals` leaving out struct tags (`ast.Field`) and import paths** (lines 155-175). None.
6. **`buildKeyNameIndex`** (lines 37-60): seed-only modules are skipped, open and per-name maps are indexed without their entries, and trailing parts get indexed. None directly. A broken index would mostly surface as stale allowlist entries or a flood of reports, never as a quiet miss.
7. **The scan floor**: `scankit.RequireFloor` (line 204), held by scankit's own tests. Pinned.
8. **The `lym config` help naming `shuttle.deny_python` through `configengine.KeyName`** (`internal/configcli/configcli.go:516`). This is static help prose, so it is exempt. The new guard also covers it as a regression check.
9. **Comment and package-doc rewrites across the other pieces.** Comment-only and no behaviour, so exempt.

**Findings**

- **MEDIUM, class `design`**, `internal/configreg/keynames_enforcement_test.go:115-175`
  - Summary: the guard's detector and its skip rules have no test of their own. None of the colon, `.`/`%`, `--set`/`--unset`, one-word "`x` key", whole-literal, `KeyName`-call or `ast.Field` branches has a row, so a change that makes any of them skip too much passes silently.
  - Why it matters: the only signal is `RequireNoStale`, which covers just the nine allowlisted underscored names in their own files. A regression in the one-word form or the KeyName-call skip would keep every allowlist entry live and stay green.
  - Repository convention: other guards here test their detectors on synthetic input. Those that do include `internal/cliwire/bannedecl_enforcement_test.go`, `internal/shuttleengine/seam_enforcement_test.go` and `internal/lymcwd/enforcement_test.go`.
  - Fix: add one table-driven test that feeds literal texts to `bareKeyNames` and `stringLiterals` on parsed source. Expected bare names come from literals, for example:
    - ``"set `scoper` key"`` reports `scoper`;
    - `"max_bounces"` inside a longer text reports it;
    - `"loom.code.max_bounces"`, `"--set max_bounces=1"`, `"max_bounces: 3"` and a `KeyName("loom", "code.max_bounces")` argument report nothing.

No other finding in this lens.

Tool calls:
1. Listed the `*enforcement_test.go` files and grepped `configcli` tests for help-text assertions, to see whether any test pins the help line or the detector. The context had neither.
2. Grepped `pattern/` for `lym:guard` and read `PATTERN-test-speed.md`'s guard section, to check the repository's rules for guard tests. That background file was not in the manifest.
3. Grepped enforcement tests and `scankit` for synthetic-input tables, to see whether guards here test their detectors. The context did not show any.
