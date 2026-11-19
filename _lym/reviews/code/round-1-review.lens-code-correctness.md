```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each function and branch the diff adds or changes, step by step, with concrete inputs.

**Function / branch, case run, verdict**

- `RegisterShipped` (`internal/stencilstore/skew.go:29-34`)
  - First call from `contracts/stencils` init: `shipped` is nil, so it gets set. Correct.
  - Second call: panics. The only call site is the single package `init` at `contracts/stencils/stencils.go:1140-1142`, which Go runs once per binary, test binaries included. Correct.
  - Data race: `shipped` is written only during init, and init happens-before `main` and every test. The unsynchronised reads are safe. Correct.
- `runningWriter` (`skew.go:18-21`)
  - A binary with no VCS stamp (any test binary): `buildvcs.Running()` returns the zero `Identity`, so `Time` is zero. `shippedOverNewer` then exits at `running.Time.IsZero()`. Correct.
  - `sync.OnceValue` makes it safe to call concurrently. Correct.
- `contentForRunning` (`skew.go:39-52`)
  - `shipped == nil` (the stencilstore tests, and any binary that never imports `contracts/stencils`): returns `onDisk` unchanged, so the old behaviour holds. Correct.
  - Fallback taken, two concurrent reads of the same name: `newerCopyWarned.LoadOrStore` lets exactly one of them log. Correct.
  - The log's `written_by` field is `ParseWriter(onDisk).Revision`, the writer the decision used. Correct.
- `shippedOverNewer` (`skew.go:59-72`), each guard in turn:
  - Name the registry does not ship: returns `onDisk, false`. Correct.
  - Running binary has a zero time: returns `onDisk, false`. Correct.
  - Copy with no stamp, or a stamp without writer keys: `ParseWriter` returns the zero `Writer`, `Time.After` is false, so it returns `onDisk`. Correct.
  - Writer time equal to running time: `!After` is true, so it returns `onDisk`. Correct.
  - Same revision: returns `onDisk`. Correct.
  - Copy from a newer writer whose body is edited: the stamp no longer matches the body, so `Classify` gives `StateEdited` (or `StateReconciled`) and it returns `onDisk`. Correct. `Classify` (`stencilstore.go:315-329`) returns `StateUntouched` only when the parsed stamp equals `BodyHash(onDisk)`, so a copy with no parseable stamp can never pass this guard.
  - Untouched newer copy whose body equals the shipped body: the `BodyHash` equality check returns `onDisk`. Correct.
  - Untouched newer copy whose body differs: returns `def, true`. Correct.
- `Read` (`reconcile.go:31-38`)
  - Missing file: the error is still wrapped and returned, and `contentForRunning` is never reached. Correct.
  - Fallback: returns the embedded default, which has no stamp banner. I found no `Read` caller that parses a stamp, writer or hash from the bytes it gets back, so nothing depends on the banner. Correct.
- `readInclude` engine-block path (`engineblock.go:164`)
  - Key: `EngineBlockName("claude", "assist-fan")` is `engine/claude/assist-fan`, which matches the registry entry at `contracts/stencils/stencils.go:1010`. The block is therefore neither always unknown nor mismatched.
  - `bytes.TrimSuffix` on the fallback slice returns a subslice and does not modify the embedded default. Correct.
- `readInclude` plain-include path: it goes through `Read`, so each include gets its own fallback decision. Correct.

**Findings**

1. LOW, class `design`, at `internal/stencilstore/skew.go:65` (the `!recorded.Time.After(running.Time)` guard): a later commit time stands in for "newer", so the check misses skew whenever the running binary's commit time is later but its source lacks the writer's stencil change.
   - Example: a binary built from a branch cut before the writer's commit but committed after it. It reads the newer copy as it stands, so the bare template error the Sketch names can still occur.
   - Rendering in the reverse case is safe, since the binary always renders its own text.
   - This is a gap in detection, not a regression.
   - Evidence: `skew.go:65`, and the `buildvcs.go` `identityOf` mapping of `vcs.time`. The summary (Design decisions) accepts the trade-off with "Ordering is by commit time because the read path runs no git".

No other correctness finding. I found no swallowed error and no inconsistent state on any failure path.

**Tool calls**

1. A grep of `contracts/stencils/stencils.go` for the engine-block registry entry names. My context had the `//go:embed` lines but not the `entries` names, and I needed them to settle the `EngineBlockName` key question.
2. A grep of production callers of `stencilstore.Read` for a `ParseStamp`, `ParseWriter`, `Classify` or `BodyHash` use, or an in-place write, on the returned bytes. My context lacked these call sites, and I needed them to check whether a returned default with no banner, or a shared embedded slice, breaks a caller.
