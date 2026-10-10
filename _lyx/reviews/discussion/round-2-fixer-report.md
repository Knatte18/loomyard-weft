# Round 2 fixer report — discussion

Every finding was checked against the code and fixed.

## Fixed

- **F1 (BLOCKING, design).**
  Confirmed: `renameDeclSource` in `internal/planglyph/handle.go` is Found-only by design and refuses `multipart` for the same reason a partitioned answer would need refusing.
  Decision: a Rename's old side stays Found-only.
  Scope item 6 now names a Rename's new side only and says the old side's refusal names the way forward.
  D6 states the rule, its reason, and the `rename-old-unresolved` detail for a partitioned old side, naming candidates, constraints and the Delete-and-Create or per-file Edit way forward; the bound now says a Rename's old side never accepts several declarations.
  The acceptance criterion drops Rename-old, and a new criterion pins the refusal.
  The Constraints entry for `PATTERN-refusal-way-forward`, Auto-mode assumptions, Notes for the plan writer, support-log question 9, Rejected alternatives and the question ledger follow.
- **F2 (MEDIUM, design).**
  Confirmed: `go/build`'s `matchTag` returns true for `ctxt.Compiler`, `ToolTags` and `ReleaseTags` whatever `BuildTags` holds.
  D6 now states each environment is a zero `build.Context` with only `GOOS`, `GOARCH`, `CgoEnabled`, the true free tags in `BuildTags` and an `OpenFile`; `Compiler`, `ReleaseTags` and `ToolTags` stay empty, so `gc`, `gccgo`, `go1.N`, `goexperiment.*` and `amd64.vN` are free tags.
  It also states that `boringcrypto` is assigned as `goexperiment.boringcrypto`, the name `matchTag` looks it up under, since assigning it under its own name would fail open the same way.
  A criterion row pins `!go1.21` and `!gc` beside an unconstrained file staying `glyph-ambiguous`; a rejected alternative records `build.Default`.
- **F3 (LOW, decision).**
  The cap is four free tags, stated in D6, the criterion (five free tags is not partitioned), Open risks, Auto-mode assumptions and the ledger.
- **F4 (LOW, consistency).**
  Confirmed: `contractDeleteClause` says "a fork wrote", and the refusal-spec rows 112, 115, 116 and 120 and the run-exit audit row say a fork wrote it last.
  D3's bound now says the uncleared contract-file refusal, audit text and those refusal-spec rows name the writer as a fork or a recovery session.
  The line-152 criterion asserts both wordings, and the Constraints and plan-writer notes name the sites.
- **F5 (NIT, consistency).**
  Confirmed: `MatchFile` with a nil `OpenFile` opens the file on every call.
  D6 says each candidate is read once and served through `OpenFile`, and the rationale's cost sentence now states one read per candidate and the bounded in-memory matcher calls.

## Disputed

None.

## Deferred

None.
