```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens, tracing each change in the diff to a goal point of the board entry under `## Goal`.

**Decisions** (the record is a decision record alone; it has no `Added after Discussion` entries):
- **G1.** `clientAgentPaneVia` / `ClientAgentPane` must read the pane of the clicking client's own session, not the most recently active client's session. The board entry suggests "pass `-t <client>` (or resolve the client's session first)".
  - Built by C1, with C2–C6 forced by C1.

**Changes:**
- **C1.** `internal/reedengine/remotecontrol.go` `clientAgentPaneVia`: the read changes from `display-message -p -c` to `list-clients -F '#{client_name}\t…'` and takes the clicking client's row. It adds the error `no client %q is attached` for a client absent from the listing.
  - Traces to G1.
  - tmux expands each `list-clients` row against that client's own session, so this does what the entry's "resolve the client's session first" asks, in one call. The entry's `Fix:` text is a suggestion, and `summary.md` records why this route was chosen over it.
  - The new error is forced: the row lookup has a "not listed" outcome the old read did not have.
- **C2.** The doc comment added to `clientAgentPaneVia` (remotecontrol.go:38-40): forced by C1.
- **C3.** The argv and rows of `TestClientAgentPaneVia` in `internal/reedengine/remotecontrol_test.go`, including the new rows `OtherClientsRowsAreSkipped` and `ClientNotListed`: forced by C1 (new argv, new output shape, new error).
- **C4.** The new `internal/reedengine/remotecontrol_integration_test.go`, `TestClientAgentPane_ReadsEachClientsOwnSession`: the regression test for G1's defect. It is a test that G1's fix forces.
- **C5.** The fake tmux in `internal/orchcli/remotecontrol_test.go` now answers `list-clients` with a four-field row, and its comment changes: forced by C1, because the rc-on verb test's fake must answer the new call.
- **C6.** Commit `01218f11f`, which breaks over-long comments in the same files: forced by the comment lint on C2–C4's comments.

**Not in the diff, correctly left out:** `switch.go` and `buttonclick.go` have the same `display-message -c` defect. They are outside G1, which names only `ClientAgentPane` / `clientAgentPaneVia`. They are filed as board note `reed-switch-wrong-client` (B2) and not changed, which is the right scope.

**Findings:** none. Every change traces to G1 or is forced by C1, and the diff builds G1. No `Pattern-Departure:` trailer is present or needed.

Tool calls: none. The diff pieces, the commit log, the decision record, `summary.md` and board note B2 were already in my inherited context.
