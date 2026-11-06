```yaml
lens: fix-claims
outcome: reported
```

Task: check the fix's added or rewritten claims against the code at HEAD, through the fix-claims lens.

no findings

Claims checked, all of which hold:
- **`hubFake` raw line:** `readLine` (`internal/creelengine/protocol.go:135`) needs the `\n`. A raw line with a line end therefore reaches `json.Unmarshal`, whose failure in `Client.exchange` is a plain error rather than `connLostError`, so it falls to the `bad-answer` catch-all.
- **Unknown-verb refusal:** `internal/creelengine/server.go:384` gives `creel: unknown verb %q`. That is daemon refusal text, which reaches the notice quoted.
- **`hubs` with a failing `seam.registryDir`:** `runVerb` passes a body error to `refuse(cmd, err)`, and `errMissingStateHome` names both `XDG_STATE_HOME` and `HOME`. The fix's wording is limited to a missing home, so it stays true of `errRelativeHome`, whose text names neither.
- **None refusal:** `lym creel inbox` runs through `runVerb` and `client.Call`, so it reaches the daemon, which records the hub (card 4).

Calls:
- One Bash grep read `readLine`, `runVerb`, the unknown-verb refusal and `inboxCmd`, because my context held only `Client.exchange` and the `standalonestate` errors.
