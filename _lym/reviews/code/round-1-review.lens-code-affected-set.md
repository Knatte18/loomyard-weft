```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. For each change the diff makes, the set of things it forces elsewhere, and whether the diff covers them.**

| Change | Set found | Covered? |
|---|---|---|
| `Geometry`: `CursorPath` and `CursorFileName` removed; `Machine`, `ChannelMapPath`, `InboxDir`, `SocketLockPath` and `LiveHubs` added | `hubgeom/slackgeom.go` fills them; `hubgeom` tests and doc; `beamcli` integration test. A repo-wide search finds no leftover `CursorPath`, `cursor.json`, `ReadToken` or `WriteToken` outside tests | Covered |
| Credential file now holds two tokens (`ReadCredentials`, `Credentials.Missing`) | `start.go` `ensure` (`len(Missing()) > 0` → `OutcomeNotConfigured`), `RequireConfigured`, `runRelay` in `wiring.go`, the `slackcli` `login` and `setup` verbs, `engine/doc.go`, `slackcli/doc.go` | Covered |
| `LastSlackOK` now means last-healthy time | `start.go` `lifetime` comment, `slackcli` status (`last_healthy`), `relaystate.go` | Covered. The JSON key `last_slack_ok` stays, as D1 says |
| Outbound posts go to the sender's channel, with no `[sender]` prefix and no answer hint | `contracts/orchrole/role-slack.md`, `docs/overview.md`, `engine/doc.go`. A search for "answer hint" and "operator's channel" finds no stale prose elsewhere | Covered |
| `setup --channel` removed; `setup --operator` added | A search for `slack setup --channel` finds nothing. `RequireConfigured`, `keeper.go`, `inbound.go` and `outbound.go` name `--operator <user-id>` | Covered |
| Scope lists, manifest and `channels:join` | `slackcli/cli.go` `botScopes` and the `login` help, `docs/slack-app.md` (bot scopes with `channels:join`, no user scope), `docs/overview.md`. No `xoxp`, `unarchive` or `channels:write` string is left in production code or docs | Covered |
| New wait loops (holder lock wait, ping, reconnect, inbox watch and on-disk fallback) | `holder.go` is added to `unboundedWaitFiles` in `cmd/lym/daemonwakeups_test.go`. `pattern/PATTERN-daemon-wakeups.md` lists the Slack loops with the holder and the inbox watch | Covered. The list does not name the wake loop's `time.After` retry timers separately, but they sit in `relay.go`, which is already listed |
| Declared change to `PATTERN-agent-addressing` | The `PATTERN.md` line (line 43) already equals the declared text, and the record says the overview line keeps its text. The Slack boundary rule in the background file names the bot and app-level tokens and drops the user-token sentence, as the operator's amendment requires | Landed. No finding |
| `Pattern-Departure:` on `fdc141fb` (`PATTERN-lym-files`) | See finding 1 | Finding |
| `docs/overview.md` slack bullet and the package docs | The overview bullet describes the push route, the channel per orch, the two tokens and `docs/slack-app.md`. `slackengine/doc.go` and `slackcli/doc.go` agree with it | Covered |

## Findings

1. **LOW, `consistency`: the `Pattern-Departure:` trailer on `fdc141fb` names an entry that does not forbid the way taken.**
   - Location: `internal/slackengine/holder.go:557` (`os.CreateTemp(geom.Dir, hintTempPattern)`), the trailer on commit `fdc141fb`, and `pattern/PATTERN-lym-files.md`.
   - The trailer names `PATTERN-lym-files` and says the hint's temp file sits in the hub's Slack directory, outside `lymtemp`.
   - The entry's temp rule (line 46) forbids `os.TempDir()` and `""` as the temp directory, and the guard `cmd/lym/baretemp_test.go` fails only those.
   - A temp file created in the hub's own `.lym/slack/` directory is not forbidden, and the entry's own wording (line 37) says to put the temp directory beside the target so a rename never crosses a filesystem.
   - So the trailer excuses a difference the entry does not forbid. The rubric makes a trailer that names an entry that does not forbid that way a finding. The fix is to drop the trailer, since no departure exists.
   - I did not check whether the `lymdirs` rule requires this directory name to come from `lymdirs`. The diff takes the directory from `geom.Dir`, which `hubgeom` builds from `lymdirs.SlackDirName`.

I found no other stale reference in the affected set.

## Tool calls (4 of 6)
1. Grep for stale `--channel`, `last_slack_ok`, `@<slug>`, `xoxp`, `unarchive` and similar strings outside `_lym`, `.lym` and tests. My context had no repo-wide search for these.
2. `git diff` of `docs/overview.md`, the two `doc.go` files, `hubgeom/slackgeom.go`, `geometry.go`, `template.yaml` and `start.go`. The earlier output was truncated or unread.
3. Grep of `pattern/PATTERN-lym-files.md` for its temp rules, and a grep of the other `pattern/` files for Slack mentions. I needed the entry text to judge the departure trailer and to find other entries the diff touches.
4. Grep for leftover "answer hint", "cursor.json", `CursorPath`, `ReadToken` and `WriteToken`. I needed to confirm that nothing stale remains in contracts or stencils.
