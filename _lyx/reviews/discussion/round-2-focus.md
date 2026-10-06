---
round: 2
focus:
  - "In decision-record.md, check that the rerun and the report replacement are specified as `cmd/testtiming` can actually do them: its `-pkg` flag takes a single go pattern and `-out` rewrites the whole file, so settle whether the stated invocation count and the splice into `docs/benchmarks/test-redundancy.md` are achievable and say who performs the splice."
  - "In decision-record.md, check that every place a reason is kept (a kept candidate, an unmade tier move, a deleted candidate whose assertion moved) names where the reason durably lives, and that a deletion carrying unique assertions cannot pass without a recorded receiving test."
  - "In decision-record.md, check that the keep criterion is checkable given the tool lists candidates regardless of a `//testtiming:keep` directive, and that the repeated rerun-then-decide pass has a stated stop."
  - "In decision-record.md, check that a required reference update or deletion landing in a core directory or `internal/testkit/` has a stated winner against the no-dangling-reference acceptance criterion."
  - "In decision-record.md, check that the coverage method states whether `-coverpkg` is used and that its rule for explained coverage drops agrees with its rationale, and that the unused-helper rule states which build tags it considers."
---
