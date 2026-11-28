---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/testkit/gofmt_test.go#gofmtFinding
    summary: gofmtFinding reports no finding for a source that differs from its gofmt form only by a missing final newline or extra trailing blank lines, so the tier0 guard passes a file gofmt would rewrite
    fix: "In gofmtFinding, after the line loop, return a finding instead of \"\", false: the bytes already differ there, so name the line where the line counts diverge (the line after the shorter slice's last real line) with a message that the file's ending differs from its formatted form, and keep the same `run \"lym loom fmt <file>\"` hint. Add rows to the table in TestGofmt_ModuleFilesAreFormatted for \"package a\" (no final newline) and \"package a\\n\\n\\n\" (extra trailing blank lines), each expecting a finding that names a.go and the lym loom fmt hint."
  - id: F2
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:19
    summary: The change description says the check reads every Go file of the module, but scankit's walk skips every directory whose name contains testdata, so the Go files under internal/planglyph/testdata and other fixture trees are never checked
    fix: "In _lym/landing/summary.md, in the Design decisions bullet that opens \"The check stays whole-tree\", replace \"it reads every Go file of the module\" with \"it reads every Go file of the module outside testdata directories, which hold fixtures\"."
---

### [MEDIUM] Trailing-newline differences pass the guard

**Location:** `internal/testkit/gofmt_test.go:39-62` (`gofmtFinding`)

**Issue:** When `format.Source` output differs from the source only at its end, `bytes.Equal` fails, but the line loop finds no differing line.
For `"package a"`, the split gives `["package a"]` against `["package a", ""]`;
at index 1 the out-of-range `have` defaults to `""`, which equals `wanted`, so the loop ends and the function returns `"", false`.
`"package a\n\n\n"` against `"package a\n"` ends the same way.
The guard therefore passes a Go file that `gofmt -l` lists, which contradicts the goal of failing tier0 on a mis-formatted Go file and the test's own doc comment.
A file written without a final newline, as an agent's file-writing tool can produce, is the realistic case.
The logic came over unchanged from `cmd/lym/gofmt_test.go`, but this diff adds it at the new site, where it now gates every task.

### [LOW] Change description overstates the walk's reach

**Location:** `_lym/landing/summary.md:19`

**Issue:** `scankit`'s `skipDir` drops every directory whose name contains `testdata`, as well as `_lym`, `.lym` and the other skip names (`internal/testkit/scankit/scankit.go`, `skipDirNames`, `skipDir`).
The tree holds 11 Go files under `testdata` directories in `cmd`, `internal` and `tools`, and the guard reads none of them.
The description's "it reads every Go file of the module" is therefore not true as written.
Leaving fixtures out is a reasonable choice; the claim should say so.

## Rejected

- code-scope F1 (the `lym loom fmt <file>` fix hint is unrequested scope): the goal asks for a tier0 formatting check, and the message a failing check prints is part of that check. Naming the sanctioned fix verb, which the edit directive already prescribes, makes the check usable now that every task hits it. It is not a separate change.
- Missing `Root-Cause:` trailer (handler consideration): the entry asks for a new tier0 check of a gap in gating, not a fix of a defect in existing code, so the trailer rule for bug fixes does not apply.
- Summary claim that the `internal/testkit` test binary "imports only the test kits" (handler consideration): read as direct imports, which the test files bear out (`scankit`, `tiergate`, `tmuxkit`). The transitive imports come through `tmuxkit.Main`, which the package already used.

## Focus departures

- "Gate reach: ... a diff that does not select internal/testkit and a diff whose only mis-formatted file is a test or build-tagged file?": settled by reading `internal/gatecli/scope.go` (card, diff, inert-only and whole branches, `scanGuards`) and `internal/impactset/guards.go`, and by running `lym gate test ./internal/testkit` at tier0, which ran the guard and passed. No synthetic mis-formatted diff was driven, because this reviewer's write surface allows no change to a target file. The walk uses `Filter: scankit.All` with no build tags, so test and build-tagged files are read.
- "Repository rules: ... Check marker placement (guard, watch, only), whether internal/testkit is an allowed home for a whole-repository guard": `lym board get` refused without a payload, so the board entry was read from the decision record's `## Goal`, which holds its body.
