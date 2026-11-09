```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-2 fix's changes to `decision-record.md` agree with the sections and criteria around them, and find any change whose twin elsewhere was left as it was.

**[LOW] D11: archive wiring sentence is now stale**
- **Changed text:** the new D11 paragraph saying `lym beam start`'s foreign-owner refusal "runs the same fabric archive call for the subject".
- **Problem:** the unchanged sentence above it, "The archive is one fabric call the dobby's deps wire.", still names the dobby as the only thing that wires the call.
- **Evidence:** D11 now has two entry points, the dobby's probe (steps 1–3) and the beam-start refusal. That sentence still describes only the first, and so does the Constraints line "the dobby's Bolt pull, the archive and the archive read are fabric calls". A plan writer could take the archive to be dobby-only wiring.
- **Suggested change:** reword it to say both entry points share one fabric call, for example "The archive is one fabric call, which the dobby's deps and `lym beam start` both wire."

I checked the other changes against their siblings and found no disagreement:
- **D2 field list (F3):** agrees with D2's "keeping the done steps and stored inputs" and with D9's "the steps done".
- **D2 discard criterion (F5):** agrees with D1's criterion that a caller whose worker lost the lock to `--discard` exits non-zero naming the re-run, and with D2's "refuses while the lock is held".
- **D3 added sentence and D5 beam-slug conversion (F2):** agree with each other, with the D5 criteria, and with the Constraints' "pauses only slugs the record does not list as paused".
- **D5 criterion rewrite (F4):** matches D5's two exceptions, the front task and a slug holding another caller's entry.
- **D11 Bound and new criterion (F1):** agree with D10's `lym s` order (`adopt --here` before `beam start`), with D4's coverage condition, with D10's Bound reference to "D11's old-owner path", with D12's "the old owner archives only after this adopt's owner write reaches it", and with the Scope line's "the old owner's archive".

I made no further tool calls.
