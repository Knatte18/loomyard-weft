---
round: 3
focus:
  - "D6 in `_lyx/discussion/decision-record.md` resolves a partitioned glyph answer like `multipart` for `statusFindings`, and Scope item 6 says Rename accepts a partitioned member. A Rename old side derives its declaration through `renameDeclSource` in `internal/planglyph/handle.go`, which requires `quarry.StatusFound`. Settle whether D6 decides how a partitioned old side derives its declaration, or drops Rename-old from the scope, and whether the acceptance criteria match that decision."
  - "D6 defines an environment set built with `go/build`'s `Context.MatchFile`. `matchTag` treats `Compiler`, `ReleaseTags` and `ToolTags` as always true whatever `BuildTags` holds. Settle whether D6 states the base context each environment starts from, and whether a negated `go1.N` or `gc` constraint beside an unconstrained file is classified so that coexisting candidates are never passed as partitioned."
  - "D6 bounds the weakened ambiguity guard by a cap on free tags. Settle whether the cap and the enumeration cost it bounds are stated in D6 as decided values, and whether the Open risks entry and the acceptance criterion that depend on the cap agree with them."
  - "D3 in `_lyx/discussion/decision-record.md` keeps the contract-file finding standing after a recovery write to `outcome.yaml`. The refusal built by `contractDeleteClause` in `internal/websterengine/contractevidence.go` names the writer as a fork. Settle whether D3 and its acceptance criterion cover the refusal wording for a recovery writer, as `PATTERN-refusal-way-forward` requires."
---

Round 3 re-checks the D6 and D3 decisions that round 2 touched.
The D1 deadline outcome and the D3 reset own-path set were cleared in round 2 and are not repeated here.
