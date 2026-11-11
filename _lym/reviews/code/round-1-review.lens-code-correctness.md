```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the per-orch Slack button diff.

**Per-function list (case run, verdict)**

- `forwarding.go` `WriteForwarding`, with a nil `Orchs` map and `on=false` for an unknown orch: `current.Orchs[orch].On` reads false, so it returns unchanged and nothing is written. Correct.
- `ReadForwarding` and `ForwardingOn`, with a missing file and with a corrupt file: missing gives none, corrupt gives an error naming the path. Correct.
- `dropForwarding`, with a nil map: `delete` on a nil map is safe. It still rewrites the file every time, so it wakes the watcher (see F2).
- `ChannelName`, with empty parts and a name over 80 characters: empty parts are left out and long names are cut with a hash. Correct. An unreadable board returns an error, and a beam with no project entry gets no handle.
- `keeper.ensure`, with no entry and the button off: returns `errForwardingOff` and makes no channel. An open entry is returned whatever the button. Correct.
- `keeper.create`, with `Project` failing: wrapped error, no channel made. Correct.
- `retire` / `retireEntry`, using the new `Renamed` flag: a legacy entry that is already under a dated name but has no `Renamed` flag is renamed again. The date suffix is tried until Slack accepts, so this ends in a double-dated name rather than failing. Old retire state is not migrated, which is low severity and unlikely. Not filed separately.
- `retire`, button drop when `dropForwarding` fails after the archive stands: wrong, see F1.
- `retire`, drop for an orch with a button but no channel entry: wrong, see F2.
- `ensureForwardedChannels`, with `beams=nil` after a failed beams read: beam orchs are skipped and retried at the next wake. The prime still gets its channel. Correct and intended.
- `ensureForwardedChannels`, with `slugOf` failing: the error is joined and the orch is skipped. Correct.
- `relayStart` and `wake`, with `ReadForwarding` failing: logged per episode, nothing crashes. Correct.
- `wake`, `diskCheck`, `watch`: before the first wake `forwardingKnown` is false, so no false positive. The forwarding file is watched by name in its own directory. Correct.
- `inbound.handle`, button off: the message is answered, settled and the cursor moves, with no forward and no presence change. `away`, `back` and `status` still work because they run before the button check. Correct.
- `inbound.handle`, button read failing: goes to `retryOrGiveUp`. Correct.
- `outbound.postMessages`, button off: acked unposted. `ReadForwarding` failing returns an error before any message, so the whole batch waits. Acceptable.
- `outbound.postTo`, `errForwardingOff` on a redirect to the prime: dropped and acked. Correct.
- `outbound.reportStale`, prime button off with an open prime channel: the relay's own stale line still posts. This is fine, since it is the relay's line rather than an orch's mail.
- `start.ensure`: `ReadForwarding` empty gives `OutcomeForwardingOff`; corrupt gives an error. Correct.
- `slackcli.away`, outcome `ForwardingOff`: sets the presence without waiting for a relay. Correct.
- `toggleForwarding`, no orch: refusal with a way forward. Correct.
- `forwardingOn`, `ensure` failing: the button stays on and the error names `lym slack start`. Correct.
- `forwardingOff`, last button: `stop` ends the relay (the `c.stop` seam is `slackengine.StopRelay`, which does not touch the presence). The stop is skipped when another orch's button is still on. Correct.
- `barLine` and `barSlackWord`, with an empty dir, an unresolvable dir, a wire error and an orch lookup error: each prints only the away word. A failed read of the button prints unlit. Correct.
- `worktreeOrch`: the prime gives the hub's orch, a listed beam gives its orch, anything else gives "". Correct.
- `replyHook`, pair strand: `TaskSlug` gives the pair's slug, so the button reads off and nothing is posted, as intended. `ForwardingOn` failing returns a warning and an error. Correct.
- `composeSlackStateCommand`: `#` in the binary path is doubled for the extra format expansion. The `@lym_button_dir` option is already shell-quoted (`buttonrow.go:20,176`). Correct. I could not check the `##` handling against tmux itself.
- `orchProject`, empty slug: root handle and subject. Beam without an entry: empty handle and the slug. Correct.

**Findings**

- **F1 — MEDIUM, design.** `internal/slackengine/keeper.go:426-436` with `:413`. If `retireEntry` archives the channel but `dropForwarding` then fails, the orch is marked refused. Every later retire sees `entry.Archived` and skips it (`:413`), so the forwarding entry is never dropped. The beam's button then stays on. Because the relay is ended only on a button press, the relay keeps running. A later beam with the same slug starts with its button already on and gets a channel with no press. The drop is only retried while the entry is open.
  - Fix: drop the forwarding entry for every archived or non-listed beam orch on each retire pass, independent of the archive step.

- **F2 — MEDIUM, design.** `keeper.go:392,426-429`, `relay.go:377-396`, `toggle.go:208-212`. Forwarding entries are dropped only for orchs that have a channel entry. A beam whose button is on but which never got a channel (retired before the relay made one, or `ensureForwardedChannels` skipped it as unlisted) keeps its entry forever. `ReadForwarding` is then non-empty, so `start.ensure` keeps starting the relay and `forwardingOff` never ends it. This breaks "the relay runs only while at least one orch is active".
  - Fix: prune forwarding entries of beam orchs the hub no longer lists, whether or not a channel entry exists, in the same wake that retires.

- **F3 — LOW, design.** `relay.go:597-674`. When retire's `dropForwarding` removes the last button (a beam-only hub whose beam retires), nothing ends the relay. Only the toggle verb calls `StopRelay`, so the relay keeps its Socket Mode connection with no active orch until the next press or stop.
  - Fix: have the wake loop return, and the relay exit cleanly, when `ReadForwarding` is empty after a wake. The start paths already refuse to start without a button, so there is no restart loop.

**Tool calls**
1. `grep` for `buttonDirOption`, `TaskSlug` and the `c.stop` seam: to confirm the dir option is quoted, that `TaskSlug` gives the slug for a pair's orch strand, and that `c.stop` is `StopRelay` and not the presence-setting verb. My context lacked those definitions.
