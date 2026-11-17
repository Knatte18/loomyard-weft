---
round: 2
focus:
  - "Every card that creates a production function: does a card at or before it call that function from a `main`-reachable path, or does the plan carry an allowlist entry (`cmd/lym#testOnlyCodeAllowed`) that a later calling card removes, as `PATTERN-test-only-code` and its tier1 guard in `cmd/lym/testonlycode_linux_test.go` require at every batch boundary?"
  - "Cards 10 and 11 run tier0 rows over a fake mailbox in `slackengine_test`: can that fake produce an error that `creelengine.IsDaemonRefusal` matches, given `daemonRefusal` is unexported and built only by `Client.do`?"
  - "Card 11's bridge-channel `approve` and `revoke` call `runPeerCommand` from the bridge-poll goroutine: what happens when it fails, and which goroutine owns the inbound state it touches (`failingKey`, episodes, `answer`), given `wakeLoop` in `internal/slackengine/relay.go` documents a single reading goroutine?"
  - "Does the card that introduces each new verb, flag or routing edge state in its `**Intent:**` what it lets through and what bounds it, in particular the program-only deliver verb that stamps mail `(other machine, trusted orch)`?"
---
