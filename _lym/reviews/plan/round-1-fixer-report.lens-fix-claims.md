```yaml
lens: fix-claims
outcome: reported
```

Task: check, through the fix-claims lens, that each claim the fix added or rewrote holds against the code at HEAD.

no findings

What I checked:
- **Card 3, "Every reading `Store.Read` does not make is a claim's".** This holds. The only `Reading` constructors in production code are `internal/creelengine/verbs.go:264` (`Read`) and `verbs.go:312` (`ClaimAtBoundary`), and card 4 adds the ring's reading through `claimLocked`.
- **Card 4, "its registration's wake has the worker probe again".** This holds. `HookWait` calls `wakeLocked(sender.Name)`, which appends to `s.wakes` (`pool.go:181`), and `pool.go:164` drains that into the wake hook. The comment on the superseded subtest at `cli_test.go:983` relies on the same behaviour: "each registration wakes the worker, whose probe reads the turn".
- **Card 6, the superseded subtest's new stderr of `hookLine(1, alpha, "mail")` and `hookLine(2, alpha, "more mail")`.** This holds. `hookLine(id int, sender, body string)` (`cli_test.go:154`) is used with sender `alpha` throughout the hook subtests. Both short posts are claimable when the "more mail" post's wake rings the newer waiter.
- **Card 10, "Stop hook feedback lines" and "Stop says line".** These names come from D1 (`decision-record.md:36`, `:40`), not from the smoke code, so they are grounded.
- **Card 4 F3 Bounds, F5 bound sentence, and the Constraint exceptions.** These match `decision-record.md` lines 72–75 and 127–129, and D3's Stop-count reset.
- **Card 9, "a released empty claim keeps its Debug `nothing` line".** This matches `handOut`'s `Release` on a failed write (`server.go:455-458`).

Tool calls:
1. Grepped the `Reading` constructors, `wakeLocked` and `hookLine`: my context did not hold their definitions.
2. Grepped `hookLine` usages, the `s.wakes` drain and the smoke pane strings: I needed the sender convention and the wake delivery.
3. Grepped the smoke test's Stop wording and the read client's ack handling: my context lacked the smoke file.
4. Grepped the decision record for the D1 pane terms: needed to ground the card 10 wording.
5. Grepped the client's ack path: I wanted to confirm that the read client prints before it acks. This was inconclusive. The claim was carried over from card 3's original text and the fix did not add it.
