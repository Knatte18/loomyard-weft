# Support log — lyx config --set refuses a value the module's loader rejects

## Interview

A darn run holds no interview: the writer read board entry `config-set-validate` (issue #620) and the code, and made every call itself.

## Rejected alternatives

- **A shape rule in `internal/yamlengine`, refusing a bracketed value at a key whose template value is a scalar.**
  It covers only the list case and none of the other values a loader rejects, such as a malformed model-spec scalar; the loader check covers both.
- **Checking only with `CheckValues`.**
  It runs on local-layer documents alone, checks a few named keys, and leaves model-spec grammar to the loader.
- **Validating the in-memory merged document instead of writing and rolling back.**
  The loaders read the stack from disk through `configengine`, and no loader takes a document; adding one per module would duplicate each loader's stacking.
- **Refusing on any after-write load failure.**
  A broken lower layer or an uninitialised base would block every `--set`, including the one that repairs it.
- **Running the load check in the editor paths too.**
  The board entry asks for `--set`; the editor paths keep their current checks.

## Review rounds

_No rounds yet._

## Question ledger

- Q: Which writes get the check? Self-pick: every `--set` layer, worktree, local and board-dir, since the bad value fails a reader on each.
- Q: What does a stack that already fails to load do? Self-pick: a write is refused only when it introduces a load failure the stack did not already have.
- Q: Where does the per-module loader hook live? Self-pick: a function field on `configreg.Module`, beside `CheckValues` and `Migrate`.
