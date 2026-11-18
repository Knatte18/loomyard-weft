```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged each proposal the fixer carried out as written (F1, F2, F3, F4, F6, F7, F8, F9).

**MEDIUM — F9's cut drops the only criterion tying the bootstrap to the builder**
- Changed text: `_lym/discussion/decision-record.md`, Acceptance criteria, first D2 bullet. The clause "the loom bootstrap and `ensureRunsStrand` both use it" is gone.
- Summary: F9's premise was that "the D1 and D2 criteria already check the specs both paths produce", but no remaining criterion checks the spec the bootstrap produces.
- Concrete case: the runs strand comes first, then `lym loom start`. The bootstrap could keep today's `statusStrandAddSpec` at `internal/loomcli/bootstrap.go:31-44`, which sets an own-window anchor with `KeepViewer` and no `Display.Window`. Every criterion would still pass: the D1 bullets drive `ensureRunsStrand` only, and the D2 bullets test the builder in isolation. The checklist window would then be named `loom-status`, not the recipe, which breaks the Goal and D1's Rationale claim that "It covers both orders".
- Fix: replace the cut clause with a behavioural one, for example "D2: with a runs strand present, `lym loom start` adds the status strand with the own-window anchor, `KeepViewer` and the recipe's window group."

**No case found for the other proposals:**
- **F1:** the derivation is right. `createOwnWindowPaneLocked` stores the group record before it returns on the place-apart branch (`spawn.go:183-186`), so at `window_share_max` 0 the record exists and says not shared. For a single-member group, liveCount is 0, so a shared record always means the window bears the group's name.
- **F2 and F3:** `resolveStatusStrandAction` (`internal/loomcli/bootstrap.go:226-236`) reaches the add branch only when no `loom-status` strand was found. A skip by `AddStrandUnless` can therefore only mean another add landed concurrently, never a strand the bootstrap had seen and wanted to replace.
- **F4, F6, F7, F8:** no case found.

Tool calls:
- Read `resolveStatusStrandAction` in `internal/loomcli/bootstrap.go`, to test whether F3's skip can hide a strand the bootstrap meant to replace. My context lacked that function.
- Grepped for other `Display.Window` group names, to test whether a recipe group can collide with another group under F1's derivation. My context lacked the other group names. Only the seat tables' groups turned up, and they are unlikely to collide.
