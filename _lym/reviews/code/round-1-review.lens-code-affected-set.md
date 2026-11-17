```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set, listing per change what the diff forces outside its own lines.

**Per change: set found, covered or not**

- **`PushPairAnchored(l, opts, PairPush)` signature.** Production callers: `internal/pairteardown/handover.go:50` (passes `Here`), `internal/fabriccli/weft_verbs.go:297` (passes `Here`), `internal/beamcli/deps.go:315` (passes no `Here`). All compile against the new type. Not covered for the beam push (F1 below).
- **`DetachedPush` and `CoalescePushRequest` gain `here`.** The only production callers are `internal/fabriccli/weft_verbs.go:263` and `DetachedPush` itself. Covered. I did not read the `here` argument at line 263.
- **Retired `respawnForeignFlags`, `foreignFlagRequest` and `respawnPairPush`.** A repo-wide grep over `.go` and `.md` finds no leftover caller or doc mention outside `_lym`. Covered.
- **`PushOwedOffline` renamed to `PushOwed`.** No production, spec or stencil text still names the old field. Only the research notes under `docs/research/loom/adopt-build-revision-2026-11-16/` do, and those record a point in time. Covered.
- **`statuscommit.Deps.Spawn` replaced by `Push`, `Admit`, `Owed` and `Sleep`.** No `Spawn:` field is left in the wiring greps for `statuscommit`. Covered.
- **`fabricengine/doc.go` and other prose on the detached push.** Lines 327–347 and 855 still say every fabric commit and sync spawns a detached push. That stays true for the detached pushes that remain (anchor change, fabric commit spawn, `lym fabric sync` and `add`). The pending-flag and teardown-lock respawn prose is gone. Covered.
- **PATTERN declared changes.** `PATTERN.md` carries the `PATTERN-run-fsm` add and the `PATTERN-pair-remote` change, both with the declared text. `pattern/PATTERN-run-fsm.md` is new, `pattern/PATTERN-pair-remote.md` is rewritten to match, and `pattern/PATTERN-daemon-wakeups.md` lists the owed-push backoff. Covered.
- **Commit messages and `Pattern-Departure:` trailers.** The range carries no `Pattern-Departure:` trailer, so none is wrong. Covered.

**Findings**

- **F1 — MEDIUM, scope.** `internal/beamcli/deps.go:315`. `PushBeam` calls `PushPairAnchored` with `PairPush{LockWait: fabricengine.StatusPushLockWait}` and no `Here`. A zero `Here` turns the takeover read off. D2 asks for the read for "any caller of the pair push", and the plan's `names-from-the-caller` decision says only `CoalescePushBothAt`, a fixture's push, passes a zero value. A beam pair's rejected records push therefore replays instead of handing over. Fix: fill `Here` from `pairteardown.ThisMachine()` in `PushBeam`, or run it through `pairteardown.PushRun`.

**Tool calls**

1. Grep for retired symbols, `PushPairAnchored`, `CoalescePushRequest` and `DetachedPush` callers, and the diff of `pattern/` and `PATTERN.md`: the manifest lacked the callers and the pattern diff.
2. Grep for stale detached-push and respawn prose, the `PATTERN.md` entries, and `Pattern-Departure` in the range's commit messages: the manifest lacked these searches.
3. Read `beamcli/deps.go` 300–325, grep `doc.go` and wiring files for stale spawn prose, and grep for `PushOwedOffline` and "unreachable" wording: the manifest lacked the beam caller's `PairPush` and the leftover-wording searches.
