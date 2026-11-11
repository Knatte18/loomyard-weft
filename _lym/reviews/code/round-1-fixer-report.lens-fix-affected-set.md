```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- **F1, F2, F3 (old title forms):** a repo-wide grep over `.go`, `.md` and `.yaml` files for the old shapes found no remaining match. The shapes were `ly: P…`, `<hub shortname>: orch`, `: orch"`, `P3: T…` and `<P#>: <T#>`. I excluded `_lym/` and `.lym/`.
- **F4 (callers of `Title`):** the callers are `Workspace` in `consoletitle.go:48` (name is `name+" workspace"`, never empty), five calls in `internal/boardengine/handles.go:196-222`, and `smoke_drivingsurface_test.go:254`. The new rule drops an empty `name` along with its separator. The `handles.go` calls pass a subject, slug or name, so none changes behaviour. The smoke test passes `RecipeTapestry`, which is non-empty.

I made 2 further tool calls:
1. A grep for the old title-form strings across the repo, rerunning the F1/F2/F3 set. My context held only the files the findings named.
2. A grep for every `Title(` call site, rerunning the F4 set. My context held only the `Title` definition and the tests.
