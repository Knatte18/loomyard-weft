---
round: 5
exclude_lenses: []
focus:
  - "D8 in _lyx/discussion/decision-record.md (git and lyx read-only forms of the classifier, `internal/fabricengine/refscanner.go` `IsReadOnlyCommand`, which splits a segment with `strings.Fields` on the original text and rejects a quoted word only in first position): does the Decision, as written, make every accepted command incapable of writing a ref, config, remote or file when a word is quoted, backslash-escaped, brace-expanded, glob-bearing or a `$VAR`, in any position of any form? Settle whether the stated bound is true for all word shapes, not only dash-words and positional words."
  - "D12 in _lyx/discussion/decision-record.md and `internal/websterengine/beginbatch.go` (the terminal `dead` batch refusal text) with `contracts/specs/refusal-spec.md` (the `report already present` row): after D12's retry rung for a retryable dead batch, does every refusal or stencil sentence that still tells Master a dead batch is exhausted stay true, and are all reworded refusals named in the Constraints line for `PATTERN-refusal-way-forward`?"
  - "D7 guard refusal in _lyx/discussion/decision-record.md: does the refused fork or recovery strand get a step that is a verb, a run transition or a git command, as the refusal-spec defines a way forward?"
---

The D8 classifier is the one key open across four rounds; the focus targets the class (word shapes), not another instance.
