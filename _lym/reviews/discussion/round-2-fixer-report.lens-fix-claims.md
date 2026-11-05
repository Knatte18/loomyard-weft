```yaml
lens: fix-claims
outcome: reported
```

Task: check that each bound, behaviour or code fact the fix added or rewrote holds against the code at HEAD.

**Claims checked and holding:**
- **D4 and D6, creel refuses a read or post while a live strand's pane process is unknown.** This holds for every request, a read included. `server.go:208` resolves every request through `resolveSender`, and `identity.go` refuses on `unreadable != ""` before it looks at the program claim.
- **D5, the 30-second deadline and "a refused or lost acknowledgement".** This holds. `awaitAck` (`server.go:478-480`) sets the deadline once the read is handed out and answers either a confirmation or a refusal. Any other answer releases the read, so the client can release on purpose.
- **D8, the `resumed_at` of the latest resumed entry covering the child.** This holds. `pauserecord.Entry.ResumedAt` exists, resumed entries are kept for `Retention`, and dobby already reads coverage by slug (`internal/beamcli/dobbydeps.go:169`).
- **D3 re-exec Bound, "only one relay polls".** This holds, because D3 takes the lock before the first creel read and before the first poll.

**Findings:**

- **LOW**
  - **Text:** D2, "a changed channel takes D4's first-start path", and the D4 criterion "after a `lym slack setup` changes the channel, the relay forwards none of the new channel's history".
  - **Summary:** Keying the cursor by channel id helps only for a channel the relay has never relayed. If setup switches back to a channel relayed earlier, the relay finds that channel's old cursor and forwards everything posted there since.
  - **Evidence:** D4 now reads "a cursor keyed by channel id… A first start with no cursor for the configured channel writes…". Nothing drops a cursor when the channel changes, so the stated bound and criterion are wider than the design holds. A fix would be to drop other channels' cursors when setup writes a new channel, or to narrow the claim to a channel never relayed before.

Tool calls:
1. grep of `server.go` for request identity resolution, to confirm that reads go through `resolveSender` (my context only showed `identity.go`).
2. `sed` of `server.go:205-226` plus greps for the pause-record path, to confirm that every request is resolved and to find where the pause record lives.
3. grep for readers of `pauserecord` and the pause directory name, to confirm that dobby already reads pause coverage by slug.
