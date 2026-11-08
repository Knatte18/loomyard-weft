---
verdict: APPROVED
---

Reviewed `8a54fbfbab5aeea0ddb844a9e9a476b0cb2b95f7..HEAD` against the decision record and round focus.
No findings.

The removed prefix has no remaining reader in production code, contracts or documentation.
The affected readers were the orch role, inbound expected bodies and the relay test's `forwarded` predicate;
the diff updates each.
`handle` still forwards through `forward`, and the production mailbox signs its posts as the SLACK program.
Creel renders the sender independently of the body.
The role file remains embedded through `orchrole.OnDemand` and written by `RenderRoleFile`.

The notice producers in `SetPresence`, `readFailed`, `missingTokenExit`, the holder, channel keeper and outbound fallback
all retain either the presence or relay notice prefix.
The revised role distinguishes those notices from operator mail.
The overview and package documentation no longer describe the removed prefix.

The existing inbound assertions compare complete forwarded bodies, including attachment lines and converted Slack markup.
The cap rows now exercise 4096 and 4097 bytes, and the relay predicate compares the bare body exactly.
No test was added, and the changed expectations introduce no duplicate scenario or new wait.

Verification limit: `timeout 3600 lym gate test ./internal/slackengine` could not acquire a gate slot.
The gate's holder and process records are under the hub's `_bolt/.lym`, outside this session's writable paths.
No tests ran in this review; this infrastructure refusal is not a finding against the diff.
