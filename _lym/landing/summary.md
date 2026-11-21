# Trim superfluous text from every package doc.go

The package `doc.go` files held 27,301 comment lines, and the earlier comment rewrite did not shrink them.
This change cuts what is superfluous in 130 `doc.go` files: text that restates function or type docs,
narrates how the code works step by step, repeats itself, or records history.
Each file keeps a short package overview, the package's contracts and invariants, and a "why" the code alone does not show.

The diff touches only `doc.go` files and only comments: 4,812 lines added, 21,016 removed.
The largest cuts are in `websterengine`, `reedengine`, `shuttleengine`, `shuttleengine/claudeengine` and `shedadapters`.
No code changes, and comments in other files are out of scope.

## Design decisions

- The cut applies to every `doc.go`, not to a chosen set of packages.
- Where a `doc.go` and a symbol doc said the same thing, the symbol doc stays and the `doc.go` text goes.

## To check by hand

- `internal/adoptcli/doc.go`: the adopt's step order and the envelope fields it names are complete and match the code.
- `internal/batcher/doc.go`: it no longer restates the docs of `Active` and `PeakContext`, and it defines the start base.
- `internal/yamlengine/doc.go`: it no longer restates function docs, and the `Reconcile` rationale reads correctly.
- Skim the largest cuts for a contract or invariant removed along with the narration.

## Merge danger

**Door:** two-way
**Blast radius:** docs
