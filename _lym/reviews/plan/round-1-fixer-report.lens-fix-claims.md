```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD (fix-claims lens).

The claims about the F1, F2, F3, F5 and F6 changes hold:
- A `daemonRefusal` that wraps `codedRefusal` keeps `errors.Is` through `codedRefusal.Unwrap` (`internal/creelengine/identity.go:31`), as long as `daemonRefusal` itself has an `Unwrap`, which "wraps" implies.
- No code type-asserts `codedRefusal`. `client.go:161` checks only `connLostError`, so a `daemonRefusal` still ends the call with no reconnect.
- `Response.HubPath` and `Response.Code` exist (`internal/creelengine/protocol.go:90-91`).
- `internal/reportengine/doc.go:203` holds the Replies section, and the `RoleReplies` target exists.
- Both verb lists in `docs/overview.md` exist.

Findings:

1. **LOW.** `_lym/plan/04-hub-directory.md`, Intent: "a live daemon whose last record failed records the hub at the next verb it serves".
   - Summary: this overstates the behaviour, because not every served verb records.
   - Evidence: the sentence before it limits `recordHub` to "each locally resolved verb". Card 7 says of `deliver` and `confirm`, which `daemon.serve` also serves, that "these verbs skip `recordHub`". A verb that `resolve` refuses (`internal/creelengine/server.go:208`) is not locally resolved either.
   - The claim should read "the next locally resolved verb", as the sentence's own later "such verb" does.

2. **LOW.** `_lym/plan/01-hub-registry.md:56`: "after which `lym creel inbox` in each hub records it again (card 4)", with card 4's "so card 1's corrupt-record way forward holds at once".
   - Summary: the claim is unconditional, but it fails for a hub whose last record succeeded inside the past minute.
   - Evidence: hub B's daemon records successfully at t0, and the file goes corrupt at t0+10s. Hub A's verb is refused, the operator removes the file, and `lym creel inbox` in B at t0+30s skips the record, because B's last record succeeded and `hubRecordInterval` has not passed. B's entry stays missing until a verb past the interval, and card 4's own second clause says so. Card 1's sentence and "at once" don't carry that bound.
   - The "at once" clause holds only for a hub whose own record failed, and card 1 says "each hub".

Tool calls:
- Read `internal/creelengine/client.go:1-200`: my context held only lines 200 onward, and I needed the `Call` and `connect` error paths.
- Grep for `VerbInbox`, `errors.As` and the `Response` fields: to check that inbox is a served verb, which error type checks exist, and the field names.
- Read `internal/creelengine/verbs.go:115-139`: to confirm the `errors.As` at line 128 concerns `interruptRefusal`, not a client refusal.
