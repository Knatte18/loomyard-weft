```yaml
lens: code-affected-set
outcome: reported
```

**Lens code-affected-set: per change, the set found and whether the diff covers it.** No finding survives, so there are no findings to list.

1. **Seam method `BoundaryBlock` → `HookOutput(event, mail, echo)`:** covered.
   - The interface (`shuttleengine/engine.go:244`) and the implementation (`claudeengine/capabilities.go:70`) both carry the new signature.
   - The callers all use the new name: `creelcli/hookread.go:130`, `cli_test.go:167`, `claudeengine/settings_test.go:208`, `shuttleengines_test.go:344-345`.
   - Repo-wide grep finds no `BoundaryBlock` or `stopBlockReason` left.
   - The `shuttleengine`, `shuttleengines` and `claudeengine` docs say "hook output".

2. **`Store.HookWait(sender, takesMail)`:** covered.
   - The production caller `server.go:307` passes `req.TakesMail`.
   - The test callers in `deliver_test.go` (1703, 1712, 2242, 3033) and `flags_test.go:144` pass the new argument.

3. **`ringHook(strand Strand, …)`:** covered. Its single caller `deliver.go:280` passes the `Strand`.

4. **Ring now carries mail (`OutcomeRing`, `Outcome.Reading`, `Request.TakesMail`, `Response.Echo`):** covered.
   - `pool.go`, `protocol.go`, `server.go`, `client.go` and `creelengine/doc.go` describe it.
   - `docs/overview.md:480-481` and the `creelcli/cli.go` `Long` texts are updated.
   - `creelcli/doc.go` states the new `hook-wait` exits and `claudeengine/doc.go` the new Stop and PostToolUse shapes.
   - `shuttleengine/claudeengine/doc.go` and `settings.go` state the `rewakeMessage` and `rewakeSummary` labels.

5. **Stencil wording:** covered.
   - `orch-template-role.md:39-40` is the only stencil that describes hook delivery or the ring's bell wake; a grep of `contracts/stencils` and `contracts/specs` for wake, bell and delivery wording finds no other.
   - It keeps the "already read, answer with `lym creel post --reply`" rule, adds the idle-wake delivery path, and keeps the unreached-orch fixed line.
   - `darn-template-write.md:43` ("a note alone wakes nothing") is unrelated.
   - The creel trust sentence (`settings.go:435`) is unchanged and still matches the `[creel]` prefix of the wake label.
   - No test asserts the orch stencil's prose, so none breaks.

6. **`cli.go` root `Long`:** covered.
   - `cmd/lym/clitree_test.go:167` caps `Long` at 2000 bytes, which is what the `829dfcf9` fix commit addressed.
   - I did not measure the root `Long`'s byte count; if the gate is green it holds.

7. **`stopBlockCap` and `stopBlocks` naming and comments:** covered.
   - The wording is updated in `verbs.go`, `doc.go` and `claudeengine/doc.go`.
   - `pool.go:51` still reads "Stop blocks of its current turn", which is a LOW-or-NIT wording leftover (comment only; the identifier `stopBlocks` is unchanged).
   - `verbs_test.go:1261` still says "Stop block %d", which is test message wording only.

8. **`flaggedFilesLocked` includes owed echoes:** covered by the `flags.go` change, `flags_test.go` and `creelengine/doc.go`.

9. **PATTERN declared changes and departures:** none to land.
   - The constraints section declares no changes.
   - The diff touches neither `PATTERN.md` nor `pattern/`.
   - No commit carries a `Pattern-Departure:` trailer.
   - The commit messages are bare card titles plus the attribution line.

10. **Repo rules touched:**
    - `PATTERN-shuttle-provider-seam`: the Claude JSON keys and rewake text stay in `claudeengine`, and `creelcli` passes `mail` and `echo` strings through the seam.
    - `PATTERN-tolerant-records`: `TakesMail` is an `omitempty` request field with no `DisallowUnknownFields` involved.
    - `PATTERN-spawn-observability`: the log levels follow the policy, with `Debug` for the empty claim.

**Tool calls:** 2 Bash greps. They covered callers of `HookWait`, `ringHook`, `HookOutput` and `RewakeOutput`, stale wording across contracts, docs and code, the `Long` byte cap, and `Pattern-Departure` trailers. My context lacked all of these.
