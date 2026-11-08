```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Task: fix-consistency lens on the F1–F4 change set.

Checks, all consistent:
- **F4 twin:** no other "three seconds" or "within 3 s" acknowledgement claim exists in slack code or docs. The reworded `Socket.Ack` comment has no stale twin.
- **F1:** the `login` Long help ("Replacing a stored token ends the relay of each live hub of this machine; run lym slack start in each") and the `endLiveRelays` doc agree with the new collect-and-continue behaviour. The error still names the stored tokens and the stop-then-start way forward. The new test row asserts both hubs are stopped and both are named. On failure the fields are discarded by `runVerb`. Ended hubs are therefore named in the error text, which is the only visible channel, and `relays_ended` is set only on success.
- **F2:** `appIDMissingRefusal` uses the same "nothing was stored; way forward: run `lym slack login`" shape as the mismatch refusal. The test rows' wanted substrings ("no app id", "nothing was stored", the token kind) match the message text. In the empty-`bots.info` row `h.appID = ""` also leaves the hello empty, but the bot-side check runs first, so the "bot token" want holds.
- **F3:** the test's `LOCALAPPDATA` and `XDG_CONFIG_HOME` handling matches how `UserStateDir` and `githubclient.cacheDir` read them. The Windows skip is justified because both use `LOCALAPPDATA`.

Tool calls:
1. One grep over `internal`, `docs` and `pattern` for the "three seconds" twin and the `relays_ended` / app-id wording. I ran it because my context did not show those docs.
