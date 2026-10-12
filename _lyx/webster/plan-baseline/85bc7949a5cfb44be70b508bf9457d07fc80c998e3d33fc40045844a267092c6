# Card 4 — Rename ReadJSONStrict for what still sets it apart

**Rename:**
- `internal/state#ReadJSONStrict` -> `internal/state#ReadJSONNoMkdir`
- `internal/state#TestReadJSONStrict` -> `internal/state#TestReadJSONNoMkdir`
- `internal/state#TestReadJSONStrict_MissingFile_NoMkdirAll` -> `internal/state#TestReadJSONNoMkdir_MissingFileCreatesNoDirectory`

**Edit:**
- `contracts/specs/loom-status-spec.md`

**Intent:**
D1: after card 2 the read is no longer strict, and what still sets it apart from `ReadJSON` is that it creates no missing parent directory, so it is renamed `ReadJSONNoMkdir`.
The function and its two tests are renamed; the tests' file, `internal/state/strict_test.go`, keeps its name, since the plan gate refuses a file rename beside renames of its members, and its header comment is reworded.

Every reference in the module follows the rename: the call sites in `internal/battencli`, `internal/loomcli`, `internal/loomengine`, `internal/loomshed`, `internal/pairteardown`, `internal/shedengine`, `internal/shedverbs`, their test files and those of `internal/battenrecipe`, `internal/loomrecipe` and `internal/shedcli`, and every comment that names it, such as in `internal/loomengine/seed.go`, `internal/shedengine/run.go`, `internal/shedverbs/spec.go`, `internal/battencli/arm.go`, `internal/loomcli/smoke_test.go` and `internal/state/doc.go`.
In `internal/state/strict_test.go` the test `TestReadJSONStrict_MissingFile_NoMkdirAll` is first renamed by hand to `TestReadJSONNoMkdir_MissingFileCreatesNoDirectory`, its declaration and every mention, and only then does `replace_all` run on that file.
Each file is changed with one Edit using `replace_all` on `ReadJSONStrict`, and the card ends with `grep -rn ReadJSONStrict --include=*.go .` finding nothing.
A comment that calls the read "strict", or says it rejects unknown fields, is reworded where the rename touches it: for example the test file's header comment, which describes "decode-strictness".
The comment on the `content` field in `TestReadJSONStrict`'s table points at `TestReadJSONStrict_MissingFile_NoMkdirAll` by name, which `replace_all` would turn into a name that does not exist, so it is pointed at `TestReadJSONNoMkdir_MissingFileCreatesNoDirectory` by hand.

`contracts/specs/loom-status-spec.md`'s "Strict, fail-loud parsing" paragraph says the read rejects unknown fields through `DisallowUnknownFields`; it becomes: the `internal/state` read (`state.ReadJSONNoMkdir`) rejects malformed JSON and accepts unknown fields, and a rewrite keeps them (D1, D2).
The paragraph after it, which says `product` is decoded after "the strict shell decode", drops "strict".

**ImpactSummary:** a pure rename across 47 Go files and one spec paragraph; no behavior changes.

**Commit:** 4: rename state.ReadJSONStrict to ReadJSONNoMkdir

**Verify:**
- `grep -rn ReadJSONStrict --include=*.go . ; test $? -eq 1`
