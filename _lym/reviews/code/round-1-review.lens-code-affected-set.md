```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: per change, what each one forces outside its own lines.

**Per-change list**

1. `WorktreeRoot` renamed to `AnchorDir` on `CaptureRequest`, `CompareRequest`, `SpawnRequest` and `stepRun`.
   - Every production construction site is converted:
     - `internal/driftcli/cli.go:240`, `:302`, `:345` and `:416`.
     - `internal/loomcli/driftwiring.go:141`, `:161` and `:203`.
   - The test fixtures are converted. The `CommandStep.Dir` comment in `config.go`, the `doc.go` text and the `docs/overview.md` line are updated. Covered.
   - `Holder.Worktree` now receives the anchor directory. Not covered, see F1.
2. `drift.enabled` is read at every pass (`passWhenSwitchedOff`).
   - `Config` becomes a required seam of `DriftAwait`. The `entries_drift.go` seam check and its test row are updated, and `driftDepsFrom` builds the deps for all three producers, so the production wiring is covered.
   - The `doc.go` files and `docs/overview.md` carry the new text. Covered.
3. `StartCapture` failure wording.
   - `startFailure` no longer names a way forward without `FallbackDetach`.
   - The only non-fallback caller, `captureAgain`, wraps the error into a capture halt that names `lym loom resume` and `lym drift defer`. The CLI path still has the fallback wording. Covered.
4. `RecordScope` is admitted for the parent at a blocked Drift-Baseline or Drift-Await.
   - The `scope` verb help is updated, as are `driftengine/doc.go` and the `RecordScope` doc comment. The tests cover the new rows. Covered.
   - Wording that still says "once" is left alone, and each instance is still accurate for the baseline agent:
     - the stencil step 3 text,
     - the scope note at `baseline.go:85`.
5. The baseline stencil gains `{{.capture_command}}`.
   - `BaselineTable` fills it, and `markers_test.go` and `seats_test.go` are updated.
   - The only template and registration points are `stencils.go:51-54`, `:569` and `:699`, and none needs a change. Covered.
   - Stencil wording: the "refuses a second call, call it once" rule is kept, and no existing rule is dropped.
6. `drift defer` at a capture halt.
   - `Defer` itself is not in the diff and already admitted a capture halt. The diff only makes the unstarted-capture halt carry `CaptureHaltPrefix`.
   - The test asserts that `Defer` admits the halt. Covered.
7. PATTERN entries and commit message.
   - The directive declares no PATTERN changes, and none are expected.
   - `PATTERN-cwd-resolution` is followed: engines receive an absolute path from `location.AnchorPath()`.
   - The commit message claims match the diff. No `Pattern-Departure` trailer is present or needed.

**Findings**

- **F1, LOW, consistency.** Location: `internal/driftengine/steprun.go:149`.
  - **Summary:** `acquireSlot` now writes `Holder{Worktree: run.AnchorDir}`, but `Holder.Worktree` is documented as the worktree and every other holder is written with the worktree root (`gatecli/run.go:584` uses `location.WorktreePath()`).
  - **Evidence:** `internal/gateslot/pool.go:68-72`. Readers print it as the worktree: `gatecli/status.go:71`, `detachfront.go:920`, `loomcli/reviewstatus.go:47` and `fabriccli/anchordeps.go:220`.
  - **Effect:** in an anchored hub, a drift holder shows a subdirectory path where every other holder shows the worktree root. The diff changed this value without mentioning it.
  - **Fix:** pass the worktree root for the holder identity, for example a separate `WorktreeRoot` field kept on `stepRun` and the two request types. Otherwise, document in the `Holder` comment that this is a path inside the worktree.
  - **Scope note:** the readers I found only display or log it, so I found no functional break.
- **F2, NIT, consistency.** Location: `internal/driftcli/wiring.go:114`.
  - **Summary:** `gateslot.WaitDir` is already given `AnchorPath()` here. This is unchanged, but it shows the anchor-versus-root split is applied inconsistently.
  - **Why it matters:** `CaptureJobName(w.root())` keeps the root, and the Holder now takes the anchor.
  - **Fix:** none required. Fold it into F1's choice.

**Tool calls (3)**
1. Grep for readers of `Holder.Worktree` and for production construction sites of the changed request types. My context lacked the other consumers.
2. Grep for docs, stencils and the stencil registry mentioning drift scope or the baseline template. My context lacked the registry and other references.
3. Grep for remaining "once" and "refused" wording about drift scope or defer. I also checked the `Config` wiring and the `root()` and `AnchorPath` uses in `driftcli`, which my context lacked.
