# Darn prompt names a bare-slug board get and a nonexistent final-summary spec path

`darn-prompt-paths` · Note · bug, stencil

Imported from [issue #531](https://github.com/Knatte18/loomyard/issues/531).

The darn agent prompt contains two instructions that fail when followed literally.

- It says to read the task with `lyx board get <slug>`, but `board get` takes a JSON object, so the bare slug fails with `invalid json`.
  `lyx board get '{"slug":"<slug>"}'` works.
- It names the final-summary spec at `<hub>/_board/_lyx/specs/final-summary-spec.md`, which does not exist.
  The spec lives in the repository at `contracts/specs/final-summary-spec.md`, and the agent had to find that copy itself.

Both are abnormal because the prompt is the agent's authoritative instructions, so each wrong pointer costs a failed call or a search and risks the agent using the wrong source.

Fix the darn prompt stencil to emit the JSON-object form of `lyx board get` and the `contracts/specs/final-summary-spec.md` path.

Provenance: task slug `ide-opt-in`, note `Darn.md`.
