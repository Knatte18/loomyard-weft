---
round: 1
focus:
  - "internal/stencilstore/skew.go `shippedOverNewer`: settle whether `Classify(onDisk, true, def)` returns `StateUntouched` only for a copy whose stamp matches its own body, and whether a copy with no parseable writer or stamp can fall back wrongly."
  - "Every place stencilstore reads a stencil or include from disk (`Read`, `readInclude`, `ReadSet`, `ReadForEngine`, any other `os.ReadFile` of a board copy): settle whether each one now goes through `contentForRunning`, and whether a read path left out can still mix a board include with a shipped body, which is the failure the Sketch names."
  - "internal/stencilstore/engineblock.go `readInclude`: settle that `EngineBlockName(engine, block)` is the key `Registry.Default` and `Classify` expect for an engine block, so a block is neither always unknown nor mismatched."
  - "`RegisterShipped` and the `shipped` global: settle whether the package-init registration in contracts/stencils is data-race free against concurrent reads, and whether the panic on a second call can fire in a test binary or a second importer."
  - "`contentForRunning`: settle that the once-per-stencil log and the `runningWriter` read behave under concurrent calls and when `buildvcs.Running()` has no revision; the diff's only test covers the pure `shippedOverNewer`, so check which of these behaviors the diff adds with no coverage."
  - "Scope against the Sketch: the Sketch asks for early detection of the skew with an error naming the way forward, or a safe fallback to the shipped stencil. Settle whether the fallback is safe where a newer board copy is edited, equal-time, or written by a binary with a later commit time but an unrelated branch."
  - "Doc comments in doc.go, reconcile.go `Read` and skew.go against the repository's own comment conventions and the claims they make about behavior (the `Read` doc now says it falls back; check it matches the code, and that doc.go names the same rule)."
---

Seed pass: the diff is small and confined to `internal/stencilstore` plus one `init` in `contracts/stencils`.
The focus entries point at read-path completeness, the writer-time comparison and test coverage of the wiring.
