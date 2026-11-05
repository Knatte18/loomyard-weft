```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code paths that already run against the state each decision adds or changes.

**Paths traced, per decision**

- **D1 `hubs.json`:**
  - The daemon start path (`lym creel daemon` through `spawnDaemon`) survives: the record is written after the lock.
  - `hubgeom.CreelGeometry` is the only geometry builder (`cli.go:219`, `cli.go:957`, `goclient.go:70`). It reads the same `ReadShortname` that D1 checks against, so it survives.
- **D2 outbound event:**
  - The fold ignores an unknown event (doc.go:64), so the older-binary path survives.
  - The flag reconcile, delivery worker, reminder and sweep only act on messages a local strand holds. D2 keeps outbound messages out of all of them, so they survive.
  - Retention carry (doc.go:68) carries only "unread or chaseable" messages, and D2 extends it explicitly. Survives.
  - Binary-replace shutdown (doc.go:38) loses no message: a `deliver` in flight is retried through D3's confirm-until-journaled rule. Survives.
- **D2/D3 `deliver` and `confirm` reaching the daemon:**
  - `server.go:208` runs `d.resolve` on every request before `dispatch`. That path breaks for these two verbs; see finding 2.
- **D3 replies:** a client checks that the response's hub path matches the hub it called (doc.go:32). The outbound worker builds its client from the other hub's `CreelGeometry`, so this survives.
- **D4 paths:**
  - `Reply`, `Forward`, the chase `CREEL` notice (sent only to a listed strand or a program, doc.go:294) and the sweep step back to "no strand" (doc.go:270) all match D4's stated behaviour. Survive.
- **D5 config:**
  - `LoadOrTemplate` degrades to the template when the base file is absent, giving an empty `trusted_hubs`. Survives.
  - `lym config reconcile` pruning unknown keys does not touch a declared key. Survives.
- **D6 stamp:**
  - The hook line and notice are `MessageLine` (`reminder.go:38-40`), which D6 changes. Survives.
  - The reminder line is covered by D6. Survives.
  - The text output of `lym creel read` (`cli.go:600`) breaks; see finding 1.
- **D7:** the `[creel]` trust sentence (`settings.go:473`) and the parent directive (`parent-directive-parent.md:15`) both read the stamp. Through `lym creel read` they break; see finding 1.
- **D8:** off Linux, `resolveFromEnvironment` (`identity.go:104`) turns a strand name of another hub into EXTERNAL. That is consistent with D8. Survives.

**Findings**

1. **MEDIUM, design. D6/D7: `lym creel read`'s text output shows no stamp.**
   - `writeMessagesText` prints `#&lt;id&gt; from "&lt;sender&gt;"` (`internal/creelcli/cli.go:600`). D6 keeps `from` as the bare name and adds trust and hub only "as fields of their own", and it never says the text header an agent reads carries the stamp.
   - D6 sends cross-hub mail to a pane-mail strand only as a reminder that says "run lym creel read" (doc.go:93). Every Codex strand is a pane-mail strand (doc.go:84). So the body reaches such a strand only through this header, with no `(other hub, trusted orch)` or `(other hub, untrusted)` marker.
   - Trusted and untrusted mail from `ly:orch` therefore look identical there, and D7's instructions refer to a stamp that path never shows.
   - Fix: D6 should state that the read text header carries the stamp after the quoted sender, and an acceptance bullet should cover it.
2. **MEDIUM, design. D2/D3: daemon-to-daemon requests go through the local sender check, and the record does not say how.**
   - Every request is resolved by `d.resolve` before `dispatch` (`internal/creelengine/server.go:208-213`). A peer in no pane is refused whenever any live strand's pane process is unknown or the strand listing fails (`internal/creelengine/identity.go:68-69`, `paneOfPeer` at 120-131).
   - A `deliver` from the sending daemon, or a `confirm` from the receiving daemon, is such a peer. It would be refused with a generic error that is neither `accepted`, `refused` nor a transient refusal.
   - D2 says a `refused` returns the message at once, so the record leaves open whether this error returns the message or retries it. It is also silent on whether the two verbs skip resolution, and on which sender kind they run as (EXTERNAL).
   - Fix: D3 should state that `deliver` and `confirm` bypass the local sender check, or that its failure counts as transient.

Tool calls:
- Grepped for the creel line formatters and the request caps, to find every path that renders the `from "&lt;sender&gt;"` line D6 changes.
- Grepped for `CreelGeometry` callers and readers of `Sender.Kind`, to find the other builders of hub geometry and the strand-only gates.
- Read `writeMessagesText` in `cli.go`, the strand-only checks in `verbs.go` and the verb list, to see what `lym creel read` prints and which verbs gate on the sender kind.
