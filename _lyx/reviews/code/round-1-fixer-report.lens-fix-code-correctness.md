```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the F1/F2/F3 code changes for correctness, running concrete inputs through them.

**Findings**

1. **LOW**: `startsTypeDecl` in `internal/dotnetimpact/source.go` still declares a bogus type when `record` is a variable and the next tokens are `is {`.
   - Evidence: for `record is { X: 1 }`, `declaredName` returns name `is`, and `next` points at `{`. `{` is in the follower list, so the function returns true and a type named `is` is added.
   - This is a narrow residue of F2, and the direction is safe because it only over-selects.
   - The review's alternative fix (reject the names `in`, `is`, `as`, `when`) would close it.

**Traced and correct**

- **F1 (`lex.go`, `quotes >= 3 && !verbatim`):**
  - `@""""` has four quotes. The verbatim loop skips the opening quote, treats `""` as an escape, and `"` closes it.
  - `@"""Hidden1 ""x"" "` has three quotes. The loop reads the opening quote, `""` as an escape, the text, `""x""` as escapes, and a final `"` closes it. `Visible1` is lexed afterwards.
  - `@""` still hits the `quotes == 2` empty-string branch.
  - Raw strings (`$"""`) are unaffected, since `verbatim` is false for them.
- **F1 test row:** with the defect in place, `@""""` sends the lexer into `rawBody(4)`, which finds no closing run of four quotes and swallows the rest of the source. `Visible1` and `c` are then missing, and `Hidden1` stays absent either way. The row fails without the fix.
- **F2 (`source.go`):**
  - `record R(int X)`, `record R<T>`, `record struct S(...)`, `record class C {` and `record R : B` all end in a follower token and still return true. This matches the existing row near line 95.
  - `var record in rs` and `record is Foo` give name `in`/`is` with `next` at an identifier, so they return false.
  - `record.X`, `(record)` and `record =` give an empty name, so they return false.
  - `typeDecl` still gets the same `declaredName`.
  - `isPunct` is bounds-safe.
- **F2 test row:** with the defect in place, a type named `in` or `is` appears in `Types`. The row compares the full `want` list, so it fails without the fix.
- **F3:** the change is doc text only, with no code effect.

**Tool calls:** none. The context already held every file.
