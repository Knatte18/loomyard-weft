```yaml
lens: code-affected-set
outcome: reported
```

**Lens code-affected-set: what each change forces outside its own lines, and whether the diff covers it.**

Per change:

1. **Batching sentence removed from `claudeengine`.**
   - Set found: `settings.go` constant, `init` check, `appendPromptText`, `claudeengine.go` and `command.go` comments, `doc.go`, `prepare_test.go`, `settings_test.go`, the `docs/overview.md` shuttle and creel paragraphs.
   - Searched the other docs, stencils and contracts for the sentence: only `docs/research` reports cite it, which are dated measurement records.
   - Covered, except F2.
2. **New `heddle` config module (`heddle.yaml`, `lym_override`).**
   - Set found: `configreg` entry, the `configreg_test` hub-wide flag row, the `configstrictness_test` degrading set, `PATTERN-config-layers` degrading list, `docs/overview.md` package tree, `template.yaml`.
   - The `configreg_test` per-module key maps (`wantLocalKeys`, `wantLocalDenied`) are left as they are. I take it heddle's key is a cross-pair category that the local layer refuses, but I did not check how those maps treat that category. Whether they still pass is the gate's to decide.
   - Covered.
3. **Shipped layer in render, show, passages, list and get.**
   - Set found: `target.go` file targets and re-render list, `cmd/lym/main.go` passage reader and seam, `PATTERN-heddle-instructions`, `PATTERN-lym-instructions-in-code` and `PATTERN.md`, `heddlecli` Long text and `doc.go`, `heddleengine/doc.go`.
   - Covered. The `PATTERN-durable-vs-ephemeral-state` and `PATTERN-fabric-isolation` mentions of heddle need no change.
   - Every worktree with the marker now gets a `CLAUDE.local.md` even with no repository entries. The only other reader is the Merriam base size, whose test uses a synthetic worktree. Covered.
4. **New Bolt file `heddle-lym-override.json`.**
   - Set found: heddle's Bolt replayer (`replayer.go:24`) and the replay classification in `fabricengine/replay.go:675` and `:700`. Not covered, see F1.
5. **`--layer` flag, `storeFile` plumbing, store load and save generalised.** Set found: `heddlecli/verbs.go`, `store.go`, `override.go`. Callers and tests are covered.

**Findings**

**F1 — LOW, design.** `internal/heddleengine/replayer.go:24` (evidence: `internal/fabricengine/replay.go:675-700`).
- Heddle's Bolt replayer still claims only `heddle.json`.
- A commit that touches `heddle-lym-override.json` is not claimed. `replay.go:675` requires every path to be claimed, so such a commit falls to a plain cherry-pick (`replay.go:700`).
- Two machines tuning the same override diverge with a cherry-pick conflict. `heddle.json` instead gets a by-entry merge.
- `PATTERN-heddle-instructions` says the Bolt files are written "only by `lym heddle` verbs and heddle's Bolt replayer", but the replayer never handles the override file.
- Fix: have `Claims` also accept `OverrideFileName` and `Apply` merge it by entry. Otherwise record in the pattern or `doc.go` that the override file is replayed by plain cherry-pick.

**F2 — NIT, consistency.** `internal/shuttleengine/claudeengine/doc.go:~222`.
- The doc says "The standing passages the hub's Bolt checkout holds for the claude engine follow it".
- `Passages` now also returns the shipped layer's `claude-prompt` entries, which come from the binary.
- The doc and `heddleengine/doc.go` ("heddle adds only the passages its layers hold") disagree.
- Fix: reword to "the passages heddle's layers hold".

**Tool calls**
1. `grep` for `"sampler"` / `sampler.yaml`: find files enumerating config modules.
2. Read `fabricengine/replay.go:694-793`: how an unclaimed Bolt path is replayed.
3. `grep` for batching text in docs, stencils and contracts, plus heddle mentions in two pattern files and the fabric template: leftover references to the sentence and pattern coverage.
4. `sed` of `configreg_test.go:1-45` plus a `grep` for `"drift"` in tests: per-module pinned enumerations.
5. `sed` of `configreg_test.go:60-110`, `grep` of `envkit` for sampler, `grep` of `CLAUDE.local.md` in tests outside heddle, `grep` of `configcli` for hub-wide lists: other enumerations and readers of the rendered file.
