```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the code and tests the fix changed (F3, F4, F5, plus the F1 and F2 text) for correctness.

No findings.

- **F4 row** (`runsettings_test.go`): `Recipe: "loom"` with no profile and `profilesYML: "light: 3\n"` goes through `Load`. `decode` refuses `light` as not a mapping, with a message naming `profile "light"`, which matches `wantStuck`.
  If `prepareRunSettings` skipped loading for a profile-less task, this row would see no Stuck and fail, so it pins the behaviour.
- **F3 test** (`TestTemplateDocsProfile`): the `want` values match `template.yaml` lines 35-38. YAML decodes the integers to `int`, the same type `TestTemplateLadder` asserts. The tier-0 run passed.
  Changing the docs writer, its rounds or its hard cap breaks `DeepEqual`, and adding a loom block breaks the `Settings("docs","loom")` error check.
- **F5 comment**: it matches `TestLoad`'s "a profile named defaults is refused" row, which expects the `recipe "loom" must name a profile` error from `decodeDefaults`.
- **F1 and F2**: they change only comment and help text, which no caller reads. The `defaults` key and the `"ok":true` envelope in the `doc.go` text match `cli.go:390` and the help example at `cli.go:376`.

Further tool calls: none, because my context already held every changed file and the passing test output.
