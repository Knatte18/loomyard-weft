# creel-agents — every agent messages through creel

All four batches reached `done`, with no departures or renames reported, no warnings, and no recovery or re-fork.

## 01-orchroute-walk (head 3cdf05b5)

Cards 1–4.
The orch walk, liveness and strand lookups moved into the new `internal/orchroute`, with the asker rule.
`lym creel post --parent` was added.
The hold notice now names `lym creel post --reply` and `lym creel post`, and the burler, loom and webster hold-notice notifiers resolve their orch with the asker rule.
Deviations: `internal/creelcli/cli.go`.

## 05-expect-reply-engine (head 6fd1129f)

Cards 5–7.
A post can carry an expected-reply mark that the holder's reply clears.
`post --expect-reply` and the mark and reply-to in read and log output were added.
A chase clock chases a read, marked, unanswered message twice, then records it unanswered.
Deviations: `internal/creelcli/cli_test.go`, `internal/creelengine/deliver.go`, `deliver_test.go`, `fold.go`, `verbs.go`, `verbs_test.go`.

## 08-chase-reply-forward (head 0009874a)

Cards 8–12.
A reply to a chase answers the chased message, and a chase is never forwarded.
A holder gone after reading a marked message settles it unanswered with reason `holder-gone`.
`post --wait-reply` exits 1 on unanswered, and `log` shows the chase number.
`lym report replies` was added.
Every block 7 stencil site and the seat smoke fixtures now message through creel.
Deviations: `internal/creelengine/deliver_test.go`, `internal/reportcli/replies.go`, `internal/reportengine/replies.go`.

## 13-drop-peer-messaging (head f03b8c2b)

Cards 13–16.
`RequirementPeerMessaging` was removed from the closed set and the seat specs.
The creel trust sentence defers other mail to the role's rules.
`lym shuttle send` and `lym shuttle interrupt` were removed and their refusals re-pointed at creel.
A tier0 scan keeps `SendMessage` out, and PATTERN gained `PATTERN-agent-messaging`.
Deviations: `internal/creelcli/cli.go`, `internal/seatengine/engine_test.go`, `internal/shuttlecli/cli.go`.

## Scope guard

Every deviation above is informational and was reported by the guard; none made a batch stuck.
