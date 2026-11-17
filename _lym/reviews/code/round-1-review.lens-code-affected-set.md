```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens.

**Per change, the set I found:**
- **`&lt;machine&gt;::` address and the sole-parser rule:** the callers are `creelengine/crossmachine.go`, `machinemail.go`, `machineinbound.go`, and `slackengine/bridgeout.go`, `bridgemessage.go` and `bridgein.go`. The slackengine and `machineinbound.go` callers go through `ParseMachineAddress` and are covered. Two sites in creelengine split the separator themselves, which is finding 1.
- **Stamp and authority passages (D9):** the set is `contracts/orchrole/orch-role.md`, `contracts/orchrole/role-slack.md`, `contracts/stencils/parent/parent-directive-parent.md`, `internal/shuttleengine/creelhooks.go` and `internal/creelengine/fold.go`. All four passages that grant authority name the new stamp. The other files that mention the `(other hub, …)` stamp are tests and `docs/research` files. Covered.
- **`SendingMachine` added beside `SendingHub`:** the non-test `SendingHub` readers are:
  - `verbs.go` Reply and Forward: updated.
  - `cli.go` message fields: updated.
  - `sweep.go:239`: reads `SendingHub` only. For machine-inbound mail `msg.From` is `machine::name`, which `isStrandName` rejects, so the message is still journaled undelivered. Covered, no finding.
  - `receive.go` dedupe and `crosshub.go` width: hub-only by design. Covered.
- **Declared `PATTERN-agent-addressing` change:** the `PATTERN.md` line matches the declared text exactly, and the background gains the "Across machines" rule and the hand-off in the Slack boundary rule. Covered.
- **Other PATTERN files:** `pattern/PATTERN-daemon-wakeups.md` lists the machine hour loop and the bridge poll, and `cmd/lym/daemonwakeups_test.go` lists both files. `pattern/PATTERN-sole-parsers.md` lists the envelope. Covered.
- **Commit trailers:** the range's commit bodies carry no `Pattern-Departure:` trailer, so none needs judging.
- **Docs:** `docs/slack-app.md` gains the pairing section and the manifest is unchanged. The `creelcli/doc.go`, creelengine `doc.go` and `slackengine/doc.go` diffs are covered. Covered.

**Findings**

1. **BLOCKING, class `design`.** Code outside `internal/agentname` splits the `&lt;machine&gt;::` address, against `PATTERN-agent-addressing` ("only `internal/agentname` forms, parses or matches the `&lt;machine&gt;::` name", as the decision record's Constraints repeat). There are two sites:
   - `internal/creelengine/crossmachine.go:177` does `strings.Cut(original.From, agentname.MachineAddressSeparator)` to get the receiving name in `replyAcrossMachinesLocked`.
   - `internal/creelengine/machinemail.go:232` does `strings.Cut(to, agentname.MachineAddressSeparator)` to get the machine in `machineHoldWayForward`.

   Fix: have `agentname` expose the split, or use the parsed address or the entry's `ReceivingMachine`, and have both callers use that.

Calls: 4 in total, all reads. Three greps covered `SendingHub` readers, passages carrying the other-hub stamps, and separator splits outside agentname. One read of `sweep.go` lines 215–265 checked the removed-holder sweep. My context lacked all of them because they are searches beyond the manifest.
