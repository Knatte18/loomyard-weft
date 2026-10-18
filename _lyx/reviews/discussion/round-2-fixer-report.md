# Fixer report — discussion, round 2

## Changes

- F1 (MEDIUM), fixed as proposed: D6 states that Reconcile's launcher restore stays and that an otherwise healthy pair whose launcher directory it restored reports a new `launchers_restored` action in place of `portal_restored`.
  An acceptance criterion pins it.
  The premise holds: `restorePortalAndLaunchers` in `internal/fabricengine/reconcile.go` restores the launcher directory on its own, and `ReconcileActionPortalRestored` is set when either restore ran.
- F2 (LOW), fixed as proposed: D2 states that lyx owns the file, that every regeneration overwrites edits made in VS Code or by hand, and that workspace settings belong in the prime's `.vscode/settings.json`.
  An Open risks bullet names the loss of a folder or setting added through VS Code's workspace UI.
- F3 (NIT), fixed as proposed: D3 and the failing-regeneration criterion say "writes no workspace file" in place of "writes nothing".
  D3's rationale, "Writing nothing on a failed prime resolution", got the same change.

## Self-check

- fix-consistency, LOW, Scope Out puts `_launchers` out of scope while D6 adds a launcher action: fixed as proposed, Scope In gains a bullet for Reconcile's launcher-restore report.
- fix-claims, LOW, and fix-way-forward, LOW (merged), the criterion leaves out that the action replaces only `already_healthy` (`reconcile.go:405`): fixed as proposed, D6 and the criterion say "otherwise healthy".
- fix-proposal, LOW, the launcher restore creates launchers for a local-only pair that `AddLocal` deliberately skips (`add.go:140`, `tools/sandbox/SANDBOX-FABRIC-SUITE.md:576`): fixed as proposed, D6 has the restore skip a pair whose warp branch carries the local-only mark, and the criterion says a local-only pair gets no launcher directory.
- fix-proposal, LOW, the help text in `internal/fabriccli/fabric.go` and the sandbox suite still name `portal_restored`: fixed as proposed, D8 names both as updated to `launchers_restored`.

## Disputed

None.

## Deferred

None.
