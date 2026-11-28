```yaml
lens: code-affected-set
outcome: reported
```

Affected-set lens over the config-key sweep diff, range `7cb0048..HEAD`.

**Per change, the set found and whether it is covered:**

- **`lym config` help** (`internal/configcli/configcli.go:516`): the literal `deny_python=false` is now rendered as `configengine.KeyName("shuttle", string(shellguard.SwitchDenyPython))`.
  - Readers I searched: any test, doc or fixture quoting `deny_python=false` or "lets Python through" under `internal`, `cmd` and `docs`. The only hit is the changed line itself.
  - Covered; nothing is left stale.
- **`PATTERN.md` `PATTERN-config-key-names` gains ` (test)`.**
  - The tag format comes from `internal/pattern/doc.go:45-48` and `internal/pattern/check.go:42` (`entryTestMarker`), which trims it before the entry text is compared. The line keeps the entry shape the parser accepts: rule text, ` (test)`, no background link. `PATTERN-github-auth` has the same shape.
  - No registry of guard tests exists: `TestLeafInvariant_AllowlistOnly` and the new test name are referenced only in their own files.
  - The rubric's "Declared PATTERN changes landed" applies to nothing here, because this round's directive declares no PATTERN changes.
  - Covered mechanically. Whether `(test)` is earned when the guard reads only string literals is the handler's and scope lens's focus item; I raise no affected-set finding on it.
- **New guard** (`internal/configreg/keynames_enforcement_test.go`): checked against `pattern/PATTERN-test-speed.md:122-151`.
  - It reads repository files outside its package (`cmd`, `internal`), so it needs `//lym:guard`. It carries it, directly above `func Test…`.
  - It is a tier0 file, which the rule allows.
  - The "`//lym:only parent-align finalize`" rule binds only a whole-repository guard of `cmd/lym`. This test lives in `configreg`, so it does not bind.
  - The allowlist entries' existence is enforced by `RequireNoStale`.
  - Covered.
- **Comment and package-doc renames across the 59 package pieces.**
  - No code, test, stencil or fixture reads Go comments. The only reader is the comment lint, which the verify gate owns.
  - The sweep added no backticked identifier that another tool parses; doc-comment openings and symbol names are unchanged.
  - Covered.
- **Commit messages** (`git log` over the range): no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer appears.
  - The answer key is not a bug fix, so no root-cause trailer is owed.
  - No test assertion was loosened: the only changed test file is new.
  - Commit `537848eb1` lists the guard's skips without the `.` and `%` prefixes. That is a commit-message description, not a claim of agreement or improvement, so it is no finding.

**Findings:** none from this lens.

**Tool calls:**
1. Grep for `(test)` handling, `//lym:guard` users, help-text asserts and mentions of `config-key-names`. My context lacked how the tag is enforced and who reads the help literal.
2. Grep in `internal/pattern/check.go`, `internal/pattern/doc.go` and `PATTERN.md` for the `(test)` marker. My context lacked the rule for test-tagged entries.
3. Read of `internal/fabriclayout/leaf_enforcement_test.go` plus a grep of `PATTERN-test-speed.md` for the guard rule. My context lacked the conventions for guard markers.
4. Read of `pattern/PATTERN-test-speed.md:120-160`. My context lacked the `//lym:guard` and `//lym:only` rules for repository-scanning tests.
5. Grep for the marker rules in `internal/pattern/check.go`, for existing key-name tests, and `git diff --stat` over the range's md, docs and stencil paths. My context lacked whether any doc or stencil changed beside the code.
6. Grep for any registry listing guard tests by name. My context lacked whether a new guard must be registered anywhere.
