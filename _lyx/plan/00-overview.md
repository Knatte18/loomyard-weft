---
format: 5
approved: true
language: go
---

# Plan: inserting a card, unreachable prescribed refusals

Close the two plan gaps the decision record names: give an agent editing a plan sanctioned verbs, `lyx loom insert-card`, `lyx loom remove-card` and `lyx loom restore-renumber`, that insert or remove one card and renumber the cards after it under a journal that makes an interrupted call restorable, bounded by the frozen begun cards and webster's state-mutation lease, with their rows in the refusal table and the docs that describe them;
and pair a reachability rule for prescribed refusals between the Plan-Write stencil and the Plan-Review rubric, so an unreachable refusal is a plan-time finding.
The renumber lives in `internal/planparser`, the verbs in `internal/loomcli`, and the begun bound reads webster's run record.

## Card Index

1 — begun-card-bound — add `websterengine.LastBegunCard`, the frozen-card bound read from the run record
2 — planparser-renumber — add the plan scratch paths, the raw I/O seam, the journal and the insert and remove renumber to `planparser`
3 — planparser-restore — add `planparser.RestoreRenumber`, the journal's only reader
4 — loom-card-verbs — add the three `lyx loom` verbs, their refusal rows, the lease and the docs
5 — plan-stencil-rules — tell the writer, rework and review stencils about the verbs and pair the reachability rule

## Shared Decisions

### Decision: scratch-path-ownership

- **Decision:** `planparser` declares the plan directory's never-tracked mirror and the journal path by joining `lyxdirs.DotLyxDirName` onto its own segment, exactly as `PlanDir` joins the durable form; no new file spells `_lyx` or `.lyx`.
- **Rationale:** `internal/lyxdirs` declares only the two directory tokens, and every module's scratch accessor is built from them beside its durable one; that is how the decision record's "declared in `internal/lyxdirs`" is honoured under `PATTERN-lyxdirs-single-declarer` and `PATTERN-durable-vs-ephemeral-state`.
- **Applies to:** cards 2, 3, 4

### Decision: raw-io-seam

- **Decision:** the renumber performs every raw file operation through `planparser.RenumberIO`, a struct of five function fields filled from `os` by `OSRenumberIO`; a test fails one named operation on one named file to reach each reversal and restore path.
- **Rationale:** the acceptance criteria inject a failure at every apply step, and a struct value passed explicitly keeps the tests parallel, where a package-level seam would not.
- **Applies to:** cards 2, 3, 4

### Decision: refusal-wording-split

- **Decision:** `planparser` returns sentinel errors and never names a `lyx` verb; `loomcli`'s `renumberRefusal` is the one place that words each verb's message and way forward, and the refusal rows land with the verbs in card 4.
- **Rationale:** the way forward of a plan-directory refusal is a `lyx loom` verb, which the parser has no business naming; one wording site keeps the message and the table row from disagreeing.
- **Applies to:** cards 2, 3, 4

### Decision: move-mechanics

- **Decision:** a moved card is written under its new name, then its old name is removed, in the apply order the decision record fixes; `Rename` is used only to move the temporary journal into place.
- **Rationale:** the journal's `Created` list and `Files` map reverse either half of a move, and an atomic rename is what makes a journal on disk always complete.
- **Applies to:** cards 2, 3

### Decision: begun-bound-from-records

- **Decision:** the frozen bound is `websterengine.LastBegunCard`, read from every batch record's cards, terminal or not, with no batchifier.
- **Rationale:** it is the begun set `DispatchScope` derives, which the Plan-Review rubric and `lyx webster rebaseline` already freeze, not the done set `DoneCards` returns.
- **Applies to:** cards 1, 4

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
