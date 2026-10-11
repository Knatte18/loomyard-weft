# Darn: two of the prompt's pointers were wrong

The prompt says to read the task with `lyx board get seats-chair-first`, but `board get` takes a JSON argument; the bare slug fails with `invalid json`, and the working form is `lyx board get '{"slug":"seats-chair-first"}'`.

The prompt also names the final-summary spec at `<hub>/_board/_lyx/specs/final-summary-spec.md`, but that file does not exist; `_board/_lyx/specs/` holds only a `loom` directory.
The spec lives in the repository at `contracts/specs/final-summary-spec.md`, and I used that copy.
