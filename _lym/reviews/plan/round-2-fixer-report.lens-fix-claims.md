```yaml
lens: fix-claims
outcome: reported
```

fix-claims: I checked every claim the fix added against the code at HEAD.

**MEDIUM: the promised "way forward" is missing from some errors.** This concerns card 6's new sentence "A hub whose geometry fails is an error naming that hub's path and the geometry's own error with its own way forward". It also concerns card 9's "carrying the seam's error as it is, which names the path and its way forward" and card 10's "which names the hubs record or the failing hub and its way forward".

- Summary: some geometry errors and some `Live` errors carry no way forward. A notice that quotes the seam's error as it is then gives the operator none, which falsifies the "its way forward" claims in cards 9 and 10.
- Evidence, geometry errors:
  - `internal/hubgeom/creelgeom.go:22` returns `hub %q has no usable shortname record %s`, with no way forward.
  - `internal/hubgeom/slackgeom.go:32` returns `slack: find the running lym binary: %w`, with no way forward.
  - Only `slackgeom.go:28`, the error for no per-user directory, names one.
- Evidence, `Live` errors:
  - `internal/hubregistry/hubregistry.go:146-150`: `refuseUnreadable` adds the way forward only for `state.ErrDecode`. Any other read error passes through unchanged.
  - So a failed `Live` read that is not a decode error, such as an I/O or permission error, carries the path from card 6's wrapper but no way forward.
- Possible fix: `liveHubGeometries` adds a way forward of its own when it wraps a geometry error and a `Live` error that is not a decode error. Or cards 9 and 10 drop "and its way forward", saying the notice carries whatever way forward the error names.

Claims that hold:
- `Live` → `readEntries` → `refuseUnreadable` at `hubregistry.go:98-123`. The decode refusal names the file and "remove …, after which lym creel inbox in each hub, a minute on, records it again", as card 6 says.
- `newSlack` builds `&http.Client{}` at `internal/slackcli/wiring.go:112`, matching "the client built as `newSlack`'s is".
- Card 2 gives `NewDialer(baseURL, appToken string, client *http.Client, helloTimeout time.Duration)` and `HelloTimeout`, so the new card 6 closure is well-typed.
- `chat.postMessage` is the engine's posting method (`internal/slackengine/api_test.go:161`).

Tool calls:
1. I read `hubregistry.go:40-68`, `hubgeom.SlackGeometry` and a grep for `chat.postMessage`. My context did not show `Live`'s body or the geometry error paths.
2. I read `Live`, `readEntries` and the error returns of `CreelGeometry`. The first call landed on `Record`, not `Live`, and `CreelGeometry`'s errors were not in my context.
