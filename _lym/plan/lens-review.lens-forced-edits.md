```yaml
lens: forced-edits
outcome: reported
```

Lens: forced-edits. These are the changes the cards force outside the members they name, plus missing listings.

1. **MEDIUM — D1/D2/D3, `docs/overview.md`: no card updates the creel paragraph, and these changes make it wrong.**
   - `docs/overview.md:479` says the mail arrives "in a `PostToolUse` block and at its turn end in a `Stop` block that records no turn end while it blocks, at most 6 blocks a turn". Cards 1 and 2 remove both blocks.
   - `docs/overview.md:480` says "the woken turn's hooks hand it the mail". Under cards 4 and 6 the ring hands it over.
   - No card lists `docs/overview.md`. Card 1 or card 6 should list it under `**Edit:**`.

2. **MEDIUM — D1, `internal/creelengine/doc.go` Stop paragraph: no card's Intent covers it.**
   - `internal/creelengine/doc.go:127` says the Stop hook "blocks the turn end while it hands out mail".
   - `internal/creelengine/doc.go:203` says "the Stop block delivers it".
   - `internal/creelengine/doc.go:133` says a message is delivered once by "the claim during a turn, or the pane path". It omits the ring's claim from card 4.
   - Cards 3, 4, 5, 7 and 9 list this `doc.go`, but each Intent names only its own paragraph. Card 1, which makes D1 true, does not list it.
   - The scope bullet "the package docs and code comments that state these renderings" requires the change.

3. **LOW — D1, creelengine comments that still describe a Stop "block": no card lists them.**
   - `internal/creelengine/verbs.go:294` (`Boundary.StopHookActive`, "an earlier Stop block").
   - `internal/creelengine/verbs.go:298` (`stopBlockCap`, "consecutive Stop blocks … before it overrides a block").
   - `internal/creelengine/verbs.go:305` (`Store.ClaimAtBoundary` doc). Cards 4 and 7 list this member, but their Intents do not reword it.
   - `internal/creelengine/pool.go:51` (the `stopBlocks` field comment on `internal/creelengine#Store`).
   - `internal/shuttleengine/engine.go:232` (`HookInput.StopHookActive`, "an earlier Stop block"). Card 1 lists only `internal/shuttleengine#ToolBoundaryContextRenderer`; `internal/shuttleengine#HookInput` is missed.

4. **LOW — D3, the `lym creel` group help is wrong after the ring hands over the mail.**
   - `internal/creelcli/cli.go:77-78` (`internal/creelcli#Command` Long): an idle strand "is woken first by a waiter that runs the hidden hook-wait verb", then gets the message "at its next tool call or turn end".
   - Card 6 lists `hookWaitCmd` and `internal/creelcli/doc.go`, but not `internal/creelcli#Command`.

5. **LOW — D3, comments that card 4 and card 5 make wrong.**
   - `internal/creelengine/pool.go:454-455`: the `wakeMark` doc says "A ring is answered by a claim that handed out mail, or by a probe". Card 4 makes the ring's own claim answer it at the acknowledgement, and no card lists `internal/creelengine#wakeMark`.
   - `internal/creelengine/protocol.go:96`: the `Response.Messages` comment says "the messages a read hands out". Card 4 writes ring responses with Messages and AckRequired, and card 7's Intent on `Response` only adds `Echo`.
   - `internal/creelengine/protocol.go:33`: the `VerbHookWait` comment is unlisted.
   - `internal/creelengine/client.go:278` (`Client.Call` doc: "resumes the same wait") and `internal/creelengine/client.go:322` (`Client.resume` doc: "a hook-wait resume[s]"). Card 5 lists only `CallWatching`, `exchange` and `waitState`; `internal/creelengine#Client.Call` and `#Client.resume` are missed.

6. **LOW — Card 1, member glyphs under `Edit`.**
   - Card 1 lists the file `internal/shuttleengine/claudeengine/capabilities.go` under `**Edit:**`, but its Intent removes the named members `Claude.BoundaryBlock` and `stopBlockReason` (`capabilities.go:66,72`).
   - They should be a `**Delete:**` group of `internal/shuttleengine/claudeengine#Claude.BoundaryBlock` and `internal/shuttleengine/claudeengine#stopBlockReason`. The file stays only for its header comment (`capabilities.go:2`).

7. **LOW — Card 1, test names that still say "block".**
   - `internal/creelcli/cli_test.go:848`: a `TestCreelCLI` subtest is named "…as the engine's visible block…". Card 1 lists only `hookBlock`, not `internal/creelcli#TestCreelCLI`.
   - `internal/creelcli/hookread_test.go:27-28`: `TestHookReadExit` rows are named `StopWroteTheBlock…`. No card lists `internal/creelcli#TestHookReadExit`.
   - The assertions still pass; only the names are stale.

Tool calls:
- Grep for `HookWait(` callers, the `cli.go` help, the `shuttleengines.go` doc, PATTERN-daemon-wakeups, and "block"/`asyncRewake` wording across `internal`, `docs` and `contracts`. My context had no repo-wide search for the callers or for stale wording.
- Read `docs/overview.md:476-484`, searched tests for the waiter command, `testtiming:keep` reasons, other implementers of the seam, and the hook-wait comments in `protocol.go` and `client.go`. My context had not read these files or lines.
