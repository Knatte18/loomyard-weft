# Darn prompt: board read command and summary spec path

The darn prompt says to read the task with `lyx board get ide-opt-in`, but `board get` takes a JSON object, so the bare slug fails with `invalid json`; `lyx board get '{"slug":"ide-opt-in"}'` works.

The prompt also names the final-summary spec at `<hub>/_board/_lyx/specs/final-summary-spec.md`, which does not exist; the spec lives in the repository at `contracts/specs/final-summary-spec.md`, and that copy was used instead.
