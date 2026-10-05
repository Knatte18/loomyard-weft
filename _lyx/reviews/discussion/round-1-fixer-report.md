# Round 1 fixer report

## Changed

All edits are in `_lyx/discussion/decision-record.md`.

- F1: D3 now states that `Set` under a list-shaped open-map value refuses with a rewrite-as-map message and never converts, and that reconcile still carries the list whole.
- F2: D3 states what the open map lets through (no value inspection by `MissingKeys`, a mistyped `--set` name creates a label); D1 states that decode refuses a value that is neither map nor sequence, and a non-scalar description.
- F3: D7 states the carry runs before `validateWrite` and that a number also held by an untouched third entry is carried as-is.
- F4: D9 states that a subcommand wins over a same-named module, and that bare `--print` and `--set` keep their present path.
- F5: Scope In now names the picker removal and the allowlist deletion.
- F6: R3 (burler maps) deleted as a repeat of the Scope Out bullet; the former R4 is now R3.

## Deferred

Nothing.
