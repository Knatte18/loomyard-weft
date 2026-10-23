# Card 8 — fabric-line-change

**Create:**
- `plan:internal/fabricengine#LineChange` -> `type LineChange struct`
- `plan:internal/fabricengine#Fabric.LineChange` -> `func (f *Fabric) LineChange(from, to string) (LineChange, error)`
- `plan:internal/fabricengine_test#TestLineChange_CountsLinesAndListsPaths` -> `func TestLineChange_CountsLinesAndListsPaths(t *testing.T)`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#Fabric.ChangedPathsBothSides`

**Intent:** Give the fixer caps (D4, D10) the one git read they judge: the tree diff between two commits of the pair's code repository, in-process through `gitexec` as every fabric read is (`PATTERN-fabric-git`).
`LineChange` holds `Paths`, the sorted changed paths with a rename counted at both its paths as `ChangedPathsBothSides` reads them, `Lines`, the added plus deleted lines from `git diff --numstat -M` (rename detection on, so a pure rename counts no lines, and a binary file counting as one line, so it is never free), and `AddedLines`, the text of every added line from the unified diff, which the kind cap scans for a test skip or a build constraint.
`Fabric.LineChange(from, to)` returns it, and a read failure is an error naming both commits.

The integration test commits a change of three added and two deleted lines in one file, a pure rename of an unchanged file and a binary file on a hub fixture, and asserts the paths (the renamed file at both paths), a line count of six (the five text lines plus the binary file's one, the pure rename adding none), and the added lines' text.

`doc.go` lists the read beside `ChangedPaths`.

**ImpactSummary:** One additive read verb and a doc line; no caller yet.
