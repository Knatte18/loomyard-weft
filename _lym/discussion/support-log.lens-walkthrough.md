```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism end to end, one concrete case each.

**D1.** Case: a post arrives after the last tool call, the flag stands and the strand's Stop count is known.
- The merged Stop command pipes the input to `hook-read claude`.
- The claim hands out one line, and the client writes JSON holding only `hookSpecificOutput.additionalContext`, then exits 0.
- `&& exit 0` skips recording a turn end.
- Claude Code 2.1.312 parses the Stop `hookSpecificOutput` (its parse switch has a `case "Stop"` for additional context) and pushes the `hook_additional_context` attachment into the continuation list.
- The turn continues with `stopHookActive: true`.
- The operator summary's `f` list gets `Stop hook feedback: <line>`; `g` stays empty, so there is no error line.
- Purpose met.

**D2.** Case: mid-turn mail at a Bash PostToolUse.
- `hook-read` prints `additionalContext` and `systemMessage`, then exits 0.
- The PostToolUse path yields `hook_additional_context` with the hook name `PostToolUse:Bash`, so the model reads it once with no command text.
- The model renderer maps `hook_system_message` to `[]`, so the system message reaches only the operator, as `says:`.
- Purpose met, and D2's claim that the model sees one copy holds.

**D3.** Case: one 200-byte mail to an idle hook strand with a registered waiter.
- The worker reads no turn in progress, and `ringHook` claims and reserves under the lock.
- The ring outcome reaches `hook-wait`, whose `Client.Call` handle writes stderr. `Call` already acknowledges after the handle when `AckRequired` is set, and the server's `handOut` journals the read on that acknowledgement.
- `hook-wait` exits 2. Claude Code reads stderr after exit and enqueues the notification with priority `next`, which wakes the model with the mail.
- The woken turn's Stop finds no flag and records the turn end.
- Drop path: the client misses the ring line, `handOut` releases the mail, and the reconnecting waiter re-registers. The same set is never rung twice, so the bell fires after 30 s.
- Purpose met, with one finding below.

**D4.** Case: the same ring.
- The model body is `[creel] Mail that arrived while you were idle: [creel] #12 from "...": ...`.
- The operator summary is `creel mail`, taken from the setting.
- Purpose met, with one LOW finding below.

**D5.** Case: a claim that hands out a line, followed by a client that dies before acknowledging. This logs `Info` handed out, then `released`. A Stop at the cap logs `Debug` with the cap reason. Purpose met.

Findings:

- **LOW — D3, "same locked step … pause".** The pause cannot be read inside the store's locked step as D3 states.
  Evidence: `ClaimAtBoundary` calls the seam's `delivery.Paused` before `s.do`, outside the lock. A ring claim inside `ringHook`'s `s.do` would have to make that seam call under the store lock.
  Fix: state that the ring reads the pause before its locked step, as `ClaimAtBoundary` does, or that it relies on the probe's pause skip. The rest of the claim rules (hold, line cap, budget) do run in the locked step.
- **LOW — D4, model body.** Claude Code passes the rewake body through `bWe(rx(...))` before enqueueing it. `rx` escapes `system-reminder` tags and `bWe` replaces a fixed set of patterns, so a mail whose text holds one of those patterns reaches the model altered.
  Evidence: in `Bin({summary:bWe(rt),body:bWe(rx(...))})`, `bWe` reduces over `rL([...Xwi,TN,A4,MF,rBe,yw,s_,"duration_ms"])` with `OFe`.
  D3's rationale says the stderr is "escaped, not truncated", which holds, but D3 does not mention this rewrite. The Stop and PostToolUse fields may face the same escaping; I did not check.

Extra calls:
- I grepped the bundle for `hook_system_message` in the model renderer map, to know whether D2's system message also reaches the model. My context only held the terminal renderer.
- I read the PostToolUse hook loop after `updatedMCPToolOutput`, to confirm that path yields additional context. My context had seen only the PostToolUseFailure variant.
- I grepped `internal/creelengine/client.go` and `server.go` for `Call`, the acknowledgement and the hook-wait handling, to check D3's acknowledge-on-same-connection step. My context had not read the client or the server.
