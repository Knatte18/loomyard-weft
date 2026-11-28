```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the diff's changed functions and branches.

**Cases I ran**

- `gofmtFinding`, source already formatted (`"package a\n\nfunc f() {}\n"`): `bytes.Equal` holds, so it returns `"", false`. Correct.
- `gofmtFinding`, empty source `""`: `format.Source` fails with "expected 'package'", so it returns `"<file>: does not parse: …", true`. Correct. The parse branch has no fix hint, which is right, because `lym loom fmt` refuses a file that does not parse (`internal/loomcli/fmtverb.go:73-76`).
- `gofmtFinding`, a mis-spaced line (`x:=1`): it returns the line number, the current and formatted text, and `run "lym loom fmt a.go" …`. Correct.
- `gofmtFinding`, a source missing its final newline (`"package a"`): **wrong.**
  - `format.Source` gives `"package a\n"`, so `bytes.Equal` is false.
  - The lines split to `["package a"]` against `["package a", ""]`.
  - At i=1, `have` falls back to `""` because the index is out of range, so it equals `wanted == ""`.
  - The loop ends and the final `return "", false` reports the file as formatted.
- `gofmtFinding`, extra trailing blank lines (`"package a\n\n\n"` against `"package a\n"`): **wrong** for the same reason. Every line past the end of `want` reads `""`, and so do the source's trailing empty lines, so it returns `"", false`.
- `gofmtFinding`, the suggested command's path: `f.Rel` is relative to the module root. The only root `go.mod` is the worktree root (`find` showed `./go.mod`; the nested module is under `testdata`, which the walk skips). So the path is relative to the worktree, and `lym loom fmt` resolves it against a cwd at the worktree root (`fmtverb.go:47-50`). Correct for an agent standing at the worktree root.
- `TestGofmt_ModuleFilesAreFormatted`, the walk from `internal/testkit`: `startDir` is that package's directory, and `findModuleRoot` walks up to the repository's `go.mod` (`scankit.go`, `ModuleRoot` / `findModuleRoot`). So the root and the skip set are the same as from `cmd/lym`. Correct.
- The marker change: dropping `//lym:only` leaves `//lym:guard` directly above `func Test…` after the doc comment's `//` line. That is a valid placement, and with no `//lym:only` the guard step at every key takes the test. Its tier from `tiergate.Require(t, 0)` is 0, so no `GuardWrongTier`. Correct.
- The floor check: `RequireFloor(t, scanned, 500, …)` behaves the same in the new package. Correct.
- The help text: it names the existing path `internal/testkit/gofmt_test.go`. Correct.

**Findings**

1. MEDIUM, design, `internal/testkit/gofmt_test.go:39-62` (`gofmtFinding`):
   - A file whose only difference from its gofmt form is a missing final newline, or extra trailing blank lines, comes back as formatted (`"", false`) even though its bytes differ. So the guard passes a file that gofmt would rewrite. That contradicts the answer key (fail any Go file gofmt would rewrite) and the test's own doc comment ("fails for every Go file … that differs from its gofmt form").
   - Evidence: the loop at lines 49-60 fills an out-of-range line with `""` and treats it as equal to a real empty line, then the unconditional `return "", false` at line 61 runs after the `bytes.Equal` check at line 44 has already shown the bytes differ.
   - The logic came over unchanged from `cmd/lym/gofmt_test.go:28-51`, but the diff adds it at its new site, and the guard now gates every task's tier0.
   - Fix: once the bytes differ, the function must return a finding. If no line differs, name the line where the line counts diverge (or report a trailing-newline difference). Add a table row for the `"package a"` source.

Tool calls: none; my context already held everything this lens needed.
