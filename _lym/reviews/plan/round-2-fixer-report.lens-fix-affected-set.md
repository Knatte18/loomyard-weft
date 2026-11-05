```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun each set-premise finding's search (F1, F2, F3, F4) and check that the fix covers every member of the set.

Unchecked findings: none. F1, F2, F3 and F4 were all rechecked, using 4 of my 6 tool calls.

**LOW**: `_lym/plan/12-dobby-relay-tick.md:25` is a stale twin of F1's new failing-spawn path.
- **Changed text:** card 09's new bullet text "Before it calls `spawn`, `starter.ensure` writes `LastSpawnAt` … and returns the spawn's error after that write, so a spawn that fails to start … leaves a `LastSpawnAt` with no accepted first read after it". The new `TestEnsureRelay` row "a fake spawn that returns an error … the sixth answers `gave-up`" says the same.
- **Summary:** after the fix, `gave-up` also follows five spawns that never started a relay. Card 12 still describes the cap as "card 9's cap after five relays in a row that never made their first read".
- **Evidence:** card 12 line 25: "A `gave-up` outcome, card 9's cap after five relays in a row that never made their first read, is logged at Warn once per episode naming the relay log and `lym slack start`". A failed `cmd.Start` produces no relay at all, so a reader of card 12 would miss that path. Card 12 line 22 returns `EnsureRelayWhileAway`'s error joined into the tick's errors. That still holds, because a due tick now returns the spawn error rather than looping.

**F2 set** (texts that say every creelcli call has a fixed timeout): covered.
- Only `internal/creelcli/doc.go:47` and `internal/creelcli/goclient.go:17` make that claim, and the fix rewords both.
- `internal/creelcli/cli.go:320` says "the client bounds every request by its timeout". That stays true with a caller-chosen timeout, so it is not a member of the set.

**F4 set** (what `clitree_test.go` and `registration_test.go` require of a mounted module): covered.
- The tests require a `Short` on every command, the module named in `root.Long`, no bare `run` or `start` child, and the registration parse of `main.go`.
- The `Long` entry is the one the fix adds. Card 10 already provides the others: a one-line `Short` for each verb, and `newRoot` under **Edit:**.

**F3 set** (users of `Hub.Paused` and `Hub.Beams` outside card 07): covered. Only card 10:75 wires them, and the change of error policy does not reach that wiring.

Tool calls:
1. Grep of `internal/creelcli` and `docs` for fixed-timeout texts: my context had only the two lines the fixer grepped, not the whole package.
2. Grep of `cmd/lym` tests for module lists naming `creel`: my context lacked the test files.
3. Grep of the other plan cards for the restart-cap fields and `Hub.Paused`: my context held only cards 03, 07, 09 and 10.
4. Grep of the requirements that `clitree_test.go` and `registration_test.go` check: my context showed only line 151.
5. Grep of the plan cards for `EnsureRelay` callers and `gave-up`: my context lacked cards 11 to 13.
