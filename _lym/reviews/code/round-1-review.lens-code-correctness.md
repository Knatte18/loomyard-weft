```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I walked each changed branch with a concrete input from the inherited context and made no tool calls.

**Per-branch list**

| Function or branch | Case run | Result |
|---|---|---|
| `CloneHub` two-argument, `Reset`, hub absent (`clone.go:177`, `:240`) | `checkResetTarget` returns `(false, nil)`. The probe runs. `resetHub` re-checks, finds nothing, returns nil. `createExclusiveDir` runs. | Correct. |
| Two-argument, `Reset`, target is a plain directory (`clone.go:178`) | `looksLikeHub` is false, so the offline "not a fabric hub" refusal fires. The network is untouched and nothing is removed. | Correct. |
| Two-argument, `Reset`, hub present, `--shortname` disagrees (`clone.go:194`) | `resolveEffectiveShortname` errors at `clone.go:195`, before the `resetHub` call at `clone.go:241`. The hub is untouched. | Correct. This fixes the reported bug. |
| Two-argument, `Reset`, hub present, warp binding mismatch (`clone.go:190`) | `resolveEffectiveWarpURL` returns `ErrWarpBindingMismatch` before the reset. | Correct. |
| Two-argument, `Reset`, hub present, old-order guard (`clone.go:198`) | Returns before the reset. | Correct. |
| Two-argument, `Reset`, hub present, weft unreachable (`clone.go:186`) | The probe error returns and the hub survives. | Correct, and an improvement. |
| Two-argument, no `Reset`, hub present (`clone.go:181`) | "hub already exists" fires offline, before the probe. | Unchanged and correct. |
| Two-argument, no `Reset`, hub appears during the probe (`clone.go:244`) | The repeated `os.Stat` refuses with "hub already exists". | Correct and harmless. `createExclusiveDir` would also catch it. |
| One-argument, `Reset` (`clone.go:206`–`:246`) | The probe runs, then the name is derived, then `resetHub`. The offline non-hub refusal runs inside `resetHub` before any removal. | Correct and unchanged in effect. |
| One-argument, no `Reset` (`clone.go:244`) | The stat check follows the probe, as before. | Correct. |
| `resetHub` (`clone.go:623`) | If `err != nil || !present` it returns `err`. Absent returns nil, a non-hub returns the refusal, a present hub goes on to `removePath`. | Correct. |
| `resetHub` re-check after a long probe window | A hub that stops looking like a hub, or becomes a file, is refused by the second `checkResetTarget`. `removePath` also applies `ownedFabricHub`. A hub that vanishes is a no-op. | Correct. The wider window is guarded. |
| `checkResetTarget` (`clone.go:647`) | Not-exist returns `(false, nil)`. A stat error, a non-directory and a non-hub each return their own error. A hub returns `(true, nil)`. The messages are unchanged from the old `resetHub`. | Correct. |
| Deferred `res.Mutations = rec.Snapshot()` | Early refusals leave `rec` empty or nil, as before. In the one-argument form `rec` is nil until the name is derived. | Behavior unchanged. |

**Findings**

1. **MEDIUM, `design`, `clone.go:240`–`:246` with `:354`, `:361`, `:386`: a refused `--subpath` under `--reset` still removes the existing hub.**
   - The reset runs after the probe but before the clone and before step 8.
   - At step 8 `adoptRecordedAnchor` can refuse a disagreeing `--subpath` ("requested --subpath … does not match the recorded anchor").
   - The create path can also fail with "subpath … does not exist as a directory in the cloned warp repo".
   - Both go through `teardownHub`, which removes only the fresh hub. The old hub was already deleted at `clone.go:241`.
   - The decision record's Goal says "--reset validates every argument … before removing anything, so a refusal leaves the existing hub untouched". `--subpath` is an argument.
   - `_lym/landing/summary.md` admits this gap as a design decision.
   - The probe already reads `AnchorFound`, so it could carry the recorded anchor value to check the subpath before the reset. If the entry's "(shortname, binding)" parenthetical is the whole scope, the parent should record that. The summary does not record it as a correction.

2. **LOW, `design`, `clone.go:264`, `:298`: the warp and weft clones still run after the reset.**
   - An unreachable warp URL, or a warp whose HEAD names a nonexistent ref (`clone.go:276`), fails after the old hub is gone.
   - `teardownHub` removes only the half-built hub.
   - This is pre-existing, and the probe checks only the weft remote. It is the same kind of gap as finding 1, in the same Goal sentence ("against the remotes").
   - Raise it only if the parent reads the Goal as covering warp reachability.

**Comment accuracy**

- The shared block's comment at `clone.go:236` says "The hub-exists check repeats for the two-argument form, which ran it offline above."
  - With `Reset` that form ran `checkResetTarget` above, not a hub-exists check. It is accurate only for the non-reset path. It is minor and a wording matter.

Tool calls: none; the inherited context already held every definition, caller and test the lens needed.
