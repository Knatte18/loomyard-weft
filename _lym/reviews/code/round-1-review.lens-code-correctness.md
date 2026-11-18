```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each changed function and branch by hand on concrete inputs.

**Per function or branch: the case run, correct or not**

- `gateslot.Config.Gate`, tierless hub (`Tiers` nil), key `batch` with tiers `[0]`: returns `GateKey{Scope: "card"}` with no tiers and no error. Correct.
- `Config.Gate`, tierless hub, key `nope` that the config lacks: refused with `gates holds no key "nope"`, because the missing-key check now runs first. Correct.
- `Config.Gate`, tierless hub, key with an empty tier list: refused as an empty list. Correct, since the empty-list check comes before the tierless return.
- `Config.Gate`, tiers `{tier1}` with no `tier0`: refused with "declares no tier0". Correct, and the partial-tier case is still refused.
- `Config.Gate`, tiers `{tier0}`, key lists `[0, 2]`: refused, naming the undeclared tier2. Correct.
- `TierList`, `renderGateCall`, `GateCall`, `AgentGateCall`, tierless hub: `TierList` returns nil with no error, and the render now produces `lym gate test --gate &lt;key&gt;`, so shed build no longer fails. Correct.
- Callers given an empty tier list: `landingshed/verifygate.go:117-122` handles it (`len(list) == 0` → Warn and no record). `verifygate.go:405-410` falls back to a message that names no tier. `webstercli/beginbatch.go:76` reads only the error. `verifytree.Verify` reads only the exit code and its record (`verify.go:326-372`). All safe.
- `runTest` early return, `--gate finalize` in a tierless hub: `gateKeyRefusal` passes, the line goes to `request.stderr`, and the call exits 0 before any slot, plan or record. A card-scope key with no `--card` is still refused first. The exit is correct, but the return also skips the key's lints (finding 1).
- `runTest`, tierless hub with no `--gate` (for example `./pkg`): no early return, and `planTierRuns` still refuses on `cfg.Tier(0)` (`tiers.go:91`). Correct.
- `computeScope` → `allScope`, explicit `--all` (`request.gate == ""`): returns whole. Correct, and `--gate` alongside `--all` is refused upstream by `flagShapeRefusal`.
- `allScope`, `--gate finalize`, hub with no `inert_paths`: returns whole. Correct.
- `allScope`, `--gate finalize`, `parentMergeBase` fails: the stderr line names the failure and the scope stays whole. Correct.
- `allScope`, prime with no origin or parent branch (`ok` false): whole, with an Info log only. Correct.
- `allScope`, the `ChangedPaths` call fails: `allInertSince` logs at Warn and returns false, so the scope stays whole. Correct, and it matches the existing computed-scope path.
- `allScope`, empty diff since the merge base: `cfg.AllInert` gets an empty list and the doc says that reports false, so the scope stays whole. Correct, since a whole run is the conservative choice.
- `allScope`, an inert `docs/note.md` beside an untracked `lib/x.go`: the untracked file counts, so the change is not inert-only and the scope stays whole. Correct.
- `allScope`, inert-only diff: returns `scope{form: scopeAll}`, so `whole` is false and no modules are set. Correct.
  - In `scopeInvocations` (`tiers.go:240-247`), a tier with `NoCarry` runs whole and any other tier gets nil invocations.
  - `runListed` (`run.go:545-549`) then prints "the scope is empty, so tierN runs no command" and runs nothing through `runTier`.
  - With `SharedBuild` under dotnet, `buildOnly` is false because `noTests` is false, so no build runs.
  - `guardStep` stays false, as for the whole `--all` scope before this change.
  - The green record gets no entry, since `runGreenAware` is not reached. That only drops a possible base for later runs, which is conservative.

**Findings**

1. **MEDIUM, design.** `internal/gatecli/run.go:83-86`: in a hub that declares no tier, a `--gate` call exits 0 before the gate key's lint steps run, so the `comments` and `pattern` lints a key owes are skipped silently.
   - Evidence: the template's card key carries lints (`internal/gateslot/template.yaml:28`, `card: {tiers: [0], scope: card, lints: [comments, pattern], ...}`).
   - Lints run after the tiers against `request.root`, whatever tiers exist (`run.go:213-223`; `failurereport.go:110-125`, where `runLints` takes only root and names).
   - The early return comes before `request.root = buildRoot(...)` at `run.go:88` and before the lint loop. A tierless hub's card gate therefore passes a comment-lint or PATTERN-headroom violation it would otherwise catch.
   - Fix: when there are no tiers, skip the tier plan but still run `runLints` over the key's lints, and set the exit code from them.

No other correctness finding.

**Tool calls**
- Read `scopeInvocations` and the `scopeAll` uses in `internal/gatecli/tiers.go`: needed to see what an empty `scopeAll` scope plans per tier; the manifest stopped at `planTierRuns:170`.
- Read `runListed` and `runGreenAware` in `internal/gatecli/run.go:541-581`: needed to confirm an empty-invocation tier runs no command and writes no green entry; not in the manifest.
- Read the `runTest` callers and `runLints` in `internal/gatecli/failurereport.go:110-125`: needed to confirm lints do not depend on tiers; not in the manifest.
- Grepped `verifytree.Verify` for output parsing: needed to know whether a silent exit 0 is read correctly by verify sites; not in the manifest.
- Read `internal/verifytree/verify.go:326-372`: the grep was inconclusive, so I read `Verify`'s result handling directly.
