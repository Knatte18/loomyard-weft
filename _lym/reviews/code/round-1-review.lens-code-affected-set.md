```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: what the diff's signature, variable and option changes force outside the lines it touches.

**Per change, the set found:**

1. **Removed `originCallHook`, `originVerdictHook`, `callOriginHook` and `callOriginVerdictHook`, and the two `SetOrigin*HookForTest` helpers.**
   - The repo-wide search finds no remaining reference outside the touched files and old `docs/research` logs.
   - The three origin reconcile tests are moved to `ReconcileOptions.CallHook` and `VerdictHook`.
   - The production `ReconcileOptions{}` callers (`fabriccli`, `loomcli`, `adoptcli`, `orchcli`, `battencli`, `cleanupcli`) leave the hooks nil.
   - `hibernatecli` sets only the retry fields, so the zero-value hooks are production.
   - Covered.
2. **Removed `anchorFailHook`, `weftHeadSHA` and `runAnchorFailHook`; `moveWeftContent` gains a `seams` parameter.**
   - Both callers are updated: `addanchor.go:92` passes `anchorSeams{}` and `anchorchange.go:452` passes `c.seams`.
   - The only `Topology{}` literal in the repo is in `NewTopology`, so no zero-value `Topology` is built elsewhere.
   - `killAt` and `SetAnchorFailHookForTest` are used only in `anchorchange_integration_test.go`.
   - Covered.
3. **`gcAutoThreshold` becomes a `Topology` field, and `housekeepStores` takes a threshold.**
   - The only caller is `remove.go:420`.
   - The constructor sets the default, and every `SetGCAutoThresholdForTest` call is moved to the new signature.
   - Covered.
4. **New `SyncOptions.Notifier`, `recordPushFailure` and `installedNotifier`.**
   - The only in-package consumer of the notifier is `pushclassified.go:283`, which is updated.
   - `remoteteardown.go:245` still reads the installed notifier, which is intended and stays serial.
   - The only places that build a `SyncOptions` from other options are `add.go:75` and `fabric.go:92`, and neither carries a notifier.
   - Production callers pass a nil `Notifier`, so they fall back to the installed one.
   - `RecordPushFailure` keeps its signature, so the external callers in `fabriccli`, `orchcli`, `tendercli` and the `fabricengine` tests are untouched.
   - `installFakePushNotifier` has no remaining reference.
   - `doc.go` is updated for the notifier.
   - Covered.
5. **Serial tests that install the notifier (`recordNotices`, `remoteteardown_integration_test.go:79`).**
   - They stay serial. Their comments still say so (`remoteteardown_test.go:21` and `:36`).
   - Covered.
6. **Stale comments and docs naming the removed variables.**
   - No reference to a removed identifier remains in `doc.go`, `pattern/` or the source comments.
   - `pattern/PATTERN-test-speed.md:181` states a general rule that the change does not contradict.
   - Covered.
7. **Rubric items.**
   - No `Pattern-Departure` trailer appears in the two commit messages, and the diff touches no `PATTERN.md` or `pattern/` file.
   - The focus directive and the rubric declare no PATTERN changes, so there is nothing to land.
   - No stencil is changed.
   - Nothing outside `internal/fabricengine` is edited.
   - Covered.

**Findings:** none.

**Tool calls:**
1. `grep` across the repo for every removed or renamed identifier, to find callers and docs outside the diff.
2. `grep` for `SyncOptions{` and `ReconcileOptions{` in non-test code and for "process-global" comments, to find option constructors that might drop the new fields and stale comments.
3. `sed` and `grep` on `remoteteardown_integration_test.go`, to confirm the installed-notifier test stays serial.
