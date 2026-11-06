```yaml
lens: forced-edits
outcome: reported
```

forced-edits lens over the plan: 8 findings. The glyphs that exist on disk resolve in quarry (a sample of the plan's, all `found`), and each `Create` handle's package matches its file's package clause.

**1. BLOCKING — Card 1, `setTempEnv` / `TestMain_PointsTheStateHomeAtTheRunDirectory`: the existing `TestSetEnv` is not listed, and it will leak the new variable.**
- Evidence: `internal/testkit/tmuxkit/tmuxkit_test.go:59` saves only `TMPDIR`, `TMP`, `TEMP` and `GIT_CEILING_DIRECTORIES` through `t.Setenv` before `setEnv(dir)`. `tmuxkit.go:545` calls `setTempEnv`.
- Once `setTempEnv` writes `XDG_STATE_HOME`, `TestSetEnv` leaves it pointing at its own removed `t.TempDir()` child for the rest of the binary.
- The card's new test asserts the run directory, so it fails whenever it runs after `TestSetEnv`.
- The new test also repeats a behaviour `TestSetEnv:69` already checks for the sibling variables.
- Fix: list `internal/testkit/tmuxkit#TestSetEnv` under `Edit`, add `XDG_STATE_HOME` to the lists at lines 59 and 69, and drop the new test.

**2. BLOCKING — Cards 4, 5, 6, 7, 8: symbols created by earlier cards are spelled as real glyphs, not as their `plan:` handles.**
- Refs in question:
  - Card 8 `Edit`: `internal/creelengine#Store.postOtherHub` (created by card 6).
  - Card 4 `Uses`: `internal/hubregistry#Record`, `internal/hubregistry#Lookup`, `internal/creelconfig#LoadConfig`, `internal/standalonestate#UserStateDir`.
  - Card 5 `Uses`: `internal/hubregistry#Live`, `internal/creelconfig#LoadConfig`.
  - Cards 6, 7, 8, 9 `Uses`: `internal/testkit/creelfake#Hubs`.
  - Cards 6 and 8 `Uses`: `internal/creelengine#HubDirectory`.
  - Card 8 `Uses`: `internal/creelengine_test#newHubPair`.
- None of these exists in the tree, and the prompt's rule is that a later card uses "the identical handle string". As written they raise `glyph-not-found`, and the `Uses` edges to the creating cards are not derived.
- Fix: respell each as the `plan:` handle its `Create` bullet declares.

**3. MEDIUM — Cards 3, 6, 9: `lym creel report` input, `internal/reportengine#ComputeReplies`, is left inconsistent, and no card lists it.**
- Evidence: `internal/reportengine/replies.go:31-58` counts `Expected` and `Replied` only from `Event == "posted"`, but it counts `chased` and `unanswered` for every message.
- Cross-hub mail journals `inbound` and `outbound`, never `posted`. A marked inbound message that gets chased therefore adds `Chased` and `Unanswered` with no `Expected`, and a reply to it adds no `Replied`.
- `lym report replies` per-role figures then come out wrong once this mail flows.
- Fix: have card 3 or card 9 list `ComputeReplies` and its test, or have `logEntryFor` carry the inbound or outbound kind so the report can count it.

**4. MEDIUM — Card 9, `TestCreelCLI` end-to-end row (and card 5's second hub): `internal/creelcli#rig.startDaemon` is not listed.**
- Evidence: `internal/creelcli/cli_test.go:311-314` builds `creelengine.Options` with no `Hubs`, so a daemon started that way has a nil seam and a post to another hub stays an unknown receiver.
- The row's "real `hubDirectory`" and two daemons need `rig.startDaemon` changed, or a new two-hub helper.
- `newRig` also swaps the global `seam` on each call (`cli_test.go:298-300`), which a two-rig row has to handle.
- Fix: list `rig.startDaemon` and `newRig`, or a `Create` helper, on card 9.

**5. LOW — Card 3, `pool.logEntryFor`: the Intent omits branches the change forces.**
- `fold.go:483-491` fills `Kind`, `ExpectReply` and `ReplyTo` only for `eventPosted` and `Of == eventPosted`, and `fold.go:463-475` resolves `From` and `Subject` only from `messages` and `injects`.
- `internal/creelcli/doc.go:16` promises `expect_reply` and `reply_to` on such log entries. Inbound and `Of: eventInbound` entries lose those fields.
- `eventRemoteDelivered` and `eventOutboundReturned` entries lose their parties too, yet D2's notice sends the sender to `lym creel log --id <id>`.
- Fix: state these branches in the Intent and add rows to `TestJournal_Fold`.

**6. LOW — Cards 3, 4, 6, 7, 8: `internal/creelengine/doc.go` sits under `Extends`, but these cards make existing text wrong.**
- `doc.go:54`: "any other name is refused", which card 6 falsifies.
- `doc.go:80-81` and `:94`: the line and reminder forms, which card 3 falsifies.
- `doc.go:31` and `:44`: sender resolution for every request, which card 7 falsifies.
- Rubric: "`Extends` on a ref the card rewrites". Fix: move it under `Edit` on those cards.

**7. LOW — Card 1: stale description and incomplete hermeticity.**
- `pattern/PATTERN-test-isolation.md:38` lists what `tmuxkit.Main` points at the run directory. No card extends it for `XDG_STATE_HOME`.
- `setTempEnv`'s own comment (`tmuxkit.go:531-532`) needs the same update.
- `setTempEnv` has a Windows branch (`tmuxkit.go:536-539`) and `UserStateDir` reads `LOCALAPPDATA` there. The repo's own redirect helpers set both variables (`cliwire_test.go:92-97`), but the card sets `XDG_STATE_HOME` only.

**8. LOW — Cards 5, 6, 9, 10: help and role text no card updates.**
- `internal/creelcli/cli.go:74-87` (group `Long`, listing the verbs) does not gain `hubs`, though card 5 edits `Command`.
- `postCmd`'s `Long` (`cli.go:341-375`) names no other-hub receiver or the hour's return.
- `forwardCmd`'s `Long` (`cli.go:560-566`) omits the new refusal.
- `contracts/orchrole/orch-role.md:57` gives the line as `[creel] #<id> from "<sender>": <text>`, with no stamped form. Card 10 only adds to the section near line 309.

Tool calls:
- `lym quarry resolve` plus the tmuxkit and creelfake listing: glyph and package truth was not in context.
- Grep across the XDG, `clientSeam`, `AudienceRole`, `creelseam`, `replies` and board-label sites: those consumers had not been read.
- Second `lym quarry resolve` plus greps for scans, HOME reliance, other-hub names, trust prose and event-name consumers: the first resolve's filter gave no statuses, and these sites were unread.
- `TestSetEnv`, `Main`'s temp env call and `alpha` in `cli_test.go`: needed to confirm finding 1 and that CLI tests use the hub's own shortname.
