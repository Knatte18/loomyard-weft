```yaml
lens: fix-affected-set
outcome: reported
```

Tasks checked: F1 is the only finding. Its premise is a set, the sites that parse the template base, and the fix covers all of it. No findings.

- **Parse sites:** the only code that parses a base is `parseMappingRoot(base, "base")` at `internal/yamlengine/follow.go:50`, and that call is now guarded by `haveBase`. No other site in `internal/yamlengine` or `internal/configsync/configsync.go` parses a base.
- **Callers:** `FollowTemplate` has one non-test caller, `configsync.go:1046`. It passes `templateBasePresent && !fileAbsent`, so the absent-file case now reaches the guard with `haveBase` false.
- **`ErrInvalidBase` handling:** the check at `configsync.go:1047` is still reached only on the `haveBase` true path, so it still holds.

One note outside the finding's premise: `readTemplateBase` at `configsync.go:1022` runs before the `haveBase` decision. An unreadable base (a directory in its place) can therefore still fail an absent-file reconcile. That is a read failure, not the parse failure F1 names, and a `baseIsDir` row pins it as a failure, so I raise no finding.

Tool calls made:
1. `grep` for `templateBase`, `ErrInvalidBase` and `parseMappingRoot` in `internal/configsync/configsync.go` and `internal/yamlengine/*.go`, non-test files. I ran it to enumerate every base-parse site and caller. My context lacked the `yamlengine` call sites other than `FollowTemplate` and the parts of `configsync.go` outside the read spans.
